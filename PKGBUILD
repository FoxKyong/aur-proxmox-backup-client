# Maintainer: Dominik Csapak <dominik.csapak@gmail.com>
# Maintainer: Thomas Lamprecht <thomas@lamprecht.org>
# Contributor: Radim Vančo (FoxKyong) <radim.vanco@jifox.cz>
pkgname=proxmox-backup-client
pkgver=4.2.6
pkgrel=1
# NOTE: upstream only tags releases up to v4.2.0; 4.2.1 and later exist solely
# as debian/changelog entries (and apt packages), so there is no v4.2.6 git tag.
# Pin the proxmox-backup source by the "bump version to 4.2.6-1" commit instead.
_pbs_commit=a51146896605e2f34d0bf6239c10004a96d08e9b
pkgdesc="Client for Proxmox Backup Server"
arch=('x86_64' 'aarch64')
url="https://pbs.proxmox.com"
license=('AGPL3')
depends=(
    'acl'
    'fuse3'
    'gcc-libs'
    'openssl'
)
makedepends=('cargo' 'clang' 'git' 'llvm' 'patchelf' 'python-docutils' 'python-sphinx')
options=(!lto)
# VCS prefix used for the proxmox git repos below. The default git+https
# works everywhere; the git:// protocol (TCP 9418) is blocked on many
# networks. Set to plain "git" to use it anyway.
_gitproto=git+https
source=(
    "$pkgname-$pkgver::${_gitproto}://git.proxmox.com/git/proxmox-backup.git#commit=$_pbs_commit"
    "proxmox::${_gitproto}://git.proxmox.com/git/proxmox.git#commit=382db6bfbb1b499ed51102a2652d1f0620e1d704"
    "proxmox-fuse::${_gitproto}://git.proxmox.com/git/proxmox-fuse.git"
    "pxar::${_gitproto}://git.proxmox.com/git/pxar.git"
    "pathpatterns::${_gitproto}://git.proxmox.com/git/pathpatterns.git"
    "0001-re-route-dependencies-not-available-on-crates.io-to-.patch"
    "0002-docs-drop-all-but-client-man-pages.patch"
    "elf-strip-unused-dependencies.sh"
)
# either a git repo or tracked by this git repo, so not much gained by encoding
# checksums here in this git repo
sha512sums=(
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
    'SKIP'
)

_apply() {
  echo "applying patch '$1'"
  patch --forward --strip=1 --input="${srcdir}/$1"
}

prepare() {
  cd "$pkgname-$pkgver"
  rm .cargo/config.toml # drop Debian-style redirect of crates.io to local registry

  _apply 0001-re-route-dependencies-not-available-on-crates.io-to-.patch
  _apply 0002-docs-drop-all-but-client-man-pages.patch

  # fetch all in prepare to allow build() to be run offline
  cargo fetch --target "$CARCH-unknown-linux-gnu"
}

build() {
  cd "$pkgname-$pkgver"

  cargo build --release \
    --package proxmox-backup-client \
      --bin proxmox-backup-client \
      --bin dump-catalog-shell-cli \
    --package pxar-bin \
      --bin pxar \
    ;

  # fixup rust linking "feature" which links in all dependencies somewhere used
  # in the crate, even if not referenced at all in this binary...
  "${srcdir}/elf-strip-unused-dependencies.sh" "target/release/proxmox-backup-client"
  "${srcdir}/elf-strip-unused-dependencies.sh" "target/release/pxar"

  cd docs

  DEB_VERSION_UPSTREAM="$pkgver" DEB_VERSION="${pkgver%.*}" BUILD_MODE=release make proxmox-backup-client.1 pxar.1
}

check() {
  cd "$pkgname-$pkgver"

  mkdir -p target/testout/
  cargo test --release \
    --package proxmox-backup-client \
      --bin proxmox-backup-client \
    --package pxar-bin \
      --bin pxar \
    ;
}

package() {
  cd "$pkgname-$pkgver"

  install -Dm755 "target/release/proxmox-backup-client" "$pkgdir/usr/bin/proxmox-backup-client"
  install -Dm755 "target/release/pxar" "$pkgdir/usr/bin/pxar"

  install -Dm644 "docs/output/man/proxmox-backup-client.1" "$pkgdir/usr/share/man/man1/proxmox-backup-client.1"
  install -Dm644 "docs/output/man/pxar.1" "$pkgdir/usr/share/man/man1/pxar.1"

  install -Dm644 "debian/proxmox-backup-client.bc" "$pkgdir/usr/share/bash-completion/completions/proxmox-backup-client"
  install -Dm644 "debian/pxar.bc" "$pkgdir/usr/share/bash-completion/completions/pxar"

  install -Dm644 "zsh-completions/_proxmox-backup-client" "$pkgdir/usr/share/zsh/site-functions/_proxmox-backup-client"
  install -Dm644 "zsh-completions/_pxar" "$pkgdir/usr/share/zsh/site-functions/_pxar"
}
