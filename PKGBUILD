# $Id: PKGBUILD 76613 2012-09-25 00:57:41Z svenstaro $
# Maintainer: Mark Coolen <mark dot coolen at gmail dot com>
# Contributor: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: Xavion <Xavion (dot) 0 (at) Gmail (dot) com>
# Contributor: Calogero Lo Leggio <kalos@autistici.org>
# Contributor: Matias Hernandez <msdark@archlinux.cl>

pkgname=bacula-mysql
pkgver=5.2.12
pkgrel=2
pkgdesc="An advanced backup tool with network and tape changer support (MySQL backend)"
arch=("i686" "x86_64")
url="http://www.bacula.org"
license=("GPL")
depends=('openssl' 'lzo2' 'zlib' 'python2' 'libmysqlclient')
makedepends=('qt' 'wxgtk' 'gtk2')
optdepends=('qt: for bat'
            'wxgtk: for bwx console'
            'gtk2: for tray monitor'
            'python2: python support')
options=(!buildflags !libtool)
replaces=('bacula')
conflicts=('bacula' 'bacula-postgresql')
backup=('etc/bacula/bconsole.conf'
        'etc/bacula/bacula-dir.conf'
        'etc/bacula/bacula-fd.conf'
        'etc/bacula/bacula-sd.conf')
install='bacula-mysql.install'
source=(http://downloads.sourceforge.net/project/bacula/bacula/${pkgver}/bacula-${pkgver}.tar.gz
        bacula-sd.rc.d
        bacula-fd.rc.d
        bacula-dir.rc.d
        systemd.patch)
md5sums=('b04c22b128b73359e4bbc9de06652c38'
         '6311f10c58261c4ee6e26ae2de5580a3'
         '7c240ed89c6e42b379386d27e163d3bc'
         'bb31d783b3362961eac4746219e3e3ff'
         '375f47942b8d7bced269183a3bf2ffca')

build() {
    cd ${srcdir}/bacula-${pkgver}

    sed -i "s/python-config/python2-config/g" configure

    # Build
    ./configure --prefix=/usr \
        --enable-build-dird --enable-build-stored --enable-smartalloc \
        --enable-bat --enable-tray-monitor --enable-bwx-console \
        --enable-batch-insert --enable-ipv6 \
        --with-mysql --with-openssl --with-python \
        --with-systemd=/usr/lib/systemd/system \
        --with-fd-user=root --with-fd-group=root \
        --with-dir-user=bacula --with-dir-group=bacula \
        --with-sd-user=bacula --with-sd-group=bacula \
        --sysconfdir=/etc/bacula --with-scriptdir=/etc/bacula/scripts \
        --with-working-dir=/var/cache/bacula/working \
        --with-subsys-dir=/var/cache/bacula/working \
        --with-archivedir=/var/cache/bacula/archive \
        --with-pid-dir=/var/run/bacula/

    make
}

package() {
    cd ${srcdir}/bacula-${pkgver}

    make DESTDIR=${pkgdir} install

    # Hack to setup systemd units
    mkdir -vp ${pkgdir}/etc/tmpfiles.d
    mkdir -vp ${pkgdir}/usr/lib/systemd/system
    cd platforms/systemd
    make DESTDIR=${pkgdir} install
    cd ../..
    mkdir -vp ${pkgdir}/usr/lib/tmpfiles.d
    mv ${pkgdir}/etc/tmpfiles.d/* ${pkgdir}/usr/lib/tmpfiles.d/
    rmdir ${pkgdir}/etc/tmpfiles.d
    patch -d ${pkgdir}/usr/lib/systemd/system < ${srcdir}/systemd.patch

    # Permissions
    chmod a+x ${pkgdir}/etc/bacula/scripts/{update_bacula_tables,delete_catalog_backup,update_mysql_tables,make_catalog_backup,bconsole}

    # Daemons
    mkdir -p ${pkgdir}/etc/rc.d/
    install -Dm755 ${srcdir}/bacula-dir.rc.d ${pkgdir}/etc/rc.d/bacula-dir
    install -Dm755 ${srcdir}/bacula-fd.rc.d ${pkgdir}/etc/rc.d/bacula-fd
    install -Dm755 ${srcdir}/bacula-sd.rc.d ${pkgdir}/etc/rc.d/bacula-sd

    # Logs
    install -D -m644 ${srcdir}/bacula-${pkgver}/scripts/logrotate ${pkgdir}/etc/logrotate.d/bacula
    sed -i "s|/var/cache/bacula/working/log|/var/log/bacula.log|g" ${pkgdir}/etc/{bacula/bacula-dir.conf,logrotate.d/bacula}

    # Fix permissions
    chmod -R 755 ${pkgdir}/usr/sbin
}
