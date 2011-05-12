# How many source files to build in parallel within each package
jobs=-j2

# Source and binary directories
src=/src
prefix=/target
work=$(src)/macosx
stow=$(prefix)/stow

include ../makefile.common

pyqt=PyQt-mac-gpl-4.8.4


# Useful macros for building packages
pkgconfig=PKG_CONFIG_PATH=$(prefix)/lib/pkgconfig
cflags=-m32 -I$(prefix)/include
ldflags=-m32 -L$(prefix)/lib -headerpad_max_install_names
flagsm32=CFLAGS="$(cflags)" CXXFLAGS="$(cflags)" LDFLAGS="$(ldflags)" LFLAGS="$(ldflags)"
path=PATH=$(prefix)/bin:$$PATH
configure=$(pkgconfig) $(flagsm32) $(path) ./configure --prefix=$(prefix)
configurepy=python configure.py \
	--bindir $(prefix)/bin \
	--destdir $(prefix)/lib/python2.6/site-packages \
	--sipdir $(prefix)/share/sip
cmake=$(pkgconfig) cmake .. \
	-DCMAKE_OSX_ARCHITECTURES=i386 \
	-DCMAKE_INSTALL_PREFIX=$(prefix) \
	-DQT_HEADERS_DIR=$(prefix)/include/ \
	-DQT_LIBRARY_DIR=$(prefix)/bin
