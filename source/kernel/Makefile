###############################################################################n
#    ./Makefile
#    Create By CaiZhiYong
#    2011.06.19
#
################################################################################
ifeq ($(CFG_HI_EXPORT_FLAG),)
SDK_DIR := $(shell cd $(CURDIR)/../.. && /bin/pwd)
include $(SDK_DIR)/base.mak
endif

CROSS_COMPILE   := $(HI_KERNEL_TOOLCHAINS_NAME)-
# atf always use aarch64 toolchain for hisilicon armv8 cpus
CROSS_COMPILE64 := $(CFG_HI_AARCH64_TOOLCHAINS_NAME)-
KERNEL          := $(CFG_HI_KERNEL_VERSION)
KERNEL_CONFIG   := $(CFG_HI_KERNEL_CFG)
CFG_FS_BUILDIN  := $(CFG_HI_FS_BUILDIN)

################################################################################
KERNEL_OUT_DIR := $(LINUX_DIR)
# KERNEL_PATCH_DIR ?= $(KERNEL_SRC_DIR).patch

ifeq ($(KERNEL_SRC_DIR),$(KERNEL_OUT_DIR))
BUILD_DIR :=
else
BUILD_DIR := $(KERNEL_OUT_DIR)

$(shell mkdir -p $(KERNEL_OUT_DIR))
endif

################################################################################
ifeq ($(findstring fpga_umap5, $(CFG_HI_BOOT_REG_NAME)), fpga_umap5)
CONFIG_HI3796MV2X_FPGA = 1
endif

ifeq ($(findstring fpga_umap6, $(CFG_HI_BOOT_REG_NAME)), fpga_umap6)
CONFIG_HI3796MV2X_FPGA = 1
endif

ifdef CFG_HI_ADVCA_FUNCTION
ifeq ($(CFG_HI_LOADER_APPLOADER),y)
INITRAMFS_SOURCE = $(KERNEL_DIR)/initramfs.cpio.gz
endif
else
INITRAMFS_SOURCE = $(HI_ROOTBOX_DIR)
endif

