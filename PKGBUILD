# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Maintainer: Fabian Bornschein <fabiscafe@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>

pkgbase=glib2
pkgname=(
  glib2
  glib2-devel
  glib2-docs
)
pkgver=2.82.4
pkgrel=2
pkgdesc="Low level core library"
url="https://gitlab.gnome.org/GNOME/glib"
license=(LGPL-2.1-or-later)
arch=('x86_64' 'aarch64')
depends=(
  bash
  glibc
  libffi
  libsysprof-capture
  pcre2
  util-linux-libs
  zlib
)
makedepends=(
  dbus
  dconf
  gettext
  gi-docgen
  git
  gobject-introspection
  libelf
  meson
  python
  python-docutils
  python-packaging
  shared-mime-info
  util-linux
)
checkdepends=(
  desktop-file-utils
  glib2
)
source=(
  "https://github.com/GNOME/glib/archive/refs/tags/$pkgver.tar.gz"
  "https://github.com/GNOME/gvdb/archive/0854af0fdb6d527a8d1999835ac2c5059976c210.tar.gz"
  0001-glib-compile-schemas-Remove-noisy-deprecation-warnin.patch
  gio-querymodules.hook
  glib-compile-schemas.hook
)
b2sums=('SKIP'
        'SKIP'
        '47cd08ba7e4b3ca0cd19f6dc20e4d73e30cf90f2b78c3d620ee0c7a4d8a4b325a5e88ec2dcc3a63402c16cc1ce8061130afc313e3cbfcd220dff3e642b113a69'
        '14c9211c0557f6d8d9a914f1b18b7e0e23f79f4abde117cb03ab119b95bf9fa9d7a712aa0a29beb266468aeb352caa3a9e4540503cfc9fe0bbaf764371832a96'
        'acc2f474139e535f4bdd70ac22a9150f786b3395e679b14d0d3fbb9361d511bb1b5069d95b2a7ac9c0f3d901b03a0c037eb273446ba00764191b30a777bd2bc9')
validpgpkeys=(
  53EF3DC3B63E2899271BD26322E8091EEA11BBB7 # Emmanuele Bassi <ebassi@gnome.org>
  923B7025EE03C1C59F42684CF0942E894B2EAFA0 # Philip Withnall <pwithnall@gnome.org>
)

prepare() {
  cd glib-$pkgver

  # Drop dep on libatomic
  # https://gitlab.archlinux.org/archlinux/packaging/packages/qemu/-/issues/6
#  git revert -n 4e6dc4dee0e1c6407113597180d9616b4f275f94

  # Suppress noise from glib-compile-schemas.hook
#  git apply -3 ../0001-glib-compile-schemas-Remove-noisy-deprecation-warnin.patch

#  git submodule init
#  git submodule set-url subprojects/gvdb "$srcdir/gvdb"
 cp -r $srcdir/gvdb-0854af0fdb6d527a8d1999835ac2c5059976c210/* subprojects/gvdb/
#  git -c protocol.file.allow=always -c protocol.allow=never submodule update
}

build() {
  local meson_options=(
    --default-library both
    -D documentation=true
    -D dtrace=disabled
    -D glib_debug=disabled
    -D introspection=enabled
    -D man-pages=enabled
    -D selinux=disabled
    -D sysprof=enabled
    -D systemtap=disabled
  )

  # Produce more debug info: GLib has a lot of useful macros
  CFLAGS+=" -g3"
  CXXFLAGS+=" -g3"

  # use fat LTO objects for static libraries
  CFLAGS+=" -ffat-lto-objects"
  CXXFLAGS+=" -ffat-lto-objects"

  arch-meson glib-$pkgver build "${meson_options[@]}"
  meson compile -C build
}

check() {
  meson test -C build --no-suite flaky --no-suite slow --print-errorlogs
}

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

package_glib2() {
  depends+=(
    libffi.so
    libmount.so
  )
  provides+=(libg{lib,io,irepository,module,object,thread}-2.0.so)
  optdepends=(
    'dconf: GSettings storage backend'
    'glib2-devel: development tools'
    'gvfs: most gio functionality'
  )
  options+=(staticlibs)
  meson install -C build --destdir "$pkgdir"

  install -Dt "$pkgdir/usr/share/libalpm/hooks" -m644 *.hook
  touch "$pkgdir/usr/lib/gio/modules/.keep"

  python -m compileall -d /usr/share/glib-2.0/codegen \
    "$pkgdir/usr/share/glib-2.0/codegen"
  python -O -m compileall -d /usr/share/glib-2.0/codegen \
    "$pkgdir/usr/share/glib-2.0/codegen"

  cd "$pkgdir"

  # Split docs
  _pick docs usr/share/doc

  # Split devel
  _pick devel usr/bin/gdbus-codegen
  _pick devel usr/bin/glib-{mkenums,genmarshal}
  _pick devel usr/bin/gresource
  _pick devel usr/bin/gtester{,-report}

  _pick devel usr/share/gdb/
  _pick devel usr/share/glib-2.0/gdb/
  _pick devel usr/share/glib-2.0/codegen/

  _pick devel usr/share/bash-completion/completions/gresource

  _pick devel usr/share/man/man1/gdbus-codegen.1
  _pick devel usr/share/man/man1/glib-{mkenums,genmarshal}.1
  _pick devel usr/share/man/man1/gresource.1
  _pick devel usr/share/man/man1/gtester{,-report}.1
}

package_glib2-devel() {
  pkgdesc+=" - development files"
  depends=(
    glib2
    glibc
    libelf
    python
    python-packaging
  )
  mv devel/* "$pkgdir"
}

package_glib2-docs() {
  pkgdesc+=" - documentation"
  depends=()
  license+=(LicenseRef-Public-Domain)
  cd glib-$pkgver
  mv docs/* "$pkgdir"
  install -Dt "$pkgdir/usr/share/licenses/$pkgname" -m644 docs/reference/COPYING
}

# vim:set sw=2 sts=-1 et:
