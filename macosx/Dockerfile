FROM ubuntu:16.04
RUN apt-get update
RUN apt-get install -y curl xz-utils build-essential gobjc++ git python stow pkg-config libglib2.0-dev bison flex protobuf-compiler cmake unzip vim intltool gettext yasm
RUN curl -SL https://github.com/tpoechtrager/osxcross/raw/master/tarballs/cctools-886-ld64-264.3.102_db1f8d9.tar.xz | tar -xJ
RUN cd cctools-886-ld64-264.3.102_db1f8d9/cctools && ./configure --prefix=/usr --target=x86_64-apple-darwin && make && make install
RUN mkdir -p /Developer/SDKs
RUN curl -SL https://www.dropbox.com/s/yfbesd249w10lpc/MacOSX10.10.sdk.tar.xz | tar -xJC /Developer/SDKs
RUN curl -SL http://llvm.org/releases/3.9.0/clang+llvm-3.9.0-x86_64-linux-gnu-ubuntu-16.04.tar.xz | tar -xJ --strip-components=1 -C /usr
RUN ln -s /usr/bin/clang /usr/bin/x86_64-apple-darwin-clang
RUN ln -s /usr/bin/clang++ /usr/bin/x86_64-apple-darwin-clang++
RUN ln -s /usr/bin/clang /usr/bin/x86_64-apple-darwin-gcc
RUN ln -s /usr/bin/clang++ /usr/bin/x86_64-apple-darwin-g++
RUN git clone --branch=master https://github.com/clementine-player/Dependencies /src
RUN mkdir -p /target/stow
RUN mkdir -p /target/lib
RUN mkdir -p /target/bin
RUN cd /src/macosx && make all-downloads
ADD glib.cache /src/macosx/
ADD mpcdec.cache /src/macosx/
ADD oil.cache /src/macosx/
RUN chmod -w /src/macosx/*.cache
ENV GLIB_GENMARSHAL /usr/bin/glib-genmarshal
ADD Makefile /src/macosx/Makefile
ADD patches/qt-03-objc.patch /src/macosx/patches
ADD Toolchain-Darwin.cmake /src/macosx/Toolchain-Darwin.cmake
ADD https://raw.githubusercontent.com/Homebrew/brew/bbed7246bc5c5b7acb8c1d427d10b43e090dfd39/Library/Homebrew/os/mac/pkgconfig/10.10/libxml-2.0.pc /target/lib/pkgconfig/libxml-2.0.pc
RUN cd /src/macosx && make clementine-deps
