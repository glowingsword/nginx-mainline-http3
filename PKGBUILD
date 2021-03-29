# Maintainer:  Andrei Gutu <glowingsword@gmail.com>

_pkgbase=nginx
pkgbase=nginx-mainline-http3
pkgname=(nginx-mainline-http3 nginx-mainline-http3-src)
pkgver=1.19.8
pkgrel=1
pkgdesc='Lightweight HTTP server and IMAP/POP3 proxy server, mainline release'
arch=('x86_64')
url='https://nginx.org'
license=('custom')
depends=('pcre' 'zlib' 'geoip' 'mailcap' 'libxcrypt')
# The ninja was added after expirements, with ninja build of the BoringSSL work faster, when with classic make.
makedepends=('cmake' 'git' 'go' 'ninja')
backup=('etc/nginx/fastcgi.conf'
        'etc/nginx/fastcgi_params'
        'etc/nginx/koi-win'
        'etc/nginx/koi-utf'
        'etc/nginx/nginx.conf'
        'etc/nginx/scgi_params'
        'etc/nginx/uwsgi_params'
        'etc/nginx/win-utf'
        'etc/logrotate.d/nginx')
install=nginx.install
source=(https://hg.nginx.org/nginx-quic/archive/tip.tar.gz
        "git+https://boringssl.googlesource.com/boringssl"
        service
        logrotate)
noextract=('tip.tar.gz')
md5sums=('5d3a8f6e63c70bd3861668e5f2ad2269'
         'SKIP'
         'ef491e760e7c1ffec9ca25441a150c83'
         '6a01fb17af86f03707c8ae60f98a2dc2')
sha512sums=('e15b35ff698b12a8781b6d338ce5d87800aa0ad4594b8fa78652ec0b79935c95b5851a43537054a8a12b9fa3d3cc0fae6948196d53a87911f3a8902a114593c8'
            'SKIP'
            '4f90db6b8b5c13762b96ddff9ca4e846762d46b90be27c7c9d54cec6f7f12fc95585f8455919296edb0255405dd80af8ee86780b805631b72eb74ee59f359715'
            '9232342c0914575ce438c5a8ee7e1c25b0befb457a2934e9cb77d1fe9a103634ea403b57bc0ef0cd6cf72248aee5e5584282cea611bc79198aeac9a65d8df5d7')

_common_flags=(
  --with-compat
  --with-debug
  --with-file-aio
  --with-http_addition_module
  --with-http_auth_request_module
  --with-http_dav_module
  --with-http_degradation_module
  --with-http_flv_module
  --with-http_geoip_module
  --with-http_gunzip_module
  --with-http_gzip_static_module
  --with-http_mp4_module
  --with-http_realip_module
  --with-http_secure_link_module
  --with-http_slice_module
  --with-http_ssl_module
  --with-http_stub_status_module
  --with-http_sub_module
  --with-http_v2_module
  --with-mail
  --with-mail_ssl_module
  --with-pcre-jit
  --with-stream
  --with-stream_geoip_module
  --with-stream_realip_module
  --with-stream_ssl_module
  --with-stream_ssl_preread_module
  --with-threads
  --with-http_v3_module
  --with-http_quic_module
  --with-stream_quic_module
)

_mainline_flags=(
  --with-stream_ssl_preread_module
  --with-stream_geoip_module
  --with-stream_realip_module
)



prepare() {
  tar vzxf ./tip.tar.gz -C ./
  mv nginx-quic-[0-9a-z][0-9a-z][0-9a-z][0-9a-z][0-9a-z][0-9a-z][0-9a-z][0-9a-z][0-9a-z][0-9a-z][0-9a-z][0-9a-z] "$_pkgbase-$pkgver"
  cp -r $_pkgbase-$pkgver{,-src}
}

