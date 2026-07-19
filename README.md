# ish-AOK-rootfs

Manifest of downloadable root filesystems offered by [iSH-AOK](https://github.com/emkey1/ish-AOK)'s
"New Filesystem" picker. This repo is a git submodule of iSH-AOK
(`deps/rootfs-manifest`); the app bundles `manifest.json` at build time and
merges it with the small set of filesystems shipped inside the IPA itself.

This repo exists so anyone can propose a new rootfs without touching the
iSH-AOK application source.

## Contributing a new rootfs

1. Build (or otherwise obtain) a `.tar.xz` (preferred; `.tar.gz`/`.tar.zst`/`.tar.bz2` also work)
   root filesystem archive for a Linux guest architecture iSH-AOK supports:
   `i386`, `amd64` (x86_64), `arm64` (aarch64), or `riscv64`.
2. Host the archive somewhere stable and directly downloadable over HTTPS
   (a GitHub Release asset on your own fork/repo is the easiest option —
   avoid links that require auth, redirect through an HTML landing page, or
   expire).
3. Add an entry to `manifest.json` (see schema below) and open a pull
   request. Set `"tier": "community"` — only iSH-AOK maintainers promote
   entries to `"official"`, which implies the image is covered by the
   project's regression suite.
4. In your PR description, note what you tested (does `/bin/login` work,
   does the package manager work, any known issues).

Once merged here, a maintainer bumps the `deps/rootfs-manifest` submodule
pointer in iSH-AOK and the new choice ships in the next app release.

## `manifest.json` schema

A JSON array of objects. All fields are required strings unless noted.

| Field | Description |
|---|---|
| `identifier` | Stable, unique, machine-readable id (letters/digits only, no spaces). Never reuse or change an existing identifier — it's persisted in exported root metadata. |
| `displayName` | Shown in the picker row when architecture isn't grouped (fallback / accessibility label). Keep it short; put "(Experimental)" style caveats here, not in `familyDisplayName`. |
| `archiveName` | Base filename (no extension) the downloaded archive is renamed to on-device. Should be unique per identifier. |
| `importName` | Becomes the on-device root directory name and mount "source" string. Must match iSH-AOK's `RootNameIsValid` rules: letters, digits, `.`, `-`, `_` only, no spaces, can't start with `.`. |
| `initialWindow` | Currently always `"session-shell"`. |
| `guestABI` | One of `i386`, `amd64`, `arm64`, `riscv64` — must match the archive's actual userland architecture. |
| `downloadURL` | Direct HTTPS URL to the `.tar.*` archive. Must return the file directly (no HTML interstitial). |
| `downloadSize` | Human-readable approximate download size, e.g. `"~32 MB"`. Shown to the user before they download. |
| `family` | Groups architecture variants of the same distro/release into one picker row (e.g. all three Alpine 3.23.3 entries share `"alpine3233"`). Use a new family value per distro *and* per release/version you want listed separately. |
| `familyDisplayName` | Shown for the grouped row; architecture is chosen as a sub-choice. |
| `tier` | `"official"` or `"community"`. New PRs should use `"community"`. |

Keep `identifier`/`archiveName`/`importName` free of ambiguity with existing
entries in this file — the app dedupes by `identifier`.
