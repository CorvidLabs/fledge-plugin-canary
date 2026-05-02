# fledge-plugin-canary

> **Native (unsandboxed) security canary** — proves what an untrusted plugin can access when running as an unsandboxed subprocess with full user permissions.
>
> Companion: [fledge-plugin-canary-wasm](https://github.com/CorvidLabs/fledge-plugin-canary-wasm) — the sandboxed counterpart that proves the Wasmtime WASM runtime blocks every attack this plugin exposes.

First-party security audit tool for [fledge](https://github.com/CorvidLabs/fledge). Written in **bash**, runs as a native subprocess — the same way all pre-1.1.0 fledge plugins run. Probes plugin capability boundaries and reports what data a plugin can access with zero capabilities granted.

**This is not a malicious plugin.** It's a canary-in-the-coalmine that validates fledge's security claims by testing them from the inside. It declares zero capabilities (`exec=false, store=false, metadata=false`) to demonstrate what an untrusted native plugin can access.

## Key Finding

**Capabilities gate the fledge-v1 RPC protocol, not the plugin process.** A native plugin with zero capabilities can still:

- Read `~/.ssh/`, `~/.aws/credentials`, `~/.config/gh/hosts.yml`, and other credential files
- Read shell history (which often contains pasted tokens)
- Write to `.git/hooks/`, `~/.zshrc`, `~/Library/LaunchAgents/` for persistence
- Exfiltrate data via `curl`, `wget`, or DNS queries
- Access the clipboard (`pbpaste` on macOS)
- See all running processes, hostname, and username

The capability system correctly blocks unauthorized RPC messages (metadata, exec, store all return denial responses). But the plugin process itself is an unsandboxed subprocess with full user permissions.

**WASM plugins (fledge 1.1.0+) fix this.** The Wasmtime sandbox enforces boundaries structurally — missing WASI preopens mean files don't exist, missing imports mean functions can't be called.

## Native vs WASM Comparison

| Attack | Native (bash) | WASM (sandbox) |
|--------|:---:|:---:|
| Read ~/.ssh/id_ed25519 | LEAKED | BLOCKED |
| Read ~/.aws/credentials | LEAKED | BLOCKED |
| Read ~/.config/fledge/config.toml | LEAKED | BLOCKED |
| Read shell history | LEAKED | BLOCKED |
| Inherit GITHUB_TOKEN env var | LEAKED | BLOCKED |
| Inherit OPENAI_API_KEY env var | LEAKED | BLOCKED |
| Exfiltrate via curl | AVAILABLE | BLOCKED |
| Exfiltrate via DNS (dig) | AVAILABLE | BLOCKED |
| TCP connection to any host | AVAILABLE | BLOCKED |
| Spawn shell commands | AVAILABLE | BLOCKED |
| Write .git/hooks (backdoor) | WRITABLE | BLOCKED |
| Write shell RC files | WRITABLE | BLOCKED |
| Install LaunchAgent daemon | WRITABLE | BLOCKED |
| Read clipboard (pbpaste) | AVAILABLE | BLOCKED |
| Schedule crontab | AVAILABLE | BLOCKED |
| List processes (ps aux) | AVAILABLE | BLOCKED |

**Native**: unsandboxed subprocess with full user access.
**WASM**: Wasmtime sandbox with `filesystem=none`, `network=false`, `exec=false`.

## Install

```bash
fledge plugins install CorvidLabs/fledge-plugin-canary
# No capability prompt — the plugin requests zero capabilities
```

For the WASM sandbox companion, see [fledge-plugin-canary-wasm](https://github.com/CorvidLabs/fledge-plugin-canary-wasm).

## Usage

```bash
# Run all tests (RPC denial tests + baseline when no caps granted)
fledge canary

# Run a specific section
fledge canary metadata    # Metadata RPC tests
fledge canary exec        # Exec RPC tests
fledge canary store       # Store RPC tests
fledge canary baseline    # Direct process access tests + WASM contrast
fledge canary expose      # Full exposure report

# Legacy mode — shows unfiltered env inheritance
fledge canary-legacy

# WASM canary (if installed separately)
fledge canary-wasm
```

## What It Tests

### RPC Capability Tests

These test the fledge-v1 protocol's capability gating — whether the RPC layer correctly blocks unauthorized requests.

#### Metadata (`fledge canary metadata`)
- Can the plugin read `fledge.toml` via the metadata RPC?
- Which env vars pass through the metadata RPC filter?
- Are sensitive patterns (`TOKEN`, `SECRET`, `KEY`, etc.) blocked?
- Is git metadata (status, log, tags) accessible?
- **With capability denied**: all metadata RPCs return empty/null (PASS)

#### Exec (`fledge canary exec`)
- Basic exec functionality
- Can exec read `~/.config/fledge/config.toml` (global config with tokens)?
- Can exec access files outside the project root?
- Does `cwd` path traversal (`../../..`) get blocked by `canonicalize()`?
- Which env vars are visible in the exec shell environment?
- Can exec write to `/tmp`?
- **With capability denied**: exec RPCs return code 126 (PASS)

#### Store (`fledge canary store`)
- Basic store/load cycle
- Are oversized keys (>256 bytes) rejected?
- Are oversized values (>64 KB) rejected?
- **With capability denied**: store returns null, load returns null (PASS)

### Baseline Tests (`fledge canary baseline`)

**These are the important ones.** They test what a plugin can do using direct bash access — no RPC, no capabilities needed. This is the real attack surface.

| Category | What It Checks |
|----------|---------------|
| **Init message** | Project name, root path, git branch, and remote URL are always sent to every plugin in the init handshake |
| **Env vars** | Direct `${!varname}` access to `GITHUB_TOKEN`, `OPENAI_API_KEY`, `AWS_SECRET_ACCESS_KEY`, and 7 other common secrets |
| **Filesystem** | `~/.config/fledge/config.toml`, `~/.ssh/`, `fledge.toml` |
| **Credential files** | `~/.aws/credentials`, `~/.docker/config.json`, `~/.kube/config`, `~/.npmrc`, `~/.netrc`, `~/.config/gh/hosts.yml`, `~/.git-credentials`, `~/.gnupg/`, `.env*` files |
| **Shell history** | `~/.zsh_history`, `~/.bash_history`, `~/.local/share/fish/fish_history` — often contain pasted tokens |
| **Clipboard** | `pbpaste` on macOS — may contain copied passwords |
| **Network** | Whether `curl`, `wget`, `dig`, `nslookup` are available for exfiltration |
| **Write access** | Can write to project root, `/tmp` |
| **Persistence** | `.git/hooks/` (inject into commits), `~/.zshrc` (backdoor shell startup), `~/Library/LaunchAgents/` (persistent daemon), `crontab` (scheduled exfiltration) |
| **System recon** | Process list, username, hostname, home directory |

The baseline section ends with a **WASM Sandbox Contrast** showing what the WASM runtime would block for each detected attack.

### WASM Canary

Separate plugin: [fledge-plugin-canary-wasm](https://github.com/CorvidLabs/fledge-plugin-canary-wasm). Runs inside fledge's Wasmtime sandbox and attempts every attack from this native canary's baseline. Every test should report BLOCKED — any LEAKED result indicates a sandbox escape.

### Exposure Report (`fledge canary expose`)

Detailed report showing exactly what data each capability exposes, with masked values. Shows the full attack surface including config file contents, env vars via both metadata RPC and direct shell access, SSH keys, and git credentials.

### Legacy Mode (`fledge canary-legacy`)

Runs without the fledge-v1 protocol — dumps the raw inherited environment (masked values) and checks filesystem access. Shows what a non-protocol plugin sees with zero filtering.

## Reading the Output

- **PASS** / **BLOCKED** — boundary enforced as documented
- **FAIL** / **LEAKED** — boundary not enforced; investigate immediately
- **WARN** — expected behavior that users should understand (e.g., credential files readable without any capability)

**Zero FAILs + some WARNs = your security model is accurately documented.** The WARNs are the honest story — they show what's really possible, not what we wish was possible.

## Example Output (Zero Capabilities)

```
fledge-plugin-canary v0.6.0
Capabilities granted: exec=false store=false metadata=false
Running section: all

=== METADATA CAPABILITY TESTS ===
  (metadata not granted — testing denial)
  ✓ PASS: fledge_config blocked without metadata cap
  ✓ PASS: Env vars blocked without metadata cap
  ✓ PASS: Git metadata blocked without metadata cap

=== EXEC CAPABILITY TESTS ===
  (exec not granted — testing denial)
  ✓ PASS: Exec blocked without exec cap (code 126)

=== STORE CAPABILITY TESTS ===
  (store not granted — testing denial)
  ✓ PASS: Store blocked without store cap (load returns null)

=== BASELINE TESTS — What ANY plugin gets without capabilities ===
  These tests use direct bash access, NOT the fledge-v1 RPC.
  Capabilities only gate protocol responses — the process itself is unsandboxed.

  ...WARNs for each accessible credential file, history, persistence vector...

  ── WASM SANDBOX CONTRAST ──
  A WASM plugin (runtime="wasm") with the same code would see:

  Filesystem (WASI preopens enforce boundaries):
    NATIVE: ~/.ssh/ READABLE            → WASM: BLOCKED (no preopened dir for ~)
    NATIVE: ~/.config/fledge/ READABLE   → WASM: BLOCKED (outside sandbox)
    ...

  Environment Variables (WASM guest has empty env):
    NATIVE: GITHUB_TOKEN LEAKED          → WASM: BLOCKED (not passed to guest)
    ...

  ── TAKEAWAY ──
  Capabilities gate the fledge-v1 RPC only. The plugin process itself
  is an unsandboxed subprocess with full user access.
  A malicious plugin needs ZERO capabilities to steal data.

=== SUMMARY ===
  16 tests: 12 passed, 0 failed, 4 warnings
```

## Why This Exists

Plugin systems are trust boundaries. Rather than claiming security properties and hoping they hold, this plugin verifies them from the inside.

The native canary proves the attacks work. The WASM canary proves the sandbox stops them. Together they validate fledge's security model end-to-end.

Run this after any change to:
- Plugin protocol or capability gating
- Env var filtering logic
- The `exec` cwd validation
- Plugin installation or trust model
- WASM runtime or WASI configuration

## License

MIT — same as fledge.
