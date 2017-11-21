FROM alpine:3.5

ENV BINUTILS_VERSION 2.28
ENV BINUTILS_DOWNLOAD_URL http://ftp.gnu.org/gnu/binutils/binutils-$BINUTILS_VERSION.tar.bz2
ENV BINUTILS_DOWNLOAD_SIG http://ftp.gnu.org/gnu/binutils/binutils-$BINUTILS_VERSION.tar.bz2.sig
ENV BINUTILS_KEY C3126D3B4AE55E93

ENV MINGW_VERSION 5.0.2
ENV MINGW_DOWNLOAD_URL https://sourceforge.net/projects/mingw-w64/files/mingw-w64/mingw-w64-release/mingw-w64-v$MINGW_VERSION.tar.bz2/download
ENV MINGW_DOWNLOAD_SHA1 bb5409f034abb7c021b3e1c14db433fd253cbb59
ENV MINGW_ROOT /usr/local/mingw

ENV GCC_VERSION 6.3.0
ENV GCC_DOWNLOAD_URL http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-$GCC_VERSION/gcc-$GCC_VERSION.tar.bz2
ENV GCC_DOWNLOAD_SHA512 234dd9b1bdc9a9c6e352216a7ef4ccadc6c07f156006a59759c5e0e6a69f0abcdc14630eff11e3826dd6ba5933a8faa43043f3d1d62df6bd5ab1e82862f9bf78

ENV PATH $MINGW_ROOT/bin:$PATH

RUN apk add --no-cache --virtual download-dependencies \
    curl \
    gnupg \
 && mkdir -p /mingw-build \
 && cd /mingw-build \
 # Download binutils
 && curl -fsSL "$BINUTILS_DOWNLOAD_URL" -o binutils.tar.bz2 \
 && curl -fsSL "$BINUTILS_DOWNLOAD_SIG" -o binutils.tar.bz2.sig \
 && gpg --batch --keyserver pgp.mit.edu `if test "x$HTTP_PROXY" != "x"; then echo "--keyserver-options http-proxy=$HTTP_PROXY"; fi` --recv-keys "$BINUTILS_KEY" \
 && gpg --batch --verify binutils.tar.bz2.sig binutils.tar.bz2 \
 && gpg --batch --yes --delete-keys "$BINUTILS_KEY" && rm -Rf /root/.gnupg \
 && rm -f binutils.tar.bz2.sig \
 # Download MinGW-w64
 && curl -fsSL "$MINGW_DOWNLOAD_URL" -o mingw.tar.bz2 \
 && echo "$MINGW_DOWNLOAD_SHA1  mingw.tar.bz2" | sha1sum -c - \
 # Download GCC
 && curl -fsSL "$GCC_DOWNLOAD_URL" -o gcc.tar.bz2 \
 && echo "$GCC_DOWNLOAD_SHA512  gcc.tar.bz2" | sha512sum -c - \
 && apk del --purge download-dependencies \
 # Install build dependencies
 && apk add --no-cache --virtual build-dependencies \
    build-base \
    flex \
    bison \
    gmp-dev \
    mpfr-dev \
    mpc1-dev \
    cloog-dev \
    file \
 && apk add --no-cache \
    gmp \
    mpfr3 \
    mpc1 \
    cloog \
 # Build binutils
 && tar -xjf binutils.tar.bz2 \
 && mkdir build \
 && ( cd build && CFLAGS="-Os -march=native" CXXFLAGS="-Os -march=native" ../binutils-$BINUTILS_VERSION/configure \
        --prefix=$MINGW_ROOT \
        --with-sysroot=$MINGW_ROOT \
        --target=x86_64-w64-mingw32 \
        --disable-maintainer-mode \
        --disable-multilib \
        --enable-lto \
        --disable-werror \
    && make && make install && cd .. ) \
 && rm -Rf build/ binutils-$BINUTILS_VERSION/ binutils.tar.bz2 \
 # Install MinGW-w64 headers
 && tar -xjf mingw.tar.bz2 && rm -f mingw.tar.bz2 \
 && mkdir build \
 && ( cd build && ../mingw-w64-v$MINGW_VERSION/mingw-w64-headers/configure --prefix=$MINGW_ROOT/x86_64-w64-mingw32 --enable-sdk=all --enable-secure-api --host=x86_64-w64-mingw32 && make install && cd .. ) \
 && rm -Rf build/ \
 && ln -s $MINGW_ROOT/x86_64-w64-mingw32 $MINGW_ROOT/mingw \
 # Build GCC stage1
 && tar -xjf gcc.tar.bz2 && rm -f gcc.tar.bz2 \
 && mkdir gcc-build \
 && ( cd gcc-build && CFLAGS="-Os -march=native" CXXFLAGS="-Os -march=native" ../gcc-$GCC_VERSION/configure \
        --target=x86_64-w64-mingw32 \
        --disable-multilib \
        --enable-64bit \
        --prefix=$MINGW_ROOT \
        --with-sysroot=$MINGW_ROOT \
        --enable-version-specific-runtime-libs \
        --enable-shared \
        --with-dwarf \
        --enable-fully-dynamic-string \
        --enable-languages=c,c++ \
        --enable-libssp \
        --with-host-libstdcxx="-lstdc++ -lsupc++" \
        --with-gmp=/usr \
        --with-mpfr=/usr \
        --with-mpc=/usr \
        --with-cloog=/usr \
        --enable-lto \
        --disable-maintainer-mode \
        --disable-werror \
    && make all-gcc && make install-gcc \
    && cd .. ) \
 # Build MinGW-w64 CRT
 && mkdir build \
 && ( cd build && CFLAGS="-Os" CXXFLAGS="-Os" ../mingw-w64-v$MINGW_VERSION/mingw-w64-crt/configure --prefix=$MINGW_ROOT/x86_64-w64-mingw32 --enable-lib64 --host=x86_64-w64-mingw32 && make && make install && cd .. ) \
 && rm -Rf build/ mingw-w64-v$MINGW_VERSION/ \
 # Build GCC stage2
 && ( cd gcc-build && make all-target-libgcc && make install-target-libgcc && cd .. ) \
 # Build GCC
 && ( cd gcc-build && make && make install && cd .. ) \
 && rm -Rf gcc-build/ gcc-$GCC_VERSION/ \
 && apk del --purge build-dependencies \
 && cd / \
 && rm -Rf /mingw-build/
