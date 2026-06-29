---
name: fix-brew-node-dylib-mismatch
description: Fix missing .dylib libraries when brew upgrades break Node.js browser tooling — create symlinks from old version numbers to new.
tags: [brew, node, dylib, browser, troubleshooting, macos]
---

# 🔧 Fix Brew Node .dylib Mismatch

## When to Use

Hermes Agent's browser tools (`browser_navigate`, `browser_console`, etc.) fail with errors like:

```
dyld[NNN]: Library not loaded: /opt/homebrew/opt/simdjson/lib/libsimdjson.29.dylib
  Referenced from: .../node@22/22.22.0/bin/node
  Reason: tried: .../libsimdjson.29.dylib' (no such file)
```

This happens when `brew` upgrades a library (e.g. `simdjson`, `simdutf`, `icu4c`, `libuv`) to a newer major version, but the old `node@22` binary still expects the old version number. The symlink created during the brew install points to the old Cellar directory that was removed during cleanup.

## Symptoms

- All `browser_navigate` / `browser_snapshot` / `browser_console` calls fail with the same `dyld` error
- Running `node --version` returns a different version than the one giving errors (e.g. node v26 works but node@22 v22.22.0 is broken)
- The error mentions a specific `.dylib` version number (e.g. `.29`, `.31`)

## Step-by-Step Fix

### 1. Identify the Missing Library

```
dyld[NNN]: Library not loaded: /opt/homebrew/opt/simdjson/lib/libsimdjson.29.dylib
```

The key info is:
- **Library name**: `simdjson` (or `simdutf`, `icu4c`, etc.)
- **Expected version**: `.29` (from `libsimdjson.29.dylib`)

### 2. Check Current Versions

```bash
# See what versions exist now
ls /opt/homebrew/opt/simdjson/lib/libsimdjson* 2>/dev/null
ls /opt/homebrew/Cellar/simdjson/*/lib/libsimdjson* 2>/dev/null

# Check what node@22 actually links against
otool -L /opt/homebrew/Cellar/node@22/22.22.0/bin/node | grep -E "(simdjson|simdutf|icu4c)"
```

### 3. Create Compatibility Symlink in TWO Locations

Both locations are needed because different tools look in different paths:

```bash
# Path 1: /opt/homebrew/opt/ symlink (used by the launcher)
ln -sf /opt/homebrew/opt/simdjson/lib/libsimdjson.33.dylib /opt/homebrew/opt/simdjson/lib/libsimdjson.29.dylib

# Path 2: /opt/homebrew/Cellar/ symlink (used by the direct binary)
ln -sf /opt/homebrew/Cellar/simdjson/4.6.4/lib/libsimdjson.33.dylib /opt/homebrew/Cellar/simdjson/4.6.4/lib/libsimdjson.29.dylib
```

The pattern is:
```
ln -sf <CURRENT_LIB> <EXPECTED_LIB_NAME>
```

Where `<CURRENT_LIB>` is the latest version (e.g. `libsimdjson.33.dylib`) and `<EXPECTED_LIB_NAME>` is the version the old node expects (e.g. `libsimdjson.29.dylib`).

### 4. Verify

```bash
# The browser should now work
browser_navigate(url="https://example.com")
```

## Known Libraries That Have Caused This

| Library | Old Version | New Version | Check command |
|---------|-------------|-------------|---------------|
| simdjson | .29 | .33 | `ls /opt/homebrew/opt/simdjson/lib/libsimdjson*` |
| simdutf | .31 | .34 | `ls /opt/homebrew/opt/simdutf/lib/libsimdutf*` |

## Root Cause

When `brew reinstall` or `brew upgrade` runs for a library that `node@22` depends on:
1. The old version's `.dylib` files are removed from the Cellar
2. The new version has a different `.dylib` number (e.g. `.29` → `.33`)
3. `node@22` was compiled against the old number and refuses to load the new one
4. Since `node@22` is pinned at v22.22.0 (not upgraded to v26), it never gets relinked

## Pitfalls

- **Do NOT use `sudo`** — The symlinks can be created in `/opt/homebrew/opt/` and `/opt/homebrew/Cellar/` without sudo
- **Check both locations** — Creating only the `opt` symlink may not fix it if the binary reads from `Cellar` directly
- **Multiple libraries may be broken at once** — After fixing `simdjson`, `simdutf` may also fail. Check `otool -L` for all deps
- **This is a temporary fix** — The real fix is to get the Hermes agent to use the system node v26 instead of the pinned node@22
