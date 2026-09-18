# proxmox-backup-client (Arch Linux PKGBUILD)

Arch Linux `PKGBUILD` that builds the **standalone Proxmox Backup client**
(`proxmox-backup-client` and `pxar`) from the official upstream sources.

This is a continuation of the
[AUR `proxmox-backup-client`](https://aur.archlinux.org/packages/proxmox-backup-client)
package, kept up to date with newer Proxmox Backup Server releases. It exists
because the AUR package is not currently being bumped to the latest versions.
Full credit for the original packaging goes to the upstream maintainers listed
in the `PKGBUILD` header; this repo only tracks new releases on top of their work.

## What you get

- `proxmox-backup-client` — CLI client for Proxmox Backup Server
- `pxar` — the pxar archive tool
- man pages and shell completions (bash + zsh)

Only the client-side crates are built; the server is not.

## Building & installing

```bash
git clone https://github.com/FoxKyong/aur-proxmox-backup-client.git
cd aur-proxmox-backup-client
makepkg -si
```

`makepkg -s` pulls in the build dependencies; `-i` installs the resulting
package. The sources are fetched from `git.proxmox.com` at build time, over
**HTTPS** — the `git://` protocol (TCP 9418) is blocked on many networks. To
use it anyway, set `_gitproto=git` at the top of the `source` block in the
`PKGBUILD`.

## How releases are pinned

Upstream only git-tags releases up to `v4.2.0`; later point releases
(`4.2.1`, `4.2.2`, `4.2.3`, …) exist solely as `debian/changelog` entries and
apt packages, with no matching git tag. Each bump therefore pins:

- **`proxmox-backup`** by its `bump version to X-1` commit (`_pbs_commit`), and
- **`proxmox`** by a `master` commit dated at/just-before that release, chosen
  so its crates satisfy the release's `Cargo.toml` requirements.

Two patches are applied in `prepare()`:

- `0001-…` re-routes the proxmox workspace crates to the local `../proxmox`
  checkout and strips the server package so only the client builds. It may need
  a refresh when a release adds a new dependency to the root `[dependencies]`.
- `0002-…` drops all but the client man pages.

## License

The built software is licensed under **AGPL-3.0** (see upstream). This
repository contains only the packaging recipe; the actual source code is
downloaded from the official Proxmox git servers during the build.
