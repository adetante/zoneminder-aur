# Maintainer: Mesmer <mesmer@fisica.if.uff.br>
# Contributor: Troy Will <troydwill at gmail dot com>
# Contributor: /dev/rs0                  </dev/rs0@secretco.de.com>
# Contributor: Jacek Burghardt           <jacek@hebe.us>
# Contributor: Vojtech Aschenbrenner     <v@asch.cz>
# Contributor: Jason Gardner             <buhrietoe@gmail.com>
# Contributor: Ross melin                <rdmelin@gmail.com>
# Contributor (Parabola): Márcio Silva   <coadde@lavabit.com>
# Contributor (Parabola): André Silva    <emulatorman@lavabit.com>
# Contributor: Charles Spence IV         <cspence@unomaha.edu>
# Contributor: Joe Julian                <me@joejulian.name>     
# Contributor: Antoine Detante           <antoine.detante@gmail.com>
# Orginally based on a Debian Squeeze package
pkgname=zoneminder
pkgver=1.30.4
pkgrel=1
pkgdesc='Capture, analyse, record and monitor video security cameras'
arch=( x86_64 )
backup=( etc/zm.conf )
url="https://github.com/ZoneMinder/zoneminder/releases"
license=( GPL )
depends=(
  libjpeg-turbo
  ffmpeg
  mariadb
  nginx fcgiwrap php-fpm php-gd php-apcu php-apcu-bc
  perl-dbi perl-dbd-mysql perl-date-manip perl-sys-mmap perl-libwww perl-lwp-protocol-https perl-sys-meminfo perl-sys-cpu
  polkit
)
makedepends=(
  cmake
  git
)
optdepends=(
)
#install=$pkgname.install

source=(
  https://github.com/ZoneMinder/zoneminder/archive/$pkgver.tar.gz
  backport-97380f0.patch
  issue-1919.patch
  zoneminder-tmpfile.conf
  zoneminder.service
  zoneminder-php.ini
  zoneminder-nginx.conf
)
sha256sums=(
  '879f57fdb1e013b3f17b1b0e87c5935683dad14922951d5f29d1370c1e490f2e'
  '162763ed8d834ea69d076a428217e7896c9e242815c666d60fce1f6835209e7d'
  '641ff91727d1c3f60e225ad5c9aa52ea1f3508290ddf4308265c4e7d99ab9d77'
  'e2de24ba7fe940daa0ab525128ab40ec1d04163279fe21f848b6edda3d852ef9'
  'a6043ed2a166c8e2783589f4052961400ee196019f9b79df73a758bd0794a0ec'
  '5e9e149f55f8acc6289e5129221f2ab60a7c6fbff45d5ef8832f66325552ee1b'
  'a52472b0a5bda6ae056b312b29fa42cbed940035861d39567cb28b14c980f201'
)


prepare () {
    cd $srcdir/zoneminder-$pkgver/web/api/app/Plugin/
    if [ ! -d "Crud/.git" ]; then
    git clone -b 3.0 https://github.com/FriendsOfCake/crud.git Crud
    fi
    patch $srcdir/zoneminder-$pkgver/src/zm_comms.h < $srcdir/issue-1919.patch
    patch $srcdir/zoneminder-$pkgver/src/zm_image.cpp < $srcdir/backport-97380f0.patch
}

build() {
   cd $srcdir/zoneminder-$pkgver
   
  cmake -DCMAKE_INSTALL_PREFIX=/usr \
        -DZM_PERL_SUBPREFIX=/lib/perl5 \
        -DZM_WEBDIR=/srv/zoneminder/www \
        -DZM_CGIDIR=/srv/zoneminder/cgi-bin \
        -DZM_WEB_USER=http \
        -DZM_WEB_GROUP=http \
        -DZM_CONTENTDIR=/var/cache/zoneminder \
        -DZM_LOGDIR=/var/log/zoneminder \
        -DZM_RUNDIR=/run/zoneminder \
        -DZM_TMPDIR=/var/lib/zoneminder/temp \
        -DZM_SOCKDIR=/var/lib/zoneminder/sock .
     
    make
} 
     
package() {

    cd $srcdir/zoneminder-$pkgver

    DESTDIR=$pkgdir make install

    # Change Polkit directory permissions to Arch Linux policy
    chgrp -v polkitd $pkgdir/usr/share/polkit-1/rules.d/
    chmod -v 750 $pkgdir/usr/share/polkit-1/rules.d/

    # BEGIN CREATE_ZONEMINDER_DIRECTORIES
    mkdir -pv           $pkgdir/var/{cache/zoneminder,log/zoneminder}
    chown -Rv http.http $pkgdir/var/{cache/zoneminder,log/zoneminder}
    
    # corresponds to -DZM_SOCKDIR=/var/lib/zoneminder/sock
    mkdir -pv          $pkgdir/var/lib/zoneminder/sock
    chown -v http.http $pkgdir/var/lib/zoneminder/sock
    
    # corresponds to -DZM_TMPDIR=/var/lib/zoneminder/temp
    mkdir -pv          $pkgdir/var/lib/zoneminder/temp
    chown -v http.http $pkgdir/var/lib/zoneminder/temp
    
    chown -v  http.http $pkgdir/etc/zm.conf 
    chmod 600           $pkgdir/etc/zm.conf
    # END CREATE_ZONEMINDER_DIRECTORIES

    # Make content directories in /var/cache/zoneminder and to link them in /srv/http/zoneminder
    for i in events images temp; do
        mkdir              $pkgdir/var/cache/zoneminder/$i
        chown -v http.http $pkgdir/var/cache/zoneminder/$i
        ln -s                     /var/cache/zoneminder/$i $pkgdir/srv/zoneminder/www/$i
        chown -v --no-dereference http.http               $pkgdir/srv/zoneminder/www/$i
    done

    # Create a link to the Zoneminder cgi binaries
    ln -sv /srv/zoneminder/cgi-bin $pkgdir/srv/zoneminder/www

    chown -h http.http $pkgdir/srv/zoneminder/{cgi-bin,www,www/cgi-bin}

    # Install configuration files
    #mkdir -p                                        $pkgdir/etc/httpd/conf/extra
    #install -D -m 644 $srcdir/httpd-$_pkgname.conf  $pkgdir/etc/httpd/conf/extra

    mkdir -p                                         $pkgdir/etc/php/conf.d
    install -D -m 644 $srcdir/zoneminder-php.ini     $pkgdir/etc/php/conf.d/zoneminder.conf

    mkdir -p                                         $pkgdir/etc/nginx
    install -D -m 644 $srcdir/zoneminder-nginx.conf  $pkgdir/etc/nginx/zoneminder.conf
    
    mkdir -p                                         $pkgdir/usr/lib/systemd/system
    install -D -m 644 $srcdir/zoneminder.service     $pkgdir/usr/lib/systemd/system
    
    install -D -m 644 $srcdir/zoneminder-$pkgver/COPYING $pkgdir/usr/share/license/$pkgname
    
    install -Dm644 $srcdir/zoneminder-tmpfile.conf $pkgdir/usr/lib/tmpfiles.d/zoneminder.conf

}
