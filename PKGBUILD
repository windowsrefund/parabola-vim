# Just plain vim

pkgbase=vim
pkgname=('vim' 'vim-runtime')
_topver=7.4
_patchlevel=542
_tag=v${_topver/./-}-${_patchlevel}
_versiondir="vim${_topver//./}"
pkgver=${_topver}.${_patchlevel}
pkgrel=1
arch=('i686' 'x86_64')
groups=('modified')
license=('custom:vim')
url="http://www.vim.org"
makedepends=('python2' 'libxt')
source=("${pkgbase}-repo::hg+https://vim.googlecode.com/hg#tag=${_tag}"
        'vimrc'
        'parabola.vim'
        'gvim.desktop')
sha1sums=('SKIP'
          '7bacf26cb66f6c36184a62bc306ef33bfe892686'
          'a72ca0f8d941ff221598091338d9c1bf75a3494b'
          '4a579cf66590d711f49c5dfb4a25e5df116ff7ba')

prepare() {
  cd ${pkgbase}-repo

  _latesttag=$(hg parents --template '{latesttag}' -r default)
  if (( $_tag != $_latesttag )); then
    printf 'You are not building the latest revision!\n'
    printf "Consider updating to tag $_latesttag.\n"
  fi

  # define the place for the global (g)vimrc file (set to /etc/vimrc)
  sed -i 's|^.*\(#define SYS_.*VIMRC_FILE.*"\) .*$|\1|' \
    src/feature.h
  sed -i 's|^.*\(#define VIMRC_FILE.*"\) .*$|\1|' \
    src/feature.h

  (cd src && autoconf)

  cd ..
  for pkg in ${pkgname[@]}
  do
    cp -a ${pkgbase}-repo ${pkg}-build
  done
}

build() {
  cd "${srcdir}"/vim-runtime-build

  ./configure \
    --prefix=/usr \
    --localstatedir=/var/lib/vim \
    --with-features=normal \
    --with-compiledby=akosmin \
    --with-x=yes \
    --enable-multibyte \
    --disable-gui \
    --disable-signs \
    --disable-netbeans

  make

  cd "${srcdir}"/vim-build

  ./configure \
    --prefix=/usr \
    --localstatedir=/var/lib/vim \
    --with-features=normal \
    --with-compiledby=akosmin \
    --with-x=yes \
    --enable-multibyte \
    --disable-gui \
    --disable-signs \
    --disable-netbeans

  make
}

check() {
  # disable tests because they seem to freeze

  cd "${srcdir}"/vim-build

  #make test
}

package_vim() {
  pkgdesc='Vi Improved, a highly configurable, improved version of the vi text editor (with support for additional scripting languages)'
  depends=("vim-runtime=${pkgver}-${pkgrel}" 'gpm')

  cd "${srcdir}"/vim-build
  make -j1 VIMRCLOC=/etc DESTDIR="${pkgdir}" install

  # provided by (n)vi in core
  rm "${pkgdir}"/usr/bin/{ex,view}

  # delete some manpages
  find "${pkgdir}"/usr/share/man -type d -name 'man1' 2>/dev/null | \
    while read _mandir; do
    cd ${_mandir}
    rm -f ex.1 view.1 # provided by (n)vi
    rm -f evim.1    # this does not make sense if we have no GUI
  done

  # Runtime provided by runtime package
  rm -r "${pkgdir}"/usr/share/vim

  # license
  install -Dm644 runtime/doc/uganda.txt \
    "${pkgdir}"/usr/share/licenses/${pkgname}/license.txt
}

package_vim-runtime() {
  pkgdesc='Runtime for vim and gvim'
  depends=('perl' 'gawk')
  backup=('etc/vimrc')

  cd "${srcdir}"/vim-runtime-build

  (cd src && make -j1 VIMRCLOC=/etc DESTDIR="${pkgdir}" installruntime install-languages installtools)
  # man and bin files belong to 'vim'
  rm -r "${pkgdir}"/usr/share/man/ "${pkgdir}"/usr/bin/

  # Don't forget logtalk.dict
  install -Dm644 runtime/ftplugin/logtalk.dict \
    "${pkgdir}"/usr/share/vim/${_versiondir}/ftplugin/logtalk.dict

  # fix FS#17216
  sed -i 's|messages,/var|messages,/var/log/messages.log,/var|' \
    "${pkgdir}"/usr/share/vim/${_versiondir}/filetype.vim

  # patch filetype.vim for better handling of pacman related files
  sed -i "s/rpmsave/pacsave/;s/rpmnew/pacnew/;s/,\*\.ebuild/\0,PKGBUILD*,*.install/" \
    "${pkgdir}"/usr/share/vim/${_versiondir}/filetype.vim
  sed -i "/find the end/,+3{s/changelog_date_entry_search/changelog_date_end_entry_search/}" \
    "${pkgdir}"/usr/share/vim/${_versiondir}/ftplugin/changelog.vim

  # rc files
  install -Dm644 "${srcdir}"/vimrc "${pkgdir}"/etc/vimrc
  install -Dm644 "${srcdir}"/parabola.vim \
    "${pkgdir}"/usr/share/vim/vimfiles/parabola.vim

  # rgb.txt file
  install -Dm644 runtime/rgb.txt \
    "${pkgdir}"/usr/share/vim/${_versiondir}/rgb.txt

  # license
  install -dm755 "${pkgdir}"/usr/share/licenses/vim-runtime
  ln -s /usr/share/vim/${_versiondir}/doc/uganda.txt \
    "${pkgdir}"/usr/share/licenses/vim-runtime/license.txt
}

# vim:set sw=2 sts=2 et:
