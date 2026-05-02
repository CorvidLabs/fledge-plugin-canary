# fledge-plugin-canary

First-party security audit tool for [fledge](https://github.com/CorvidLabs/fledge). Probes plugin capability boundaries and reports what data a plugin can access at each permission level.

**This is not a malicious plugin.** It's a canary-in-the-coalmine that validates fledge's security claims by testing them from the inside.

## Install

```bash
fledge plugins install CorvidLabs/fledge-plugin-canary
# Grant all three capabilities when prompted — that's the point
```

## Usage

```bash
# Run all tests
fledge canary

# Run a specific section
fledge canary metadata
fledge canary exec
fledge canary store

# Legacy mode — shows unfiltered env inheritance
fledge canary-legacy
```

## What it tests

### Metadata capability
- Can the plugin read `fledge.toml`? Does it contain secrets?
- Which env vars pass through the metadata RPC filter?
- Are sensitive patterns (`TOKEN`, `SECRET`, `KEY`, etc.) properly blocked?
- Is git metadata (status, log, tags) accessible?

### Exec capability
- Can the plugin read `~/.config/fledge/config.toml` (global config with tokens)?
- Can it access files outside the project root?
- Does cwd path traversal (`../../..`) get blocked by `canonicalize()`?
- Which env vars are visible in the exec shell environment?

### Store capability
- Does basic store/load work?
- Are oversized keys (>256 bytes) rejected?
- Are oversized values (>64 KB) rejected?

### Legacy mode (`canary-legacy`)
- Dumps the full inherited environment (masked values)
- Shows what a non-protocol plugin sees without any filtering
- Checks filesystem access to global and project config

## Reading the output

- **PASS** — boundary enforced as documented
- **FAIL** — boundary not enforced; security docs need updating
- **WARN** — expected behavior that users should understand (e.g., `exec` = full shell access)

Zero FAILs + some WARNs = your security docs are accurate. The WARNs tell you what to be transparent about.

## Why this exists

Plugin systems are trust boundaries. Rather than claiming security properties and hoping they hold, this plugin verifies them. Run it after any change to the plugin protocol, capability gating, or env filtering code.
