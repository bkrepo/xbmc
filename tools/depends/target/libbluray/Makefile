include ../../Makefile.include
DEPS= ../../Makefile.include Makefile

# lib name, version
LIBNAME=libbluray
VERSION=0.8.1
SOURCE=$(LIBNAME)-$(VERSION)
ARCHIVE=$(SOURCE).tar.bz2

# configuration settings
CONFIGURE=./configure --prefix=$(PREFIX) --exec-prefix=$(PREFIX) \
  --disable-examples --disable-doxygen-doc \
  --disable-bdjava --without-libxml2 --without-freetype

LIBDYLIB=$(PLATFORM)/src/.libs/libbluray.la

all: .installed-$(PLATFORM)

$(TARBALLS_LOCATION)/$(ARCHIVE):
	cd $(TARBALLS_LOCATION); $(RETRIEVE_TOOL) $(RETRIEVE_TOOL_FLAGS) $(BASE_URL)/$(ARCHIVE)

$(PLATFORM): $(TARBALLS_LOCATION)/$(ARCHIVE) $(DEPS)
	rm -rf $(PLATFORM); mkdir -p $(PLATFORM)
	cd $(PLATFORM); $(ARCHIVE_TOOL) $(ARCHIVE_TOOL_FLAGS) $(TARBALLS_LOCATION)/$(ARCHIVE)
	# libbluray has borked Makefile.am with respect to CFLAGS, remove the offending line
	sed -i -e "s|CFLAGS=|#CFLAGS=|" $(PLATFORM)/Makefile.am
ifeq ($(OS),android)
	cd $(PLATFORM); patch -p1 < ../android.patch
endif
	cd $(PLATFORM); ./bootstrap
	cd $(PLATFORM); $(CONFIGURE)

$(LIBDYLIB): $(PLATFORM)
	$(MAKE) -C $(PLATFORM)

.installed-$(PLATFORM): $(LIBDYLIB)
	$(MAKE) -C $(PLATFORM) install
ifeq ($(OS),android)
	rm -f $(PREFIX)/lib/libbluray.la $(PREFIX)/lib/libbluray.so $(PREFIX)/lib/libbluray.so.1
	mv -f $(PREFIX)/lib/libbluray.so.1.8.1 $(PREFIX)/lib/libbluray.so
	$(RPL) -e "libbluray.so.1" "libbluray.so\x00\x00" $(PREFIX)/lib/libbluray.so
	-$(READELF) --dynamic $(PREFIX)/lib/libbluray.so | grep ibrary
endif
	touch $@

clean:
	$(MAKE) -C $(PLATFORM) clean
	rm -f .installed-$(PLATFORM)

distclean::
	rm -rf $(PLATFORM) .installed-$(PLATFORM)