ifeq ($(CFG_HI_TEE_RESERVED_MEM)_$(CFG_HI_TEE_SUPPORT),y_y)
BUILD_TGT_IMG = $(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage
else
ifeq ($(CFG_HI_CPU_ARCH)_$(CFG_HI_TEE_SUPPORT),arm64_)
ifneq ($(CFG_ADVCA_VERIMATRIX_ULTRA),y)
FIP_SRC_DIR = $(KERNEL_DIR)/arm-trusted-firmware
FIP_IMG = $(FIP_OUT_DIR)/build/$(CFG_HI_CHIP_TYPE)/debug/fip.bin
BUILD_TGT_IMG = $(FIP_IMG)
else
BUILD_TGT_IMG = $(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage
endif
else
ifeq ($(CFG_HI_OPTEE_SUPPORT),y)
FIP_SRC_DIR = $(KERNEL_DIR)/arm-trusted-firmware
FIP_IMG = $(FIP_OUT_DIR)/build/$(CFG_HI_CHIP_TYPE)/debug/fip.bin
BUILD_TGT_IMG = $(FIP_IMG)
else
BUILD_TGT_IMG = $(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage
endif
endif
endif

################################################################################
CONFIG_TOOL=$(KERNEL_SRC_DIR)/scripts/config

################################################################################
# TVP Config                                                                   #
################################################################################
ifeq ($(CFG_HI_TEE_SUPPORT),y)

tee_config:
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_SUPPORT_CA
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_CA_WARKUP_CHECK
ifneq ($(findstring hi3796mv200 hi3716mv450,$(CFG_HI_CHIP_TYPE)),)
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_CA_SUPPORT_ADVCA
endif
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_TEE
ifeq ($(CFG_HI_OPTEE_SUPPORT),y)
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_OPTEE
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_OPTEE_BENCHMARK
# op-tee always enable the psci
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_ARM_PSCI
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_ARM_PSCI_FW
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_TEE_HISI
else
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_TEE_HISI
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_SECURE_EXTENSION
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_TZDRIVER
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_TEE_LIBHWSECUREC
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_DEVCHIP_PLATFORM
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_TEELOG
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_TC_DEBUG
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_TEE_TUI
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_HISI_SPI_SMC
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_OPTEE
endif
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_SUPPORT_ATF_MEM
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config $(if $(CFG_HI_TEE_RESERVED_MEM),--enable CONFIG_TEE_RESERVED_MEM,--disable CONFIG_TEE_RESERVED_MEM)
ifeq ($(CFG_HI_CHIP_TYPE)_$(CFG_HI_TEE_SUPPORT)_$(CFG_HI_ADVCA_TYPE), hi3798mv200_y_VERIMATRIX_ULTRA)
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_TEE_VMX_ULTRA
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_ARM_PSCI
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_ARM_PSCI_FW
else
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_TEE_VMX_ULTRA
endif
	$(if $(CFG_HI_TEE_RESERVED_MEM_ADDR),$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --set-val CONFIG_TEE_RESERVED_MEM_ADDR $(CFG_HI_TEE_RESERVED_MEM_ADDR),)
	$(if $(CFG_HI_TEE_RESERVED_MEM_SIZE),$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --set-val CONFIG_TEE_RESERVED_MEM_SIZE $(CFG_HI_TEE_RESERVED_MEM_SIZE),)
ifeq ($(CFG_HI_ARM32_ATF_SUPPORT), y)
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_ARM_PSCI
ifneq ($(CFG_HI_KERNEL_VERSION), linux-3.18.y)
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_ARM_PSCI_FW
endif
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_ARM32_ATF_SUPPORT
else
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_ARM32_ATF_SUPPORT
endif

else
DISABLE_TEE = 1
endif

ifeq ($(CFG_HI_BLUETOOTH_SUPPORT),y)
ifneq ($(CFG_HI_BLUETOOTH_DEVICE_CSR851X),y)
HI_CONFIG_BLUETOOTH_DEVICE = y
endif
endif
################################################################################

################################################################################
# PCIE Config                                                                  #
################################################################################
pcie_config:
ifeq ($(CFG_HI_PCIE_SUPPORT),y)
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_PCIE_RESET
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --set-val CONFIG_GPIO_PCIE_RESET $(CFG_HI_GPIO_PCIE_RESET)
ifeq ($(CFG_HI_GPIO_PCIE_POWER_EN_SUPPORT),y)
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --enable CONFIG_GPIO_PCIE_POWER_EN_SUPPORT
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --set-val CONFIG_GPIO_PCIE_POWER_EN $(CFG_HI_GPIO_PCIE_POWER_EN)
else
	$(CONFIG_TOOL) --file $(KERNEL_OUT_DIR)/.config --disable CONFIG_GPIO_PCIE_POWER_EN_SUPPORT
endif
endif
################################################################################

all: $(BUILD_TGT_IMG)
ifneq ($(CFG_HI_LOADER_APPLOADER),y)
ifeq ($(CFG_HI_NAND_FLASH_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_NAND_IMAGE_DIR)/hi_kernel.bin
endif
ifeq ($(CFG_HI_EMMC_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_EMMC_IMAGE_DIR)/hi_kernel.bin
endif
ifeq ($(CFG_HI_SPI_SUPPORT)_$(CFG_HI_EMMC_SUPPORT),y_y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_SPI_EMMC_IMAGE_DIR)/hi_kernel.bin
endif
ifeq ($(CFG_HI_SPI_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_SPI_IMAGE_DIR)/hi_kernel.bin
endif
endif
	$(AT)mkdir -p $(HI_MODULE_DIR)/usb
	$(AT)-cp -f $(KERNEL_OUT_DIR)/drivers/usb/host/*.ko $(HI_MODULE_DIR)/usb/
	$(AT)(for file in $(wildcard $(KERNEL_OUT_DIR)/drivers/ata/*.ko) ; do ( \
			mkdir -p $(HI_MODULE_DIR)/sata && cp $$file $(HI_MODULE_DIR)/sata/ \
		) done )
ifeq ($(HI_DRV_BUILDTYPE),m)
	$(AT)(find $(KERNEL_OUT_DIR)/drivers/common -name "*.ko" | xargs cp -f -t $(HI_MODULE_DIR)/)
	$(AT)(find $(KERNEL_OUT_DIR)/drivers/msp -name "*.ko" | xargs cp -f -t $(HI_MODULE_DIR)/)
ifeq ($(CFG_HI_IR_TYPE_LIRC),y)
	$(AT)make -j16 -C $(MSP_DIR)/drv/ir/lirc
endif
endif
ifeq ($(CFG_HI_WIFI_SUPPORT),y)
	$(AT)-cp -f $(KERNEL_OUT_DIR)/net/wireless/cfg80211.ko $(HI_MODULE_DIR)/
	$(AT)(find $(KERNEL_OUT_DIR)/drivers/wifi -name "*.ko" | xargs cp -f -t $(HI_MODULE_DIR)/)
endif
ifeq ($(HI_CONFIG_BLUETOOTH_DEVICE),y)
	$(AT)test -d $(HI_ROOTFS_DIR)/lib/firmware || mkdir -p $(HI_ROOTFS_DIR)/lib/firmware
	$(AT)(find $(KERNEL_OUT_DIR)/drivers/bluetooth_usb -name "*.ko" | xargs cp -f -t $(HI_MODULE_DIR)/)
	$(AT)(find $(SDK_DIR)/source/component/bluetooth -name "*_fw" | xargs cp -f -t $(HI_ROOTFS_DIR)/lib/firmware)
	$(AT)(find $(SDK_DIR)/source/component/bluetooth -name "*_config" | xargs cp -f -t $(HI_ROOTFS_DIR)/lib/firmware)
	$(AT)(find $(SDK_DIR)/source/component/bluetooth -name "*.bin" | xargs cp -f -t $(HI_ROOTFS_DIR)/lib/firmware)
endif

ifeq ($(CFG_HI_ADVCA_RUNTIME_CHECK_ENABLE),y)
	$(AT)(find $(KERNEL_OUT_DIR)/drivers/msp/runtime_check -name "*.ko" | xargs cp -f -t $(HI_MODULE_DIR)/)
endif

ifeq ($(CFG_HI_TEE_SUPPORT),y)
ATF_EXTRA_FLAGS = "-DCONFIG_TEE_SUPPORT -DCONFIG_TEE_MEM_LAYOUT_$(CFG_HI_TVP_MEM_LAYOUT)"
endif

ifeq ($(CFG_HI_OPTEE_SUPPORT),y)
SPD_NAME = opteed
else
SPD_NAME = tspd
endif

ifeq ($(DISABLE_TEE),1)
ifeq ($(CONFIG_HI3796MV2X_FPGA),1)
$(FIP_IMG): $(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage
	make -j16 -C $(FIP_SRC_DIR) CROSS_COMPILE=$(CROSS_COMPILE64) PLAT=$(CFG_HI_CHIP_TYPE) SPD=none CONFIG_HI3796MV2X_FPGA=$(CONFIG_HI3796MV2X_FPGA) BL33=$< DEBUG=1 DISABLE_TEE=$(DISABLE_TEE) fip O=$(FIP_OUT_DIR) ATF_EXT_FLAGS=$(ATF_EXTRA_FLAGS)
else
$(FIP_IMG): $(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage
	make -j16 -C $(FIP_SRC_DIR) CROSS_COMPILE=$(CROSS_COMPILE64) PLAT=$(CFG_HI_CHIP_TYPE) SPD=none BL33=$< DEBUG=1 DISABLE_TEE=$(DISABLE_TEE) fip O=$(FIP_OUT_DIR) ATF_EXT_FLAGS=$(ATF_EXTRA_FLAGS)
endif
else
ifeq ($(CONFIG_HI3796MV2X_FPGA),1)
$(FIP_IMG): $(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage
	make -j16 -C $(FIP_SRC_DIR) CROSS_COMPILE=$(CROSS_COMPILE64) PLAT=$(CFG_HI_CHIP_TYPE) CONFIG_HI3796MV2X_FPGA=$(CONFIG_HI3796MV2X_FPGA) NEED_BL32=no BL33=$< DEBUG=1 fip O=$(FIP_OUT_DIR) ATF_EXT_FLAGS=$(ATF_EXTRA_FLAGS) SPD=$(SPD_NAME)
else
$(FIP_IMG): $(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage
	make -j16 -C $(FIP_SRC_DIR) CROSS_COMPILE=$(CROSS_COMPILE64) PLAT=$(CFG_HI_CHIP_TYPE) NEED_BL32=no BL33=$< DEBUG=1 fip O=$(FIP_OUT_DIR) ATF_EXT_FLAGS=$(ATF_EXTRA_FLAGS) SPD=$(SPD_NAME)
endif
endif

$(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage: prepare
	make -j16 -C $(KERNEL_SRC_DIR) ARCH=$(CFG_HI_CPU_ARCH) \
		CROSS_COMPILE=$(CROSS_COMPILE) \
		O=$(BUILD_DIR) \
		CONFIG_MSP=y \
		HI_CONFIG_WIFI=$(CFG_HI_WIFI_SUPPORT) \
		HI_CONFIG_BLUETOOTH=$(HI_CONFIG_BLUETOOTH_DEVICE) \
		CONFIG_ADVCA=$(CFG_ADVCA) \
		$(EXTRA_FLAGS_KERNEL) \
		CFLAGS_KERNEL=$(CFLAGS_KERNEL) AFLAGS_KERNEL=$(AFLAGS_KERNEL) uImage modules

menuconfig: prepare
	make -j16 -C $(KERNEL_SRC_DIR) ARCH=$(CFG_HI_CPU_ARCH) \
		CROSS_COMPILE=$(CROSS_COMPILE) \
		O=$(BUILD_DIR) \
		$(EXTRA_FLAGS_KERNEL) \
		CFLAGS_KERNEL=$(CFLAGS_KERNEL) AFLAGS_KERNEL=$(AFLAGS_KERNEL) menuconfig

# $(KERNEL_SRC_DIR)/tar:
# 	@echo "*****************************************************************************************************************************"
# 	@echo "*****************************************************************************************************************************"
# ifeq ($(CFG_HI_KERNEL_VERSION), linux-3.18.y)
# 	@echo "please download linux3.18 from https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.18.24.tar.gz"
# 	@echo "copy linux-3.18.24.tar.gz into the directory 'sdk/third_party/open_source/' "
# else ifeq ($(CFG_HI_KERNEL_VERSION), linux-4.4.y)
# 	@echo "please download linux4.4 from https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.4.35.tar.gz"
# 	@echo "copy linux-4.4.35.tar.gz into the directory 'sdk/third_party/open_source/' "
# else ifeq ($(CFG_HI_KERNEL_VERSION), linux-4.9.y)
# 	@echo "please download linux4.9 from https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.9.44.tar.gz"
# 	@echo "copy linux-4.9.44.tar.gz into the directory 'sdk/third_party/open_source/' "
# endif
# 	@echo "*****************************************************************************************************************************"
# 	@echo "*****************************************************************************************************************************"
# ifeq ($(CFG_HI_KERNEL_VERSION), linux-3.18.y)
# 	$(AT)test -d $(KERNEL_SRC_DIR) || test ! -f $(SDK_DIR)/third_party/open_source/linux-3.18.24.tar.gz \
# 	|| tar -xzf $(SDK_DIR)/third_party/open_source/linux-3.18.24.tar.gz -C $(SDK_DIR)/source/kernel/
# 	$(AT)test ! -d linux-3.18.24 \
# 	|| mv $(SDK_DIR)/source/kernel/linux-3.18.24 $(SDK_DIR)/source/kernel/linux-3.18.y
# else ifeq ($(CFG_HI_KERNEL_VERSION), linux-4.4.y)
# 	$(AT)test -d $(KERNEL_SRC_DIR) || test ! -f $(SDK_DIR)/third_party/open_source/linux-4.4.35.tar.gz \
# 	|| tar -xzf $(SDK_DIR)/third_party/open_source/linux-4.4.35.tar.gz -C $(SDK_DIR)/source/kernel/
# 	$(AT)test ! -d linux-4.4.35 \
# 	|| mv $(SDK_DIR)/source/kernel/linux-4.4.35 $(SDK_DIR)/source/kernel/linux-4.4.y
# else
# 	$(AT)test -d $(KERNEL_SRC_DIR) || test ! -f $(SDK_DIR)/third_party/open_source/linux-4.9.44.tar.gz \
# 	|| tar -xzf $(SDK_DIR)/third_party/open_source/linux-4.9.44.tar.gz -C $(SDK_DIR)/source/kernel/
# 	$(AT)test ! -d linux-4.9.44 \
# 	|| mv $(SDK_DIR)/source/kernel/linux-4.9.44 $(SDK_DIR)/source/kernel/linux-4.9.y
# endif

# prepare: $(KERNEL_SRC_DIR)/tar
# 	$(AT)test ! -d $(KERNEL_PATCH_DIR) || cp -rf $(KERNEL_PATCH_DIR)/* $(KERNEL_SRC_DIR)/
# 	$(AT)test ! -f $(KERNEL_SRC_DIR)/.config || rm -f $(KERNEL_SRC_DIR)/.config
prepare:
ifeq ($(CFG_HI_NAND_FLASH_SUPPORT),y)
	$(AT)test -d $(HI_NAND_IMAGE_DIR) || mkdir -p $(HI_NAND_IMAGE_DIR)
endif
ifeq ($(CFG_HI_EMMC_SUPPORT),y)
	$(AT)test -d $(HI_EMMC_IMAGE_DIR) || mkdir -p $(HI_EMMC_IMAGE_DIR)
endif
ifeq ($(CFG_HI_SPI_SUPPORT)_$(CFG_HI_EMMC_SUPPORT),y_y)
	$(AT)test -d $(HI_SPI_EMMC_IMAGE_DIR) || mkdir -p $(HI_SPI_EMMC_IMAGE_DIR)
endif
ifeq ($(CFG_HI_SPI_SUPPORT),y)
	$(AT)test -d $(HI_SPI_IMAGE_DIR) || mkdir -p $(HI_SPI_IMAGE_DIR)
endif
	$(AT)(cd $(KERNEL_SRC_DIR)/drivers && rm -rf common && ln -s $(COMMON_DIR)/drv common)
	$(AT)(cd $(KERNEL_SRC_DIR)/drivers && rm -rf msp && ln -s $(MSP_DIR)/drv msp)
	$(AT)(cd $(KERNEL_SRC_DIR)/drivers && rm -rf wifi && ln -s $(COMPONENT_DIR)/wifi/drv wifi)
	$(AT)(cd $(KERNEL_SRC_DIR)/drivers && rm -rf bluetooth_usb && ln -s $(COMPONENT_DIR)/bluetooth/drv bluetooth_usb)

ifdef CFG_HI_ADVCA_FUNCTION
ifeq ($(CFG_HI_LOADER_APPLOADER),y)
$(INITRAMFS_SOURCE): force
	@rm -f $(INITRAMFS_SOURCE)
	@cd $(HI_ROOTFS_DIR) && \
	    find . | $(CPIO) -o -H newc -T $(RULEFILE) -G $(RULEDBG) | gzip > $(INITRAMFS_SOURCE) && \
        cd -

force: ;
endif
endif

fs_buildin: prepare $(INITRAMFS_SOURCE)
	make -j16 -C $(KERNEL_SRC_DIR) ARCH=$(CFG_HI_CPU_ARCH) \
		CROSS_COMPILE=$(CROSS_COMPILE) \
		O=$(BUILD_DIR) \
		CONFIG_MSP=y \
		HI_CONFIG_WIFI=$(CFG_HI_WIFI_SUPPORT) \
		HI_CONFIG_BLUETOOTH=$(HI_CONFIG_BLUETOOTH_DEVICE) \
		CONFIG_ADVCA=$(CFG_ADVCA) \
		$(EXTRA_FLAGS_KERNEL) \
		CFLAGS_KERNEL=$(CFLAGS_KERNEL) \
		AFLAGS_KERNEL=$(AFLAGS_KERNEL) \
		CONFIG_CC_OPTIMIZE_FOR_SIZE=$(CONFIG_CC_OPTIMIZE_FOR_SIZE) \
		CONFIG_INITRAMFS_SOURCE=$(INITRAMFS_SOURCE) \
		uImage
ifeq ($(CFG_HI_CPU_ARCH),arm64)
	make -j16 -C $(FIP_SRC_DIR) CROSS_COMPILE=$(CROSS_COMPILE) PLAT=$(CFG_HI_CHIP_TYPE) $(if $(DISABLE_TEE), SPD=none,) BL33=$(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage DEBUG=1 DISABLE_TEE=$(DISABLE_TEE) CONFIG_HI3796MV2X_FPGA=$(CONFIG_HI3796MV2X_FPGA) fip O=$(FIP_OUT_DIR)
endif
ifeq ($(CFG_HI_LOADER_APPLOADER),y)
ifeq ($(CFG_HI_NAND_FLASH_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_NAND_IMAGE_DIR)/apploader.bin
endif
ifeq ($(CFG_HI_EMMC_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_EMMC_IMAGE_DIR)/apploader.bin
endif
ifeq ($(CFG_HI_SPI_SUPPORT)_$(CFG_HI_EMMC_SUPPORT),y_y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_SPI_EMMC_IMAGE_DIR)/apploader.bin
endif
ifeq ($(CFG_HI_SPI_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_SPI_IMAGE_DIR)/apploader.bin
endif
else
ifeq ($(CFG_HI_NAND_FLASH_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_NAND_IMAGE_DIR)/hi_kernel.bin
endif
ifeq ($(CFG_HI_EMMC_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_EMMC_IMAGE_DIR)/hi_kernel.bin
endif
ifeq ($(CFG_HI_SPI_SUPPORT)_$(CFG_HI_EMMC_SUPPORT),y_y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_SPI_EMMC_IMAGE_DIR)/hi_kernel.bin
endif
ifeq ($(CFG_HI_SPI_SUPPORT),y)
	$(AT)cp -f $(BUILD_TGT_IMG) $(HI_SPI_IMAGE_DIR)/hi_kernel.bin
endif
endif

################################################################################

distclean: clean
	$(AT)make -j16 -C $(KERNEL_SRC_DIR) ARCH=$(CFG_HI_CPU_ARCH) \
		CROSS_COMPILE=$(CROSS_COMPILE) \
		O=$(BUILD_DIR) \
		$(EXTRA_FLAGS_KERNEL) \
		CFLAGS_KERNEL=$(CFLAGS_KERNEL) \
		AFLAGS_KERNEL=$(AFLAGS_KERNEL) distclean
ifeq ($(CFG_HI_CPU_ARCH),arm64)
	$(AT)make -j16 -C $(FIP_SRC_DIR) CROSS_COMPILE=$(CROSS_COMPILE) PLAT=$(CFG_HI_CHIP_TYPE) $(if $(DISABLE_TEE), SPD=none,) BL33=$(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage DEBUG=1 DISABLE_TEE=$(DISABLE_TEE) O=$(FIP_OUT_DIR) distclean
endif

clean:
	$(AT)make -j16 -C $(KERNEL_SRC_DIR) ARCH=$(CFG_HI_CPU_ARCH) \
		CROSS_COMPILE=$(CROSS_COMPILE) \
		O=$(BUILD_DIR) \
		$(EXTRA_FLAGS_KERNEL) \
		CFLAGS_KERNEL=$(CFLAGS_KERNEL) \
		AFLAGS_KERNEL=$(AFLAGS_KERNEL) clean
ifeq ($(CFG_HI_CPU_ARCH),arm64)
	$(AT)make -j16 -C $(FIP_SRC_DIR) CROSS_COMPILE=$(CROSS_COMPILE) PLAT=$(CFG_HI_CHIP_TYPE) $(if $(DISABLE_TEE), SPD=none,) BL33=$(KERNEL_OUT_DIR)/arch/$(CFG_HI_CPU_ARCH)/boot/uImage DEBUG=1 DISABLE_TEE=$(DISABLE_TEE) O=$(FIP_OUT_DIR) distclean
endif
ifdef CFG_HI_ADVCA_FUNCTION
ifeq ($(CFG_HI_LOADER_APPLOADER),y)
	@test ! -f $(INITRAMFS_SOURCE) || rm -f $(INITRAMFS_SOURCE)
endif
endif
ifeq ($(CFG_HI_NAND_FLASH_SUPPORT),y)
	$(AT)rm -rf $(HI_NAND_IMAGE_DIR)/hi_kernel.bin $(HI_NAND_IMAGE_DIR)/apploader.bin
endif
ifeq ($(CFG_HI_EMMC_SUPPORT),y)
	$(AT)rm -rf $(HI_EMMC_IMAGE_DIR)/hi_kernel.bin $(HI_EMMC_IMAGE_DIR)/apploader.bin
endif
ifeq ($(CFG_HI_SPI_SUPPORT)_$(CFG_HI_EMMC_SUPPORT),y_y)
	$(AT)rm -rf $(HI_SPI_EMMC_IMAGE_DIR)/hi_kernel.bin $(HI_SPI_EMMC_IMAGE_DIR)/apploader.bin
endif
ifeq ($(CFG_HI_SPI_SUPPORT),y)
	$(AT)rm -rf $(HI_SPI_IMAGE_DIR)/hi_kernel.bin $(HI_SPI_IMAGE_DIR)/apploader.bin
endif
ifeq ($(CFG_HI_IR_TYPE_LIRC),y)
	$(AT)make -j16 -C $(MSP_DIR)/drv/ir/lirc clean
endif
	$(AT)rm -rf $(HI_MODULE_DIR)

#### for android ####
linux_prepare:
	$(AT)(cd $(LINUX_DIR)/drivers && rm -rf common && ln -s $(COMMON_DIR)/drv common)
	$(AT)(cd $(LINUX_DIR)/drivers && rm -rf msp && ln -s $(MSP_DIR)/drv msp)
	$(AT)(cd $(LINUX_DIR)/drivers && rm -rf wifi && ln -s $(COMPONENT_DIR)/wifi/drv wifi)
	$(AT)(cd $(LINUX_DIR)/drivers && rm -rf bluetooth_usb && ln -s $(COMPONENT_DIR)/bluetooth/drv bluetooth_usb)

linux_unprepare:
	$(AT)rm -rf $(LINUX_DIR)/drivers/common $(LINUX_DIR)/drivers/msp $(LINUX_DIR)/drivers/wifi $(LINUX_DIR)/drivers/bluetooth_usb

################################################################################
.PHONY: all clean distclean prepare linux_prepare linux_unprepare
################################################################################
