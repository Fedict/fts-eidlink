VERSION = 1.6
IDENTIFIER = be.bosa.eidlink
PROGRAM = eidlink
DISTDIR = ./install/dist
DEPSDIR = ./install/deps
BUILDDIR = ./build
EXE = ./Build/Release/eIDLink
BINARIES = /tmp/eIDLink.dst
DMGFILE = $(PROGRAM)_$(VERSION).dmg
INSTALL_PRODUCT = $(DISTDIR)/1.$(PROGRAM)-install.pkg
INSTALL_RESOURCES = ./install/install_resources
UNINSTALL_PRODUCT = $(DISTDIR)/$(PROGRAM)-uninstall.pkg
UNINSTALL_RESOURCES = ./install/uninstall_resources
INSTALL_COMPONENT = $(DEPSDIR)/$(PROGRAM)Component.pkg
UNINSTALL_COMPONENT = $(DEPSDIR)/$(PROGRAM)UninstallComponent.pkg
COMPONENT_PFILE = ./install/eIDlinkComponent.plist
SCRIPTS = ./install/scripts
UNINSTALLSCRIPTS = ./install/uninstallscripts
INSTALL_DISTRIBUTION_FILE = ./install/distribution.plist
UNINSTALL_DISTRIBUTION_FILE = ./install/uninstalldistribution.plist
REQUIREMENTS =./install/requirements.plist


.PHONY : all
all: $(DISTDIR) $(DEPSDIR) $(INSTALL_PRODUCT) $(UNINSTALL_PRODUCT) $(DMGFILE)

$(DISTDIR):
	mkdir $(DISTDIR)
	cp $(INSTALL_RESOURCES)/README.html $(DISTDIR)/
	cp $(INSTALL_RESOURCES)/LICENSE.html $(DISTDIR)

$(DEPSDIR) :
	mkdir $(DEPSDIR)

$(DMGFILE): $(INSTALL_PRODUCT) $(UNINSTALL_PRODUCT)
	hdiutil create -fs JHFS+ -volname “eIDLink_Installer” -srcfolder $(DISTDIR) -ov $(DMGFILE)
	codesign -s "Zetes Sa" --timestamp -i $(IDENTIFIER) $(DMGFILE)
	cp $(DMGFILE) $(PROGRAM).dmg

$(INSTALL_PRODUCT) : $(BINARIES) $(REQUIREMENTS) $(DEPENDENCY) $(COMPONENT_PFILE) $(INSTALL_COMPONENT) $(INSTALL_DISTRIBUTION_FILE)
	productbuild --distribution $(INSTALL_DISTRIBUTION_FILE) \
	--sign "Developer ID Installer: Zetes Sa" \
	--timestamp \
	--resources $(INSTALL_RESOURCES) \
	--package-path $(DEPSDIR) \
	$(INSTALL_PRODUCT)

$(UNINSTALL_PRODUCT): $(COMPONENT_PFILE) $(UNINSTALL_COMPONENT) $(UNINSTALL_DISTRIBUTION_FILE)
	productbuild --distribution $(UNINSTALL_DISTRIBUTION_FILE) \
	--sign "Developer ID Installer: Zetes Sa" \
	--timestamp \
	--resources $(UNINSTALL_RESOURCES) \
	--package-path $(DEPSDIR) \
	$(UNINSTALL_PRODUCT)

$(BINARIES):
	xcodebuild install -project eIDLink.xcodeproj
	codesign -f --options=runtime -s "Zetes Sa" -v $(EXE)
	
$(COMPONENT_PFILE) :
	@echo "Error: Missing component pfile."
	@echo "Create a component pfile with make compfiles."
	@exit 1

$(INSTALL_COMPONENT) : $(BINARIES) $(COMPONENT_PFILE)
	pkgbuild --root $(BINARIES) \
	--scripts $(SCRIPTS) \
	--component-plist $(COMPONENT_PFILE) \
	$(INSTALL_COMPONENT) \
	--identifier $(IDENTIFIER) \
	--version $(VERSION)

#COMPONENT_PFILE is empty so we can use the same for the uninstallcomponent.plist
$(UNINSTALL_COMPONENT) : $(COMPONENT_PFILE)
	pkgbuild --nopayload \
	--scripts $(UNINSTALLSCRIPTS) \
	$(UNINSTALL_COMPONENT) \
	--identifier $(IDENTIFIER) \
	--version $(VERSION)

$(INSTALL_DISTRIBUTION_FILE) :
	@echo "Error: Missing distribution file."
	@echo "Create a distribution file with make distfiles."
	@exit 1

.PHONY : distfiles
distfiles : $(INSTALL_COMPONENT) $(UNINSTALL_COMPONENT)
	productbuild --synthesize \
	--product $(REQUIREMENTS) \
	--package $(INSTALL_COMPONENT) \
	$(DISTRIBUTION_FILE).new
	@echo "Edit the $(DISTRIBUTION_FILE).new template to create a suitable $(DISTRIBUTION_FILE) file."
	productbuild --synthesize \
	--product $(REQUIREMENTS) \
	--package $(UNINSTALL_COMPONENT) \
	$(UNINSTALL_DISTRIBUTION_FILE).new
	@echo "Edit the $(UNINSTALLDISTRIBUTION_FILE).new template to create a suitable $(UNINSTALL_DISTRIBUTION_FILE) file."
	
.PHONY : compfiles
compfiles : $(BINARIES)
	pkgbuild --analyze --root $(BINARIES) $(COMPONENT_PFILE).new
	@echo "Edit the $(COMPONENT_PFILE).new template to create a suitable $(COMPONENT_PFILE) file."

notarise: $(DMGFILE)
	xcrun altool --notarize-app --primary-bundle-id $(IDENTIFIER) --username "@keychain:AC_USER" --password "@keychain:AC_PASSWORD" --asc-provider ZetesSa --file $(PROGRAM).dmg

staple: 
	xcrun stapler staple $(PROGRAM).dmg

.PHONY : clean
clean:
	-rm -f $(DMGFILE) $(INSTALL_PRODUCT) $(UNINSTALL_PRODUCT) $(INSTALL_COMPONENT) $(UNINSTALL_COMPONENT)
	-rm -rf $(BINARIES)
	-rm -rf $(DISTDIR)
	-rm -rf $(BUILDDIR)

test: all
	python ../test/test.py
