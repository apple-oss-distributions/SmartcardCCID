Project               = ccid
UserType              = Administrator
ToolType              = Commands
GnuAfterInstall       = post-install
CCIDDriversPath		  = /usr/libexec/SmartCardServices/drivers
CCIDdylib			  = /ifd-ccid.bundle/Contents/MacOS/libccid.dylib
Extra_Configure_Flags = --enable-usbdropdir=$(CCIDDriversPath) \
	PCSC_CFLAGS="-I$(SRCROOT)/ccid/MacOSX -DRESPONSECODE_DEFINED_IN_WINTYPES_H -DUSE_COMPOSITE_AS_MULTISLOT" \
	PCSC_LIBS="-framework PCSC" \
	LIBUSB_CFLAGS="-I$(DSTROOT)/usr/include/libusb-1.0" \
	LIBUSB_LIBS="-L$(DSTROOT)/usr/lib -lusb-1.0 -framework IOKit -framework CoreFoundation -framework Foundation"

Extra_Environment     = V=1

include $(MAKEFILEPATH)/CoreOS/ReleaseControl/GNUSource.make

# install-strip will install stripped versions of binaries
Install_Target = install-strip

# Strip debug symbols from any dylibs that slipped through install-strip (libccid.dylib.1.3.8)
post-install:
	find $(DSTROOT)/ -name '*.dylib*' -exec strip -S \{\} \;
	find $(DSTROOT)/ -name 'libusb*' -exec rm \{\} \;
	find $(DSTROOT)/ -name 'usb*.h' -exec rm \{\} \;
	rm -r $(DSTROOT)/usr/include
	rm -r $(DSTROOT)/usr/lib
	install_name_tool -id $(CCIDDriversPath)$(CCIDdylib) $(DSTROOT)$(CCIDDriversPath)$(CCIDdylib)
	codesign -s - $(DSTROOT)$(CCIDDriversPath)$(CCIDdylib)

# Automatic Extract & Patch
AEP_Project    = ccid
AEP_Version    = 1.5.1
AEP_ProjVers   = $(AEP_Project)-$(AEP_Version)
AEP_Filename   = $(AEP_ProjVers).tar.bz2
AEP_ExtractDir = $(AEP_ProjVers)
AEP_Patches    = destDirFix.patch ForceWithoutPcsc.patch ccid-info-plist.patch osxConfigure.patch headerpadLDFlags.patch

# Extract the source.
install_source::
	$(TAR) -C $(SRCROOT) -jxf $(SRCROOT)/$(AEP_Filename)
	$(RMDIR) $(SRCROOT)/$(Project)
	$(MV) $(SRCROOT)/$(AEP_ExtractDir) $(SRCROOT)/$(Project)
	for patchfile in $(AEP_Patches); do \
		(cd $(SRCROOT)/$(Project) && patch -p0 < $(SRCROOT)/files/$$patchfile) || exit 1; \
	done
	(cd $(SRCROOT)/$(Project) && MacOSX/configure --no-configure)