make=$(path) make $(jobs)
extractpkg=tar -xvf $(1)
buildpkg=(cd $(1) && $(configure) && $(make))
buildpkgcmake=(cd $(1) && mkdir build ; cd build && $(cmake) && $(make))
installpkg=rm -rf $(stow)/$(1) && \
	cd $(1) && \
	make install DESTDIR=$(stow)/$(1) && \
	mv -fv $(stow)/$(1)$(prefix)/* $(stow)/$(1)/ && \
	cd $(stow) && \
	stow -R $(1) && \
	touch $(work)/$@
installpkgcmake=rm -rf $(stow)/$(1) && \
	cd $(1)/build && \
	make install DESTDIR=$(stow)/$(1) && \
	mv -fv $(stow)/$(1)$(prefix)/* $(stow)/$(1)/ && \
	cd $(stow) && \
	stow -R $(1) && \
	touch $(work)/$@

extractandbuild=$(call extractpkg,$(1)) && $(call buildpkg,$(2)) && $(call installpkg,$(2))
extractandbuildcmake=$(call extractpkg,$(1)) && $(call buildpkgcmake,$(2)) && $(call installpkgcmake,$(2))


# Top-level targets
clementine-build-deps: boost liblastfm taglib libimobiledevice libmtp \
                       libgpod pyqt gstreamer glew \
                       libarchive protobuf spotify
clementine-gst-plugins: gst-plugins-base gst-plugins-good gst-plugins-bad \
                        gst-plugins-ugly gst-ffmpeg
clementine-deps: clementine-build-deps clementine-gst-plugins

# The actual commands for compiling each package follow

.done-sqlite:
	$(call extractandbuild,$(sqlitetgz),$(sqlite))

.done-gettext:
	$(call extractandbuild,$(gettexttgz),$(gettext))

.done-glib: .done-gettext
	$(call extractandbuild,$(glibtgz),$(glib))

.done-libplist: .done-glib
	$(call extractpkg,$(plisttgz))
	-mkdir $(plist)/build
	cd $(plist)/build && $(cmake) -DENABLE_PYTHON=OFF
	cd $(plist)/build && $(make)
	$(call installpkgcmake,$(plist))

.done-libgpod: .done-sqlite .done-libplist
	$(call extractpkg,$(gpodtgz))
	cd $(gpod) && $(configure) --disable-gdk-pixbuf
	cd $(gpod) && $(make)
	$(call installpkg,$(gpod))

.done-gstreamer: .done-glib .done-orc
	$(call extractpkg,$(gsttgz))
	cd $(gst) && $(configure) --disable-examples --disable-tests
	cd $(gst) && $(make)
	$(call installpkg,$(gst))

.done-gst-plugins-base: .done-gstreamer .done-ogg .done-vorbis
	$(call extractpkg,$(gstbasetgz))
	cd $(gstbase) && $(configure) --disable-examples --disable-x --disable-xvideo --disable-xshm
	cd $(gstbase) && $(make)
	$(call installpkg,$(gstbase))

.done-orc:
	$(call extractandbuild,$(orctgz),$(orc))

.done-ogg:
	$(call extractandbuild,$(oggtgz),$(ogg))

.done-vorbis: .done-ogg
	$(call extractandbuild,$(vorbistgz),$(vorbis))

.done-gst-plugins-good: .done-gstreamer .done-gst-plugins-base .done-flac .done-speex .done-wavpack .done-libsoup
	$(call extractpkg,$(gstgoodtgz))
	cd $(gstgood) && SOUP_CFLAGS=-I/target/include/libsoup-2.4 $(configure) --disable-shout2 --disable-examples --disable-x --disable-deinterlace
	cd $(gstgood) && $(make)
	$(call installpkg,$(gstgood))

.done-flac:
	$(call extractpkg,$(flactgz))
	cd $(flac) && $(configure) --disable-xmms-plugin --disable-asm-optimizations
	cd $(flac) && $(make)
	$(call installpkg,$(flac))

.done-speex: .done-ogg
	$(call extractandbuild,$(speextgz),$(speex))

.done-gst-plugins-ugly: .done-gstreamer .done-gst-plugins-base .done-oil .done-mad .done-lame
	$(call extractpkg,$(gstuglytgz))
	cd $(gstugly) && $(configure) --disable-examples --disable-x
	cd $(gstugly) && $(make)
	$(call installpkg,$(gstugly))

.done-oil:
	$(call extractandbuild,$(oiltgz),$(oil))

.done-id3tag:
	$(call extractandbuild,$(id3tagtgz),$(id3tag))

.done-mad: .done-id3tag
	$(call extractandbuild,$(madtgz),$(mad))

.done-lame:
	$(call extractandbuild,$(lametgz),$(lame))

.done-gst-plugins-bad: .done-gstreamer .done-gst-plugins-base .done-faad .done-faac .done-libmms .done-oil .done-ofa .done-libquicktime .done-libmpcdec
	$(call extractpkg,$(gstbadtgz))
	cd $(gstbad) && $(configure) --disable-acm --disable-examples --disable-x
	cd $(gstbad) && $(make)
	$(call installpkg,$(gstbad))

.done-gst-ffmpeg: .done-gstreamer .done-gst-plugins-base
	$(call extractpkg,$(gstffmpegtgz))
	cd $(gstffmpeg) && $(configure) --with-ffmpeg-extra-configure="--enable-shared --disable-debug --disable-mmx"
	sed -e 's/HAVE_EBX_AVAILABLE 1/HAVE_EBX_AVAILABLE 0/' -i "" $(gstffmpeg)/gst-libs/ext/ffmpeg/config.h
	cd $(gstffmpeg) && $(make)
	$(call installpkg,$(gstffmpeg))

.done-faad:
	$(call extractandbuild,$(faadtgz),$(faad))

.done-faac:
	$(call extractandbuild,$(faactgz),$(faac))

.done-libmms: .done-glib
	$(call extractandbuild,$(mmstgz),$(mms))

.done-libquicktime:
	$(call extractpkg,$(quicktimetgz))
	cd $(quicktime) && $(configure) --without-doxygen --without-ffmpeg --without-libiconv-prefix
	cd $(quicktime) && $(make)
	$(call installpkg,$(quicktime))

.done-usbmuxd: .done-libplist .done-usb
	$(call extractandbuildcmake,$(usbmuxdtgz),$(usbmuxd))

.done-usb:
	$(call extractandbuild,$(usbtgz),$(usb))

.done-libimobiledevice: .done-usbmuxd .done-libplist .done-gnutls
	$(call extractpkg,$(imobtgz))
	cd $(imob) && $(configure) --without-swig
	cd $(imob) && $(make)
	$(call installpkg,$(imob))

.done-gcrypt: .done-libgpgerror
	$(call extractpkg,$(gcrypttgz))
	cd $(gcrypt) && $(configure) --with-gpg-error-prefix=$(prefix)
	cd $(gcrypt) && $(make)
	$(call installpkg,$(gcrypt))

.done-taglib:
	$(call extractpkg,$(taglibtgz))
	-mkdir $(taglib)/build
	cd $(taglib)/build && $(cmake) -DWITH_ASF=ON -DWITH_MP4=ON
	cd $(taglib)/build && $(make)
	$(call installpkgcmake,$(taglib))

.done-liblastfm:
	$(call extractandbuildcmake,$(lastfmtgz),$(lastfm))

.done-boost:
	tar -xf $(boosttgz) -C $(stow)
	cd $(stow) && stow -R $(boost)
	touch $@

.done-python:
	$(call extractandbuild,$(pythontgz),$(python))

.done-sip:
	$(call extractpkg,$(siptgz))
	cd $(sip) && $(configurepy) --arch i386 --incdir $(prefix)/include
	cd $(sip) && $(make)
	$(call installpkg,$(sip))

.done-pyqt: .done-sip
	$(call extractpkg,$(pyqttgz))
	cd $(pyqt) && PYTHONPATH=$(prefix)/lib/python2.6/site-packages $(configurepy) LFLAGS+=-m32 CFLAGS+=-m32 CXXFLAGS+=-m32 \
		--confirm-license \
		--use-arch i386 \
		--no-designer-plugin
	cd $(pyqt) && $(make)
	$(call installpkg,$(pyqt))

.done-ofa: .done-curl .done-fftw
	$(call extractpkg,$(ofatgz))
	cd $(ofa) && patch -p0 < ../patches/libofa-01-framework-accelerate.patch
	cd $(ofa) && autoreconf
	$(call buildpkg,$(ofa))
	$(call installpkg,$(ofa))

.done-curl:
	$(call extractandbuild,$(curltgz),$(curl))

.done-fftw:
	$(call extractandbuild,$(fftwtgz),$(fftw))

.done-libarchive:
	$(call extractpkg,$(archivetgz))
	-mkdir $(archive)/build
	cd $(archive)/build && $(cmake) \
		-DENABLE_OPENSSL=OFF \
		-DENABLE_TAR=OFF \
		-DENABLE_CPIO=OFF \
		-DENABLE_XATTR=OFF \
		-DENABLE_ACL=OFF \
		-DENABLE_TEST=OFF
	$(call installpkgcmake,$(archive))

.done-protobuf:
	$(call extractandbuild,$(protobuftgz),$(protobuf))

.done-spotify:
	unzip -o -d $(stow) $(spotifymaczip)
	rm -rf $(stow)/$(spotifymac)/doc
	rm -rf $(stow)/$(spotifymac)/ChangeLog
	rm -rf $(stow)/$(spotifymac)/LICENSE
	rm -rf $(stow)/$(spotifymac)/README
	rm -rf $(stow)/$(spotifymac)/share
	cd $(stow) && stow $(spotifymac)
	touch $@

.done-libmtp: .done-usblegacy
	$(call extractpkg,$(mtptgz))
	$(call buildpkg,$(mtp))
	rm -rf $(stow)/$(mtp)
	cd $(mtp) && make install DESTDIR=$(stow)/$(mtp)
	rmdir $(stow)/$(mtp)/lib/udev
	rmdir $(stow)/$(mtp)/lib
	mv -fv $(stow)/$(mtp)$(prefix)/* $(stow)/$(mtp)/
	cd $(stow) && stow -R $(mtp)
	touch $(work)/$@

.done-usblegacy:
	$(call extractpkg,$(usblegacytgz))
	cd $(usblegacy) && $(configure) --disable-build-docs
	cd $(usblegacy) && $(make)
	$(call installpkg,$(usblegacy))

.done-libmpcdec:
	$(call extractandbuild,$(mpcdectgz),$(mpcdec))

.done-wavpack:
	$(call extractandbuild,$(wavpacktgz),$(wavpack))

.done-gnutls: .done-gcrypt .done-libtasn1
	$(call extractpkg,$(gnutlstgz))
	cd $(gnutls) && $(configure) --with-libgcrypt
	cd $(gnutls) && $(make)
	sed -e 's/zlib//' -i "" $(gnutls)/lib/gnutls.pc
	$(call installpkg,$(gnutls))

.done-libtasn1:
	$(call extractandbuild,$(tasn1tgz),$(tasn1))

.done-libgpgerror:
	$(call extractandbuild,$(gpgerrortgz),$(gpgerror))
	
.done-libsoup: .done-glib .done-gnutls
	$(call extractpkg,$(souptgz))
	cd $(soup) && $(configure) --without-gnome
	cd $(soup) && $(make)
	$(call installpkg,$(soup))

.done-glew:
	$(call extractpkg,$(glewtgz))
	cd $(glew) && $(make) CFLAGS.EXTRA=-m32 LDFLAGS.EXTRA=-m32 GLEW_DEST=$(stow)/$(glew)
	cd $(glew) && make install GLEW_DEST=$(stow)/$(glew)
	cd $(stow) && stow -R $(glew)
	touch $(work)/$@