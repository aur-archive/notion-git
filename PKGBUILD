# Maintainer: M Rawash <mrawash@gmail.com>
# Contributor: Pablo Lezaeta <prflr88 @ gmail [dot] com>
# Contributor: Sergej Pupykin <pupykin.s+arch@gmail.com>
# Contributor: Eugen Zagorodniy e dot zagorodniy at gmail dot com
# Contributor: aunoor

pkgname=notion-git
pkgver=20120717
pkgrel=1
pkgdesc="Tabbed tiling, window manager. Fork of Ion3"
url="http://sourceforge.net/projects/notion/"
arch=('i686' 'x86_64')
license=('custom:LGPL')
depends=('glib2' 'gettext' 'lua' 'libxext' 'libsm')
optdepends=('libxinerama' 'libxrandr')
makedepends=('git' 'pkgconfig' 'libxinerama' 'libxrandr'
	     'rubber' 'latex2html' 'texlive-htmlxml' 'texlive-latexextra')
provides=('libtu' 'libextl' 'notion')
conflicts=('notion')
changelog=ChangleLog

_gitroot="git://notion.git.sourceforge.net/gitroot/notion/notion"
_gitname="notion"

_gitroots=($_gitroot
	   "git://notion.git.sourceforge.net/gitroot/notion/libtu"
	   "git://notion.git.sourceforge.net/gitroot/notion/libextl"
	   "git://notion.git.sourceforge.net/gitroot/notion/notion-doc"
	   "git://notion.git.sourceforge.net/gitroot/notion/mod_xinerama"
	   "git://notion.git.sourceforge.net/gitroot/notion/mod_xkbevents"
	   "git://notion.git.sourceforge.net/gitroot/notion/mod_xrandr"
	   "git://notion.git.sourceforge.net/gitroot/notion/mod_notionflux"
	   "git://notion.git.sourceforge.net/gitroot/notion/contrib")

build() {
  cd ${srcdir}

  # git clone
  for gitroot in ${_gitroots[@]}; do
    msg "Connecting to the git repository..."
    gitname=`basename ${gitroot}`
    if [ -d ${srcdir}/${gitname} ]; then
        pushd ${srcdir}/${gitname}
        git pull origin
        popd
    else
        git clone --depth 1 ${gitroot}
    fi
    msg "GIT checkout done or server timeout"
  done

  # copy to notion-build
  rm -rf ${srcdir}/notion-build
  cp -r  ${srcdir}/notion ${srcdir}/notion-build
  for i in libextl libtu mod_xinerama mod_xkbevents mod_xrandr mod_notionflux notion-doc; do
    cp -r  ${srcdir}/$i ${srcdir}/notion-build/
  done

  # build notion
  cd ${srcdir}/notion-build
  msg "Starting make..."
  sed -e 's/^\(PREFIX=\).*$/\1\/usr/' \
	-e 's/^\(ETCDIR=\).*$/\1\/etc\/notion/' \
	-e 's/^\(LUA_DIR=\).*$/\1\/usr/' \
	-e 's/^\(X11_PREFIX=\).*/\1\/usr/' \
	-i system.mk
  make INCLUDES=-I${srcdir}/notion-build

  # doc workaround
  for i in ioncore mod_tiling mod_query de mod_menu mod_dock mod_sp mod_statusbar; do
    (cd $i && make _exports_doc)
  done

  # build doc and modules
  for i in mod_xinerama mod_xkbevents mod_xrandr mod_notionflux notion-doc; do
    (cd $i && make -j1 TOPDIR=${srcdir}/notion-build all)
  done
}

package() {
  cd ${srcdir}/notion-build

  # notion
  make PREFIX=${pkgdir}/usr ETCDIR=${pkgdir}/etc/notion install

  # modules
  for i in mod_xinerama mod_xkbevents mod_xrandr mod_notionflux notion-doc; do
    (cd $i && make  PREFIX=${pkgdir}/usr ETCDIR=${pkgdir}/etc/notion TOPDIR=${srcdir}/notion-build install)
  done
  cp ${srcdir}/mod_xinerama/*.lua $pkgdir/etc/notion/
  cp ${srcdir}/mod_xkbevents/*.lua $pkgdir/etc/notion/

  # contrib
  mkdir -p $pkgdir/usr/share/notion/contrib
  cp -a ${srcdir}/contrib/* $pkgdir/usr/share/notion/contrib

  # license
  install -Dm0644 LICENSE ${pkgdir}/usr/share/licenses/notion/LICENSE
}
