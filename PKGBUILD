# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/qt5-base
# 						Maintainer: Felix Yan <felixonmars@archlinux.org>
# 						Contributor: Andrea Scarpino <andrea@archlinux.org

pkgbase=qt5-base
pkgname=(qt5-base qt5-xcb-private-headers)
_qtver=5.10.1
pkgver=${_qtver/-/}
pkgrel=3
arch=(x86_64)
url='http://qt-project.org/'
license=('GPL3' 'LGPL3' 'FDL' 'custom')
pkgdesc='A cross-platform application and UI framework'
depends=('libjpeg-turbo' 'xcb-util-keysyms' 'xcb-util-renderutil' 'libgl' 'fontconfig' 'xdg-utils'
         'xcb-util-wm' 'libxrender' 'libxi' 'sqlite' 'xcb-util-image' 'icu' 'pcre2'
         'tslib' 'libinput' 'libsm' 'libxkbcommon-x11' 'libproxy' 'libcups' 'double-conversion')
makedepends=('libfbclient' 'libmariadbclient' 'sqlite' 'unixodbc' 'postgresql-libs' 'alsa-lib' 'gst-plugins-base-libs'
			'gtk3' 'libpulse' 'cups' 'freetds' 'vulkan-headers')
optdepends=('qt5-svg: to use SVG icon themes'
            'postgresql-libs: PostgreSQL driver'
            'libmariadbclient: MariaDB driver'
            'unixodbc: ODBC driver'
            'libfbclient: Firebird/iBase driver'
            'freetds: MS SQL driver'
            'gtk3: GTK platform plugin')
conflicts=('qtchooser')
groups=('qt' 'qt5')
_pkgfqn="${pkgbase/5-/}-everywhere-src-${_qtver}"
source=("http://download.qt.io/official_releases/qt/${pkgver%.*}/${_qtver}/submodules/${_pkgfqn}.tar.xz"
		"revert-Set-sharedPainter-correctly-for-QGraphicsEffect.patch")
sha256sums=('d8660e189caa5da5142d5894d328b61a4d3ee9750b76d61ad74e4eee8765a969')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

prepare() {
  cd ${_pkgfqn}

  # Build qmake using Arch {C,LD}FLAGS
  # This also sets default {C,CXX,LD}FLAGS for projects built using qmake
  sed -i -e "s|^\(QMAKE_CFLAGS_RELEASE.*\)|\1 ${CFLAGS}|" \
    mkspecs/common/gcc-base.conf
  sed -i -e "s|^\(QMAKE_LFLAGS_RELEASE.*\)|\1 ${LDFLAGS}|" \
    mkspecs/common/g++-unix.conf

  # Use python2 for Python 2.x
  find . -name '*.py' -exec sed -i \
    's|#![ ]*/usr/bin/python$|&2|;s|#![ ]*/usr/bin/env python$|&2|' {} +
    
  # Fix missing private includes https://bugreports.qt.io/browse/QTBUG-37417
  sed -e '/CMAKE_NO_PRIVATE_INCLUDES\ \=\ true/d' -i mkspecs/features/create_cmake.prf
  
  # Revert upstream commit which breaks some Deepin components (FS#57531)
  # https://bugreports.qt.io/browse/QTBUG-66226
  patch -Np1 -i ../revert-Set-sharedPainter-correctly-for-QGraphicsEffect.patch
}

build() {
  cd ${_pkgfqn}

  # FS#38796
  #[[ "${CARCH}" = "i686" ]] && SSE2="-no-sse2"

  echo "INCLUDEPATH += /usr/include/openssl-1.0" >> src/network/network.pro
  export OPENSSL_LIBS='-L/usr/lib/openssl-1.0 -lssl -lcrypto'

  PYTHON=/usr/bin/python2 ./configure -confirm-license -opensource -v \
    -prefix /usr \
    -docdir /usr/share/doc/qt \
    -headerdir /usr/include/qt \
    -archdatadir /usr/lib/qt \
    -datadir /usr/share/qt \
    -sysconfdir /etc/xdg \
    -examplesdir /usr/share/doc/qt/examples \
    -plugin-sql-{psql,mysql,sqlite,odbc,ibase} \
    -system-sqlite \
    -openssl-linked \
    -nomake examples \
    -no-rpath \
    -optimized-qmake \
    -dbus-linked \
    -system-harfbuzz \
    -no-use-gold-linker \
    -reduce-relocations

  make
}

package_qt5-base() {
  
  pkgdesc='A cross-platform application and UI framework'
  
  cd ${_pkgfqn}
  make INSTALL_ROOT="${pkgdir}" install

  install -D -m644 LGPL_EXCEPTION.txt \
    "${pkgdir}"/usr/share/licenses/${pkgbase}/LGPL_EXCEPTION.txt

  # Drop QMAKE_PRL_BUILD_DIR because reference the build dir
  find "${pkgdir}/usr/lib" -type f -name '*.prl' \
    -exec sed -i -e '/^QMAKE_PRL_BUILD_DIR/d' {} \;

  # Fix wrong qmake path in pri file
  sed -i "s|${srcdir}/${_pkgfqn}|/usr|" \
    "${pkgdir}"/usr/lib/qt/mkspecs/modules/qt_lib_bootstrap_private.pri

   # Symlinks for backwards compatibility
  for b in "${pkgdir}"/usr/bin/*; do
    ln -s /usr/bin/$(basename $b) "${pkgdir}"/usr/bin/$(basename $b)-qt5
  done
}
package_qt5-xcb-private-headers() {

  pkgdesc='Private headers for Qt5 Xcb'

  depends=("qt5-base=$pkgver")
  optdepends=()
  groups=()
  conflicts=()

  cd ${_pkgfqn}
  install -d -m755 "$pkgdir"/usr/include/qtxcb-private
  cp -r src/plugins/platforms/xcb/*.h "$pkgdir"/usr/include/qtxcb-private/
}