build() {
  cd ${srcdir}/boringssl && git checkout origin/master && git submodule init && git submodule update
  mkdir ${srcdir}/boringssl/build
  
  ##                                                                                                                                                 ##
  # Warning, when you build the boringssl you can see the error.                                                                                      #
  # With D_FORTIFY_SOURCE compilation return "FAILED: crypto/test/CMakeFiles/test_support_lib.dir/abi_test.cc.o"  error                               #
  # See the https://bugs.chromium.org/p/boringssl/issues/detail?id=403#c1https://bugs.chromium.org/p/boringssl/issues/detail?id=403#c1                #
  # You can use CPPFLAGS='' cmake -GNinja -S ${srcdir}/boringssl/ -B  ${srcdir}/boringssl/build/ -DBORINGSSL_DIR=${srcdir}/boringssl/                 #
  # or sed -i 's/write(STDERR_FILENO, buf, strlen(buf));/(void)write(STDERR_FILENO, buf, strlen(buf));/g' ${srcdir}/boringssl/crypto/test/abi_test.cc #
  #  CC=/usr/bin/clang CXX=/usr/bin/clang++ cmake -GNinja -S ${srcdir}/boringssl/ -B  ${srcdir}/boringssl/build/ -DBORINGSSL_DIR=${srcdir}/boringssl/ #
  # or varinat from the line 101                                                                                                                      #
  ##                                                                                                                                                 ##
  
  cmake -E env CXXFLAGS="-Wno-unused-result $CXXFLAGS" cmake -GNinja -S ${srcdir}/boringssl/ -B  ${srcdir}/boringssl/build/ -DBORINGSSL_DIR=${srcdir}/boringssl/
  
  ninja -C ${srcdir}/boringssl/build 
  cd ${srcdir}/boringssl && mkdir -p .openssl/lib && cd .openssl && ln -s ../include . && cd ../
  cd ${srcdir}/boringssl && cp ${srcdir}/boringssl/build/crypto/libcrypto.a ${srcdir}/boringssl/build/ssl/libssl.a .openssl/lib && cd ..
  
  
  cd $_pkgbase-$pkgver
  ./auto/configure \
    --prefix=/etc/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --sbin-path=/usr/bin/nginx \
    --pid-path=/run/nginx.pid \
    --lock-path=/run/lock/nginx.lock \
    --user=http \
    --group=http \
    --http-log-path=/var/log/nginx/access.log \
    --error-log-path=stderr \
    --http-client-body-temp-path=/var/lib/nginx/client-body \
    --http-proxy-temp-path=/var/lib/nginx/proxy \
    --http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
    --http-scgi-temp-path=/var/lib/nginx/scgi \
    --http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
    --with-cc-opt="$CFLAGS $CPPFLAGS" \
    --with-ld-opt="$LDFLAGS" \
    --with-cc-opt="-I${srcdir}/boringssl/include"   \
    --with-ld-opt="-L${srcdir}/boringssl/build/ssl -L${srcdir}/boringssl/build/crypto" \
    ${_common_flags[@]} \
    ${_mainline_flags[@]}
  touch ${srcdir}/boringssl/.openssl/include/openssl/ssl.h
  make
}

package_nginx-mainline-http3() {
  provides=($_pkgbase)
  conflicts=($_pkgbase)

  cd $_pkgbase-$pkgver
  make DESTDIR="$pkgdir" install

  sed -e 's|\<user\s\+\w\+;|user html;|g' \
    -e '44s|html|/usr/share/nginx/html|' \
    -e '54s|html|/usr/share/nginx/html|' \
    -i "$pkgdir"/etc/nginx/nginx.conf

  rm "$pkgdir"/etc/nginx/*.default
  rm "$pkgdir"/etc/nginx/mime.types  # in mailcap

  install -d "$pkgdir"/var/lib/nginx
  install -dm700 "$pkgdir"/var/lib/nginx/proxy

  chmod 755 "$pkgdir"/var/log/nginx
  chown root:root "$pkgdir"/var/log/nginx

  install -d "$pkgdir"/usr/share/nginx
  mv "$pkgdir"/etc/nginx/html/ "$pkgdir"/usr/share/nginx

  install -Dm644 ../logrotate "$pkgdir"/etc/logrotate.d/nginx
  install -Dm644 ../service "$pkgdir"/usr/lib/systemd/system/nginx.service
  install -Dm644 ${srcdir}/$_pkgbase-$pkgver/docs/text/LICENSE "$pkgdir"/usr/share/licenses/$_pkgbase/LICENSE

  rmdir "$pkgdir"/run

  install -d "$pkgdir"/usr/share/man/man8/
  gzip -9c ${srcdir}/$_pkgbase-$pkgver/docs/man/nginx.8 > "$pkgdir"/usr/share/man/man8/nginx.8.gz

  for i in ftdetect indent syntax; do
    install -Dm644 ${srcdir}/$_pkgbase-$pkgver/contrib/vim/$i/nginx.vim \
      "$pkgdir/usr/share/vim/vimfiles/$i/nginx.vim"
  done
}

package_nginx-mainline-http3-src() {
  pkgdesc="Source code of nginx-mainline $pkgver, useful for building modules"
  conflicts=($_pkgbase-src)
  depends=()
  install -d "$pkgdir/usr/src"
  cp -r $_pkgbase-$pkgver-src "$pkgdir/usr/src/nginx"
}
