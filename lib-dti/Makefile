#
# Copyright (C) 2006-2010 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=lib-dti
PKG_RELEASE:=1
PKG_BASE_NAME:=lib_dti
PKG_VERSION:=1.3.0
PKG_SOURCE_PROTO:=git
CONFIG_UGW_PKG_SOURCE_URL:=ssh://git@github.com
PKG_SOURCE_VERSION:=$(PKG_VERSION)
PKG_SOURCE:=$(PKG_BASE_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_PROTO).tar.gz
PKG_PROJECT:=maxlinear
PKG_SOURCE_NAME:=lib_dti
PKG_SOURCE_URL:=$(CONFIG_UGW_PKG_SOURCE_URL)/$(PKG_PROJECT)/$(PKG_SOURCE_NAME).git
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_SOURCE_VERSION)
PKG_MIRROR_HASH:=a2037d55c997b957304bde93573ea16f63ede92bae00f16408b2153eb76163f8

PKG_BIN_DIR=/opt/lantiq/bin
AGENT_LIB_DIR=/opt/lantiq/lib
PKG_BUILD_DEPENDS:=ifxos

PKG_FIXUP:=libtool
PKG_INSTALL:=1
PKG_LICENSE:=GPL-2.0-only BSD-2-Clause
PKG_LICENSE_FILES:=LICENSE

-include $(INCLUDE_DIR)/package-version-override.mk
include $(INCLUDE_DIR)/package.mk

ifeq ($(DUMP),)
	STAMP_CONFIGURED:=$(strip $(STAMP_CONFIGURED))_$(shell $(SH_FUNC) grep '^CONFIG_PACKAGE_lib-dti' $(TOPDIR)/.config | md5s)
endif

define Package/$(PKG_NAME)/Default
  TITLE:=DTI
  URL:=http://www.maxlinear.com/
endef

define Package/$(PKG_NAME)/Default/description
 Debug and Trace Interface (DTI)
 This package provides a generic library used for the DTI access.
endef

define Package/$(PKG_NAME)
$(call Package/$(PKG_NAME)/Default)
  SECTION:=libs
  CATEGORY:=MaxLinear
  DEPENDS:=+libpthread
  TITLE+= (library)
  SUBMENU:=DTI
endef

define Package/$(PKG_NAME)/description
$(call Package/$(PKG_NAME)/Default/description)
 This package contains the DTI shared libraries, needed by other programs.
 - libdti_agent
endef

define Package/$(PKG_NAME)-device-generic
$(call Package/$(PKG_NAME)/Default)
  SECTION:=libs
  CATEGORY:=MaxLinear
  TITLE+= (generic device library)
  SUBMENU:=DTI
  DEFAULT:=n
  DEPENDS:=lib-dti
endef

define Package/$(PKG_NAME)-device-generic/description
$(call Package/$(PKG_NAME)/Default/description)
 This package contains the DTI generic device library (for testing).
 - libdti_device_generic
endef

define Package/dti-agent
$(call Package/$(PKG_NAME)/Default)
  SECTION:=application
  CATEGORY:=MaxLinear
  TITLE+= (Default Agent)
  SUBMENU:=DTI
  DEFAULT:=n
  DEPENDS:=lib-dti
endef

define Package/dti-agent/description
$(call Package/$(PKG_NAME)/Default/description)
 This package builds a default DTI Agent.
 - dti_agent
endef

define Package/$(PKG_NAME)/config
	menu "configuration"
	depends on PACKAGE_$(PKG_NAME)
		source "$(SOURCE)/config/Config.in"
	endmenu
   $(call Package/$(PKG_NAME)/override_source_path)
endef

CONFIGURE_ARGS += \
	--enable-targetincl=$(STAGING_DIR)/usr/include \
	--enable-target=$(STAGING_DIR)/usr/lib \
	--enable-ifxos-include="-I$(STAGING_DIR)/usr/include/ifxos"\
	--enable-ifxos-library="-L$(STAGING_DIR)/usr/lib"\
	--enable-debug \
	--with-dti-tcp-port=$(strip $(CONFIG_COMMON_DTI_TCP_IP_PORT)) \
	$(call autoconf_bool,CONFIG_COMMON_DTI_ERROR_PRINTS,error-print) \
	$(call autoconf_bool,CONFIG_COMMON_DTI_CLI,cli) \
	$(call autoconf_bool,CONFIG_COMMON_DTI_STATISTICS,core-statistic) \
	$(call autoconf_bool,CONFIG_COMMON_DTI_GENERIC_DEVICE,dev-generic) \
	$(call autoconf_bool,CONFIG_COMMON_DTI_STANDALONE,dti-agent) \
	$(call autoconf_bool,CONFIG_COMMON_DTI_CLIENT_LIB,dti-client)


define Build/Clean
	@- rm -Rf $(PKG_BUILD_DIR)/ipkg*
	@- [ -e $(PKG_BUILD_DIR)/Makefile ] && $(MAKE) -C $(PKG_BUILD_DIR) clean
	$(call Build/Clean/Default)
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/{lib,include/dti}
	$(CP) $(PKG_BUILD_DIR)/src/include/* $(1)/usr/include/dti
	mkdir -p $(1)/usr/lib
	$(CP)	$(PKG_INSTALL_DIR)/usr/lib/*.{a,la,so*} $(1)/usr/lib/
endef


define Build/UninstallDev
	- rm -Rf $(1)/usr/include/dti
	- rm -f $(1)/usr/lib/libdti*.so*
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libdti_agent*.so.* $(1)/usr/lib/

ifeq ($(CONFIG_COMMON_DTI_CLIENT_LIB),y)
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libdti_client*.so.* $(1)/usr/lib/
endif

endef

define Package/$(PKG_NAME)-device-generic/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libdti_device_generic*.so.* $(1)/usr/lib/
endef

define Package/dti-agent/install
	$(INSTALL_DIR) $(1)/usr/bin $(1)/$(PKG_BIN_DIR)
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/dti_agent_main $(1)/$(PKG_BIN_DIR)
endef


$(eval $(call BuildPackage,$(PKG_NAME)))
$(eval $(call BuildPackage,$(PKG_NAME)-device-generic))
$(eval $(call BuildPackage,dti-agent))
