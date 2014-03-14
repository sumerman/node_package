BUILDDIR        = $(shell pwd)
BUILD_STAGE_DIR = $(BUILDDIR)/{{package_name}}

# Where we install things (based on vars.config)
# /usr/local based dirs
PMAN_DIR         = $(BUILD_STAGE_DIR)/usr/local/man
PBIN_DIR         = $(BUILD_STAGE_DIR)/{{platform_bin_dir}}
PETC_DIR         = $(BUILD_STAGE_DIR)/{{platform_etc_dir}}
PLIB_DIR         = $(BUILD_STAGE_DIR)/{{platform_base_dir}}
# /var based dirs
PDATA_DIR        = $(BUILD_STAGE_DIR)/{{platform_data_dir}}
PLOG_DIR         = $(BUILD_STAGE_DIR)/var/log/{{package_install_name}}

# For scanning later, remove the leading slash
# '/var/db/server' becomes 'var/db/server'
PDATA_ROOT_DIR   = $(shell echo "{{platform_data_dir}}" | cut -d'/' -f 2-4)

TARNAME = {{package_name}}-$(PKG_VERSION)-$(OSNAME)-$(ARCH).tar
PKGNAME = {{package_name}}-$(PKG_VERSION)-$(OSNAME)-$(ARCH).tbz

build: packing_list_files templates
	@echo "Building package $(PKGNAME)"
	mkdir -p packages
	cd $(BUILD_STAGE_DIR) && \
	tar -cf $(TARNAME) +CONTENTS +COMMENT +DESC +MTREE_DIRS +DEINSTALL +DISPLAY
	cd $(BUILD_STAGE_DIR) && \
	find man -type f | tar -rf $(TARNAME) -T - && \
	find {{bin_or_sbin}} -type f | tar -rf $(TARNAME) -T - && \
	find lib -type f | tar -rf $(TARNAME) -T - && \
	find etc -type f | tar -rf $(TARNAME) -T - && \
	find var -type f | tar -rf $(TARNAME) -T - && \
	find log -type f | tar -rf $(TARNAME) -T -

	cd $(BUILD_STAGE_DIR) && \
	mkdir ../../packages && \
	gzip $(TARNAME) && \
	mv $(TARNAME).gz ../../packages/$(PKGNAME)

	cd ../packages && \
		for tarfile in *.tbz; do \
		shasum -a 256 $${tarfile} > $${tarfile}.sha \
	; done

# Where most of the magic (horror?) happens
# Create a packing list according to pkg_create(1)
# Write initial settings to a local file then copy
#   to the destination folder where we use 'find'
#   to populate the files and directories
# Regarding @owner and @group callouts
#   They work on files, but not on directories.
#   Also, the last callout should be blank to revert
#   back to the default so the /var/db/pkg directory
#   doesn't get set to something bad (which is a
#   horrible bug in pkg_add).

manifest_file_section: $(BUILD_STAGE_DIR)


packing_list_files: $(BUILD_STAGE_DIR)
	@cd $(BUILD_STAGE_DIR) && \
		echo "files:" >> +MANIFEST

	@echo "Copying Man pages to staging directory"
	@cd $(BUILD_STAGE_DIR) && \
	if [ -d doc/man/man1 ]; then \
		mkdir -p $(PMAN_DIR) && \
		cp -R doc/man/man1 $(PMAN_DIR) && \
                {{#package_commands}}if [ -f doc/man/man1/{{name}}.1.gz ]; then \
                                         echo "  /usr/local/man/man1/{{name}}.1.gz" >> +MANIFEST \
                                     ; fi && \
		{{/package_commands}} echo -n; fi

	@echo "Packaging /usr/local files"
	@cd $(BUILD_STAGE_DIR) && \
	find usr -type f | while read file ; do \
	    mode=$$(stat -f%p "$$file" | cut -c 3-) && \
	    sum=$$(sha256 -q $$file) && \
	    echo "  /$$file: { sum: $$sum, uname: {{package_install_user}}, gname: {{package_install_group}}, perm: $$mode }" >> +MANIFEST; done

	@cd $(BUILD_STAGE_DIR) && \
		echo "directories:" >> +MANIFEST && \
		echo "- {{platform_base_dir}}" >> +MANIFEST && \
		echo "- {{platform_data_dir}}" >> +MANIFEST && \
		echo "- {{platform_etc_dir}}" >> +MANIFEST 

# templates: $(BUILD_STAGE_DIR)
# 	@echo "Copying metadata files to package"
# 	cp $(PKGERDIR)/+DESC $(PKGERDIR)/+COMMENT $(PKGERDIR)/+DEINSTALL \
#            $(PKGERDIR)/+MTREE_DIRS $(PKGERDIR)/+DISPLAY \
#            $(BUILD_STAGE_DIR)


# Copy the app rel directory to the staging directory to build our
# package structure and move the directories into the right place
# for the package, see the vars.config file for destination
# directories
$(BUILD_STAGE_DIR): buildrel
	@echo "Copying rel directory to staging directory"
	mkdir -p $@
	mkdir -p $(PBIN_DIR)
	cp -R rel/{{package_install_name}}/bin/* $(PBIN_DIR)
	mkdir -p $(PETC_DIR)
	cp -R rel/{{package_install_name}}/etc/* $(PETC_DIR)
	mkdir -p $(PLIB_DIR)
	cp -R rel/{{package_install_name}}/lib $(PLIB_DIR)
	cp -R rel/{{package_install_name}}/erts-* $(PLIB_DIR)
	cp -R rel/{{package_install_name}}/releases $(PLIB_DIR)
	mkdir -p $(PDATA_DIR)
	cp -R rel/{{package_install_name}}/data/* $(PDATA_DIR)
        cp +MANIFEST $(BUILD_STAGE_DIR)/+MANIFEST


# Build the release we need to package
#  * Ensure all binaries are executable
#  * copy the vars.config over for build config
buildrel:
	OVERLAY_VARS="overlay_vars=../fbsdng/vars.config" $(MAKE) deps rel
	chmod 0755 rel/{{package_install_name}}/bin/* rel/{{package_install_name}}/erts-*/bin/*

$(BUILDDIR):
	mkdir -p $@

$(PKGERDIR)/pkgclean:
	rm -rf $(BUILD_STAGE_DIR) $(BUILDDIR)
