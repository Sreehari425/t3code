# Maintainer: Your Name <youremail@domain.com>
# Reference: https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=t3-code-git

pkgname=t3-code-git
pkgver=0.1.6.r36.g825a8a2
pkgrel=1
pkgdesc='Minimal web GUI for coding agents (desktop app, git build)'
arch=('x86_64')
url='https://github.com/pingdotgg/t3code'
license=('MIT')
options=('!strip' '!debug')
provides=('t3-code')
conflicts=('t3-code')
depends=('fuse2' 'gtk3' 'nss' 'alsa-lib')
makedepends=('git' 'nodejs' 'npm' 'pnpm' 'cargo' 'rust' 'imagemagick' 'python' 'gcc' 'make')

_gitrepo="https://github.com/Sreehari425/t3code.git"
_gitbranch="main" # e.g. branch=main or branch=cliproxyapi-support-prs

source=(
  "t3code::git+${_gitrepo}${_gitbranch:+#branch=${_gitbranch}}"
  "t3-code.desktop"
  "t3-code.sh"
)
sha256sums=('SKIP' 'SKIP' 'SKIP')

pkgver() {
  cd "${srcdir}/t3code"
  local desc rest count hash tag

  desc=$(git describe --long --tags --abbrev=7 --exclude='nightly*' 2>/dev/null) || {
    printf '0.0.0.r%s.g%s' "$(git rev-list --count HEAD)" "$(git rev-parse --short=7 HEAD)"
    return
  }

  hash=${desc##*-g}
  rest=${desc%-g*}
  count=${rest##*-}
  tag=${rest%-${count}}
  tag=${tag%-}
  tag=${tag#nightly-v}
  tag=${tag#v}
  tag=${tag//-/.}

  printf '%s.r%s.g%s' "${tag}" "${count}" "${hash}"
}

prepare() {
  # Fallback: create companion files if makepkg was invoked without them in srcdir
  if [ ! -f "${srcdir}/t3-code.desktop" ]; then
    cat << 'EOF' > "${srcdir}/t3-code.desktop"
[Desktop Entry]
Name=T3 Code
Comment=Minimal web GUI for coding agents
Exec=t3-code %U
Terminal=false
Type=Application
Icon=t3-code
Categories=Development;IDE;
StartupWMClass=t3code
EOF
  fi

  if [ ! -f "${srcdir}/t3-code.sh" ]; then
    cat << 'EOF' > "${srcdir}/t3-code.sh"
#!/usr/bin/env bash
set -euo pipefail

exec /opt/t3-code/t3-code.AppImage "$@"
EOF
    chmod +x "${srcdir}/t3-code.sh"
  fi
}

build() {
  # Ensure Vite+ (vp) is available in PATH.
  # If not installed globally, install vite-plus into a local prefix within srcdir.
  if ! command -v vp &>/dev/null; then
    export npm_config_prefix="${srcdir}/.npm-global"
    export PATH="${srcdir}/.npm-global/bin:${PATH}"
    if [ ! -x "${srcdir}/.npm-global/bin/vp" ]; then
      echo "==> Installing vite-plus (vp) in local build environment..."
      npm install -g vite-plus
    fi
  fi

  cd "${srcdir}/t3code"
  vp i --frozen-lockfile
  vp run dist:desktop:linux
}

package() {
  cd "${srcdir}/t3code"

  install -d "${pkgdir}/opt/t3-code"
  install -m755 "${srcdir}/t3code/release/"*.AppImage "${pkgdir}/opt/t3-code/t3-code.AppImage"

  install -d "${pkgdir}/usr/bin"
  install -m755 "${srcdir}/t3-code.sh" "${pkgdir}/usr/bin/t3-code"

  install -d "${pkgdir}/usr/share/applications"
  install -m644 "${srcdir}/t3-code.desktop" "${pkgdir}/usr/share/applications/t3-code.desktop"

  install -d "${pkgdir}/usr/share/pixmaps"
  install -m644 "${srcdir}/t3code/assets/prod/black-universal-1024.png" "${pkgdir}/usr/share/pixmaps/t3-code.png"

  install -d "${pkgdir}/usr/share/licenses/${pkgname}"
  install -m644 "${srcdir}/t3code/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
