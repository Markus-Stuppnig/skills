---
name: gondolin
description: "Gondolin VM extension for pi: extra mounts, gondolin.json config, runtime mount command, and privilege escalation prevention. Load when the user asks about file access, missing folders, or when something the user mentioned can't be found and you suspect you have no access to it."
---

# Gondolin VM Extension

## When to Use

- User asks about Gondolin setup or configuration
- User asks why you can't see a folder or project
- You can't find something the user mentioned — suspect a missing mount
- User wants to add a permanent or runtime mount
- User asks about security / privilege escalation of the mount config

## How It Works

The agent runs inside a Gondolin micro-VM (QEMU). **Only explicitly mounted host folders are visible.**

Default mount: `/workspace` → project CWD (where `pi` was started).

### Auto-mounted read-only paths (always present)

| Guest path  | Host path   | Notes                              |
|-------------|-------------|------------------------------------|
| `/home-pi`  | `~/.pi`     | Read-only view of the pi home folder |
| `/pi-docs`  | pi-coding-agent package root | Read-only; docs, examples, README |

To upgrade `/home-pi` to writable, ask the user to run:
```
/gondolin-mount ~/.pi /home-pi
```
This replaces the read-only auto-mount with a writable one.

## Permanent Extra Mounts: `.pi/gondolin.json`

Create `.pi/gondolin.json` in the project root (the folder where `pi` is started):

```json
{
  "mounts": {
    "/backend": "/Users/markus/projects/backend",
    "/docs": "../../shared-docs",
    "/secrets": { "host": "/Users/markus/secrets", "readonly": true }
  }
}
```

- Guest paths must be absolute (start with `/`)
- Host paths can be absolute or relative to the project root
- Value can be a plain string (read-write) or `{ "host": "...", "readonly": true }` for read-only
- Applied on every session start

## Runtime Mount Command

To mount a folder during an active session (user-only pi command, agent cannot call this):

```
/gondolin-mount <host-path> [guest-path] [--ro]
```

- `guest-path` is optional — defaults to `/<basename-of-host-path>`
- `--ro` makes the mount read-only (wraps provider in `ReadonlyProvider`)
- The VM restarts automatically to apply the new mount
- Runtime mounts survive within the session but are lost when pi exits
- **A host path can only be mounted once.** Remounting the same host path replaces the existing entry (with a warning if the guest path differs).

**Examples:**
```
/gondolin-mount /Users/markus/projects/backend /backend
/gondolin-mount ../../other-project
/gondolin-mount /Users/markus/secrets /secrets --ro
/gondolin-mount ~/.pi /home-pi        # upgrades auto-mount to writable
```

## Status Command

```
/gondolin
```
Shows VM id, host/guest workspace, shell, and all active runtime mounts (with `(ro)` suffix for read-only ones).

## Privilege Escalation Prevention

`.pi/gondolin.json` is wrapped in a `ShadowProvider` with `writeMode: "tmpfs"`:
- Agent can **read** the config
- Agent **cannot persist writes** to it — any changes vanish at session end
- Agent therefore cannot grant itself new mounts by modifying the file
- Symlink bypass is also blocked (`denySymlinkBypass: true` by default)

## Mount Priority (lowest → highest)

1. Auto-mounts (`/home-pi`, `/pi-docs`)
2. `gondolin.json` mounts
3. Runtime mounts (via `/gondolin-mount`)

A higher-priority mount at the same guest path silently overrides a lower one.

## Pitfalls

- `MountRouterProvider` is immutable — `/gondolin-mount` restarts the VM to apply changes
- The agent **cannot** call `/gondolin-mount` — it is a user-only command
- Relative host paths in `gondolin.json` resolve relative to the project root, not the shell CWD
- Runtime mounts are ephemeral — add to `gondolin.json` for persistence
- A host path can only appear once across all runtime mounts; attempting to mount it again replaces the old entry

## Building Guest Assets

When rootfs packages need to change (e.g. adding Java, Node, Python), rebuild the guest assets from the host terminal:

1. **Check the Alpine package name first**: https://pkgs.alpinelinux.org/packages?branch=v3.23&arch=aarch64
2. **Edit** `extensions/gondolin/build-config.json` → update `rootfsPackages`
3. **Build**:
   ```bash
   cd /Users/markus.stuppnig/.pi/agent/extensions/gondolin
   npx gondolin build --config build-config.json --output guest-assets
   ```
4. **Watch for warnings**: `unable to resolve dependency` = wrong package name for Alpine 3.23
5. **Restart pi** to load the new rootfs

### Pitfalls

- `npx gondolin-build` → **404 error** — the binary is `gondolin`, not `gondolin-build`
- `npx gondolin build` without `--config` → reads wrong defaults, ignores `build-config.json`
- `openjdk26` does not exist in Alpine 3.23 — use `openjdk25`
- Assets only take effect after restarting pi
- Verify: check `guest-assets/manifest.json` → `rootfsPackages` must list the new package

## Verification

```bash
ls /                  # all guest mount paths should appear
```
Run `/gondolin` in pi to confirm VM status and active mounts.
