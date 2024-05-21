# Copyright (C) 2024-2024 756yang
#
# This is free software, licensed under the GNU General Public License v3.

include $(TOPDIR)/rules.mk

PKG_NAME:=udp2raw-simple
PKG_VERSION:=1.0.0
PKG_RELEASE:=1

PKG_LICENSE:=GPL-3.0-only
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=756yang

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Tunnel which turns UDP Traffic into Encrypted FakeTCP/UDP/ICMP Traffic
  URL:=https://github.com/wangyu-/udp2raw-tunnel
endef

define Package/$(PKG_NAME)/description
udp2raw-tunnel is a tunnel which turns UDP Traffic into Encrypted FakeTCP/UDP/ICMP Traffic by using Raw Socket.
endef

# makefile语法不支持多行shell命令，因而以定义多行变量
define Build_Makefile_x86
cross2:
	@echo using binary file for udp2raw-tunnel.
	wget -qO- https://github.com/wangyu-/udp2raw/releases/latest/download/udp2raw_binaries.tar.gz | tar -xvzf -
	mv `find . -name "*$(if $(subst x86_64,,$(ARCH)),x86*aes,amd64*aes)*"` udp2raw_cross
endef

# makefile要输出多行变量的值，必须借由shell环境变量
export V_Build_Makefile_x86=$(call Build_Makefile_x86)

# 在openwrt的软件包宏块中引用环境变量，需要四个'$'字符
define Build_Configure_x86
	mkdir "$(PKG_BUILD_DIR)/udp2raw-tunnel"
	echo "$$$$V_Build_Makefile_x86" > "$(PKG_BUILD_DIR)/udp2raw-tunnel/makefile"
endef

define Build_Configure_other
	git clone --depth 1 https://github.com/wangyu-/udp2raw-tunnel.git "$(PKG_BUILD_DIR)/udp2raw-tunnel"
	$(SED) 's/cc_cross[[:space:]]*=.*/cc_cross=$(TARGET_CXX)/' "$(PKG_BUILD_DIR)/udp2raw-tunnel/makefile"
endef

MAKE_FLAGS += cross2
MAKE_PATH:=udp2raw-tunnel

define Build/Configure
	$(if $(ARCH),\
		$(if $(subst i386,,$(subst x86_64,,$(ARCH))),\
			$(call Build_Configure_other), \
			$(call Build_Configure_x86) \
		), \
		$(call Build_Configure_other) \
	)
endef

define Package/$(PKG_NAME)/conffiles
/etc/config/udp2raw
/etc/udp2raw/udp2raw.conf
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) "$(1)/usr/bin"
	$(INSTALL_BIN) "$(PKG_BUILD_DIR)/udp2raw-tunnel/udp2raw_cross" "$(1)/usr/bin/udp2raw"
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/etc/config/udp2raw $(1)/etc/config/udp2raw
	$(INSTALL_DIR) "$(1)/etc/init.d"
	$(INSTALL_BIN) ./files/etc/init.d/udp2raw "$(1)/etc/init.d/udp2raw"
	$(INSTALL_DIR) "$(1)/etc/udp2raw"
	$(INSTALL_CONF) ./files/etc/udp2raw/udp2raw.conf "$(1)/etc/udp2raw/udp2raw.conf"
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
