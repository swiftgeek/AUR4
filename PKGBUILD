# Maintainer: Swift Geek <swifgeek ɐ google m č0m>
# Contributor: Nick Østergaard <oe.nick at gmail dot com>
# Contributor: olasd
#TODO: sed-out "need" for ExtUtils::Typemap or provide package if really needed

pkgname=slic3r-git
pkgver=0
pkgrel=11
pkgdesc="Slic3r is an STL-to-GCODE translator for RepRap 3D printers, aiming to be a modern and fast alternative to Skeinforge."
arch=('any')
url="http://slic3r.org/"
license=('GPL')
depends=('perl'
         'perl-moo' 'perl-boost-geometry-utils' 'perl-math-clipper' 'perl-math-convexhull' 'perl-math-geometry-voronoi' 'perl-math-planepath' 'perl-math-convexhull-monotonechain' 'perl-io-stringy' 'perl-encode-locale' 'perl-extutils-makemaker-aur>=6.82' 'perl-extutils-parsexs>=3.22' )
makedepends=('git')
optdepends=('perl-wx: GUI support'
            'perl-net-dbus: notifications support via any dbus-based notifier'
            'perl-xml-sax-expatxs: make AMF parsing faster'
            'perl-xml-sax: Additive Manufacturing File Format (AMF) support'
            'perl-wx-glcanvas: support for opengl preview' 
            'perl-opengl: support for opengl preview'
            'perl-class-xsaccessor: creating faster accessor methods')
#             'perl-growl-gntp: notifications support via growl'
provides=('slic3r' 'slic3r-xs' 'slic3r-xs-git')
conflicts=('slic3r' 'slic3r-xs' 'slic3r-xs-git')
#Consider uncommenting line below in case of false negative test results ;)
#BUILDENV+=('!check')
source=('slic3r.desktop' 'slic3r.pl')
md5sums=('1b561afff48c79f86889664375d179ed'
         '1371d698799ee97a43d22f6436a2e619')

_gitroot="git://github.com/alexrj/Slic3r"
_gitname="Slic3r"

prepare() {
  export _src_dir="$srcdir/$_gitname-build"
  # Setting these env variables overwrites any command-line-options we don't want...
  export PERL_MM_USE_DEFAULT=1 PERL_AUTOINSTALL=--skipdeps \
    PERL_MM_OPT="INSTALLDIRS=vendor DESTDIR='$pkgdir'" \
    PERL_MB_OPT="--installdirs vendor --destdir '$pkgdir'" \
    MODULEBUILDRC=/dev/null
  export SLIC3R_NO_AUTO="true"

  cd "$srcdir"
  msg "Connecting to GIT server...."

  if [ -d $_gitname ] ; then
    cd $_gitname
    git pull origin
    msg "The local files are updated."
    #TODO: check if it's the same origin
  else
    git clone --depth=1 $_gitroot $_gitname
    ##specific branch
    # git clone --depth=1 -b branch_name $_gitroot $_gitname
    ##specific commit
    # git clone --depth=1000 $_gitroot $_gitname
    # ( cd $srcdir/$_gitname; git checkout 754e6ac74b3cdbd8f8a409291f39777432a0983a )
  fi

  msg "GIT checkout done or server timeout"
  msg "Starting make..."

  rm -rf "$_src_dir"
  cp -R "$srcdir/$_gitname" "$srcdir/$_gitname-build"
  cd "$_src_dir"

  # Nasty fix for useless Growl dependency ... please post in comment real fix, if u know one ;) TODO: Change it to just line containing just like in netfabb
#  sed -i "s/        'Growl/\#&/" Build.PL
  sed -i '/Growl/d' Build.PL

  # Nasty fix for useless warning
  sed -i '/^warn \"Running Slic3r under Perl/,+1 s/^/\#/' ./lib/Slic3r.pm

  # Why true? cuz pacman is crazy... and it still doesn't work as intended
  true && pkgver="$(awk 'BEGIN{FS="\""}/VERSION/{gsub(/-dev/,"",$2); print $2 }' ./lib/Slic3r.pm).$(git rev-parse --short HEAD)"
  export _pkgver="$pkgver"
  msg2 "Fetched $_pkgver"
}

build() {
  cd "$_src_dir/xs"
  warning " ⚠  DO NOT respond to any question with 'yes'. Report a bug in comment instead.\n"
  warning "Running Slic3r under Perl >= 5.16 is not supported nor recommended\nIn case of related to this issues please use ARM repository to get older perl package\n"
  # Cuz cpan will install fixes to $HOME ... which is not the point of this package

  # slic3r-xs Build stage
  msg2 "Building Slic3r::XS (1/3)"
  /usr/bin/perl Build.PL
  ./Build

}

check () {
  cd "$_src_dir"

  msg2 "Testing Slic3r::XS - (2/3)"
  prove -Ixs/blib/arch -Ixs/blib/lib/ xs/t/

  msg2 "Testing Slic3r (3/3)"
  prove -Ixs/blib/arch -Ixs/blib/lib/ t/
}

package () {
  cd "$_src_dir"
  install -d $pkgdir/usr/share/perl5/vendor_perl/
  cp -R $srcdir/$_gitname-build/lib/* $pkgdir/usr/share/perl5/vendor_perl/

  install -d $pkgdir/usr/bin/vendor_perl/
  install -m 755 $srcdir/$_gitname-build/slic3r.pl $pkgdir/usr/bin/vendor_perl/

  #TODO : Do something about utils !
  #install -m 755 $srcdir/$_gitname-build/utils/*.pl $pkgdir/usr/bin/
  #install -m 755 $srcdir/$_gitname-build/utils/post-processing/*.pl $pkgdir/usr/bin/

  # ZSH autocompletion
  install -d "${pkgdir}/usr/share/zsh/site-functions"
  install -m 0644 "$srcdir/$_gitname-build/utils/zsh/functions/_slic3r" "$pkgdir/usr/share/zsh/site-functions/_slic3r.zsh"

  # Icons " current Build.PL is not really geared for installation "
  install -d $pkgdir/usr/bin/vendor_perl/var
  install -m 644 $srcdir/$_gitname-build/var/*  $pkgdir/usr/bin/vendor_perl/var/

  # Desktop icon
  install -d $pkgdir/usr/share/applications
  install -m 644 $srcdir/slic3r.desktop $pkgdir/usr/share/applications/

  # Welcome ultimate ugly - u² hack TODO: This may be not needed anymore!
  install -m 755 $srcdir/slic3r.pl $pkgdir/usr/bin/slic3r.pl

  ### SLIC3R-XS MERGE
  cd "$_src_dir/xs"
  ./Build install

  # Why double? 1st one was just for messages, this one is for real
  true && pkgver=$_pkgver
}

