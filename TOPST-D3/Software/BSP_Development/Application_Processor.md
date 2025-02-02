# 1 Application Processor

This chapter describes how to develop U-Boot and Linux Kernel for CA72 and CA53.

## 1.1 U-Boot Installation

### 1.1.1 Main Core (CA72)

This chapter describes the recipe and source build method in Yocto for CA72.

### 1.1.1.1 U-Boot Recipe

### 1.1.1.1.1 How to locate U-Boot Recipe and Source in Yocto

**u-boot-tcc.bb** is a recipe created by adding functionality for TOPST to the U-Boot open source.

```bash
$ cd poky/meta-telechips-bsp/recipes-bsp/u-boot
$ ls
$ u-boot-tcc  u-boot-tcc.bb
```


The following describes the features to be applied when building U-boot in Yocto for main core.

```bash
$ cat linux-telechips.inc

SUMMARY = "Universal Boot Loader for embedded devices"
HOMEPAGE = "http://www.denx.de/wiki/U-Boot/WebHome"
SECTION = "bootloaders"
PROVIDES = "virtual/bootloader"

LICENSE = "GPLv2+"
LIC_FILES_CHKSUM = "file://Licenses/README;md5=30503fd321432fc713238f582193b78e"

SRC_URI = "${TELECHIPS_AUTOMOTIVE_BSP_GIT}/u-boot.git;protocol=${ALS_GIT_PROTOCOL};branch=${ALS_BRANCH}"

SRC_URI += "${@bb.utils.contains('INVITE_PLATFORM', 'hud-display', 'file://add-dp.cfg', '', d)}"
SRC_URI += "${@bb.utils.contains('INVITE_PLATFORM', 'early-camera', 'file://add-earlycam.cfg', '', d)}"
SRC_URI += "${@bb.utils.contains('INVITE_PLATFORM', 'gpu-vz', 'file://gpu-vz.cfg', '', d)}"

# DP to HDMI (1920x1080)
SRC_URI += "${@bb.utils.contains('INVITE_PLATFORM', 'dp2hdmi', 'file://add-dp2hdmi.cfg', '', d)}"

SRC_URI_append_tcc803x = " ${@bb.utils.contains('INVITE_PLATFORM', 'with-subcore', 'file://add-subcore.cfg', '', d)}"
SRC_URI_append_tcc803x = " ${@bb.utils.contains('INVITE_PLATFORM', 'cluster-display', 'file://add-dlvds.cfg', '', d)}"

#SRCREV = "3965996feb533416e55780924154f16fd5dc7735"
SRCREV = "${AUTOREV}"

S = "${WORKDIR}/git"
B = "${S}"

PACKAGE_ARCH = "${MACHINE_ARCH}"

inherit uboot-config deploy

DEPENDS += "bison-native"

EXTRA_OEMAKE = 'CROSS_COMPILE=${TARGET_PREFIX} CC="${TARGET_PREFIX}gcc ${TOOLCHAIN_OPTIONS}" V=1'
EXTRA_OEMAKE += 'HOSTCC="${BUILD_CC} ${BUILD_CFLAGS} ${BUILD_LDFLAGS}"'
EXTRA_OEMAKE += 'STAGING_INCDIR=${STAGING_INCDIR_NATIVE} STAGING_LIBDIR=${STAGING_LIBDIR_NATIVE}'

UBOOT_ARCH_ARGS_arm = "ARCH=arm"
UBOOT_ARCH_ARGS_aarch64 = "ARCH=arm64 "

SNOR_BOOT_NAME = "${TCC_ARCH_FAMILY}_snor_boot.rom"

PATCHTOOL = "git"

def find_cfgs(d):
    sources=src_patches(d, True)
    sources_list=[]
    for s in sources:
        if s.endswith('.cfg'):
            sources_list.append(s)

    return sources_list

do_configure_append() {
        if [ "${@bb.utils.contains('DISTRO_FEATURES', 'ld-is-gold', 'ld-is-gold', '', d)}" = "ld-is-gold" ] ; then
                sed -i 's/$(CROSS_COMPILE)ld$/$(CROSS_COMPILE)ld.bfd/g' ${S}/config.mk
        fi

        unset LDFLAGS
        unset CFLAGS
        unset CPPFLAGS
        if [ -n "${EXTERNALSRC}" ] ; then
                export KBUILD_OUTPUT=${EXTERNALSRC}/${MACHINE}/
        fi
        export ${UBOOT_ARCH_ARGS} DEVICE_TREE=${UBOOT_DEVICE_TREE}
        oe_runmake ${UBOOT_MACHINE}
        if [ -n "${EXTERNALSRC}" ] ; then
                ${S}/scripts/kconfig/merge_config.sh -O ${KBUILD_OUTPUT} -m ${KBUILD_OUTPUT}.config ${@" ".join(find_cfgs(d))}
        else
                ${S}/scripts/kconfig/merge_config.sh -m .config ${@" ".join(find_cfgs(d))}
        fi
}

do_compile() {
        unset LDFLAGS
        unset CFLAGS
        unset CPPFLAGS
        if [ -n "${EXTERNALSRC}" ] ; then
                export KBUILD_OUTPUT=${EXTERNALSRC}/${MACHINE}/
        fi
        export ${UBOOT_ARCH_ARGS} DEVICE_TREE=${UBOOT_DEVICE_TREE}

        oe_runmake
}

do_install() {
        install -d ${D}/boot

        if [ -n "${EXTERNALSRC}" ] ; then
                export KBUILD_OUTPUT=${EXTERNALSRC}/${MACHINE}/
                install -m 0644 ${KBUILD_OUTPUT}${UBOOT_NAME}.${UBOOT_SUFFIX}   ${D}/boot/${UBOOT_IMAGE}
        else
                install -m 0644 ${S}/${UBOOT_NAME}.${UBOOT_SUFFIX}      ${D}/boot/${UBOOT_IMAGE}
        fi

        if ${@bb.utils.contains('INVITE_PLATFORM', 'early-camera', 'true', 'false', d)}; then
                install -m 0644 ${S}/drivers/camera/splash/${SPLASH_IMAGE}  ${D}/boot/${SPLASH_IMAGE}
        fi
}

FILES_${PN} = "/boot"

do_deploy() {
    install -d ${DEPLOYDIR}
    install -m 0644 ${D}/boot/${UBOOT_IMAGE} ${DEPLOYDIR}

        cd ${DEPLOYDIR}
    rm -f ${UBOOT_BINARY} ${UBOOT_SYMLINK}
    ln -sf ${UBOOT_IMAGE} ${UBOOT_BINARY}
    ln -sf ${UBOOT_IMAGE} ${UBOOT_SYMLINK}
        cd -

        if ${@bb.utils.contains('INVITE_PLATFORM', 'early-camera', 'true', 'false', d)}; then
        install -m 0644 ${D}/boot/${SPLASH_IMAGE} ${DEPLOYDIR}
    fi
}

do_clean_extworkdir() {
        if [ -n "${EXTERNALSRC}" ]; then
                rm -rf ${EXTERNALSRC}/arch/*/include/asm/telechips
                rm -rf ${EXTERNALSRC}/${MACHINE}
        fi
}

addtask deploy before do_build after do_install
addtask clean_extworkdir after do_buildclean before do_clean
```


### 1.1.1.2 Modify U-Boot Source

The following are the settings for using ***BitBake*** to modify the U-Boot source code in Yocto for the main core.

```bash
$ source poky/oe-init-build-env build/tcc8050-main/

Yocto Project common targets are:
    core-image-minimal
    core-image-sato
    meta-toolchain
    adt-installer
    meta-ide-support

Telechips common targets are:
    telechips-ivi-image-minimal
    telechips-ivi-image-gstreamer(minimal + GStreamer)
    telechips-ivi-image-qt(minimal + Qt)
    telechips-ivi-image(minimal + GStreamer + Qt)

    meta-toolchain-telechips(Application Development Toolkit)
    meta-toolchain-telechips-ivi(Application Development Toolkit include GStreamer)
    meta-toolchain-telechips-qt5(Application Development Toolkit include GStreamer and Qt5)

Telechips Automotive Linux Platform targets are:
    automotive-linux-platform-image(telechips-ivi-image + demo applications)

You can also run generated qemu images with a command like 'runqemu qemux86'
 or
You can also run generated Telechips images on Telechips EVB Boards 

Other commonly useful commands are:
 - 'devtool' and 'recipetool' handle common recipe tasks
 - 'bitbake-layers' handles common layer tasks
 - 'oe-pkgdata-util' handles common target package tasks
Warning: source-mirror not exist!!
         The build to be slower or fail because will be download source code from upstream before build.
```


Use ***devtool*** to download the **u-boot-tcc** source code to workspace.

You can use the ***BitBake*** **-e** option to find out the source path as follows:

```bash
$ devtool modify u-boot-tcc

$ bitbake u-boot-tcc -e | grep ^S=
S=”/home/TOPST/build/tcc8050-main/workspace/sources/u-boot-tcc”

$ cd /home/TOPST/build/tcc8050-main/workspace/sources/u-boot-tcc
$ ls
api   board  common     configs  doc      dts  examples  include  Kconfig  Licenses     Makefile  oe-local-files  README   test
arch  cmd    config.mk  disk     drivers  env  fs        Kbuild   lib      MAINTAINERS  net       post            scripts  tools
```


**U-boot-tcc** can use the **Cleanall** or **build** function by using the ***BitBake* -c** option. However, the function must be defined as the task in the recipe.

```bash
$ bitbake u-boot-tcc -c cleanall

$ bitbake u-boot-tcc -c build
```


The following is an example that describes how ty using Git and create a patch file.

```bash
$ vi arch/arm/dts/tcc8050-ivi-TOPST_sv0.1.dts

$ git status
On branch tcc8050_linux_ivi_TOPST
Your branch is up to date with 'origin/tcc8050_linux_ivi_TOPST'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   arch/arm/dts/tcc8050-ivi-TOPST_sv0.1.dts

$ git commit -s -m <title of your commit>
$ git format-patch -1
```

1.  Add patch file in the **u-boot-tcc**

```bash
$ cp 001-test-dts.patch poky/meta-telechips-bsp/recipes-bsp/u-boot/u-boot-tcc/tcc8050-main/

$ ls poky/meta-telechips-bsp/recipes-bsp/u-boot/u-boot-tcc/tcc8050-main/
add-dp2hdmi.cfg  add-dpAutoMode.cfg  add-dp.cfg 001-test-dts.patch

$ vi poky/meta-telechips-bsp/recipes-bsp/u-boot/u-boot-tcc/u-boot-tcc.bb
SRC_URI += “file://001-test-dts.patch”
```

2.  Re-build and execute U-Boot.

If you use the ***BitBake*** **-c** **build** option, the U-Boot binary is automatically saved as a path for binary downloads after compilation.

```bash
$ bitbake u-boot-tcc -c cleansstate

$ bitbake u-boot-tcc -c build

$ ls ~/TOPST/build/tcc8050-main/workspace/sources/u-boot-tcc/ca72_bl3.rom
ca72_bl3.rom
```


### 1.1.1.3 Configure U-Boot

The following shows how to use the U-Boot **menuconfig**.

```bash
$ make tcc805x_defconfig

$ make menuconfig
```
<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/3999b02e-fe6f-4e2f-9f1f-5bfd4ec5b5f5" width="750" height="400" >
</p>

```bash
$ bitbake u-boot-tcc -c build

$ ls ~/TOPST/build/tcc8050-main/workspace/sources/u-boot-tcc/ca72_bl3.rom
ca72_bl3.rom
```


### 1.1.1.4 Device Tree for U-Boot

Configure the U-Boot device tree.

The following is information in the device tree file for developing the U-Boot device driver.

```bash
$ grep tcc8050-ivi-TOPST poky/meta-telechips-bsp/conf/machine/tcc8050-main.conf
UBOOT_DEVICE_TREE = "tcc8050-ivi-TOPST_sv0.1"
```


The following is the device tree file in the U-Boot source code of the main core.

```bash
$ cd ~/TOPST/build/tcc8050-main/workspace/sources/u-boot-tcc/
$ cat arch/arm/dts/tcc8050-ivi-TOPST_sv0.1.dts

// SPDX-License-Identifier: (GPL-2.0-or-later OR MIT)
/*
 * Copyright (C) Telechips Inc.
 */

/dts-v1/;

#include "tcc805x.dtsi"
#include "tcc8050_53-pinctrl.dtsi"

/ {
	model = "Telechips TCC8050 IVI TOPST SV0.1";
	board-id = <TCC_BOARD_ID_TCC8050_EVB>;
	board-rev = <0x0>;
	interrupt-parent = <&gic0>;

	sdhc2_pwrseq: sdhc2_pwrseq {
		compatible = "mmc-pwrseq-simple";
		reset-gpios = <&gpsd1 8 GPIO_ACTIVE_LOW>;
	};
};

&chosen {
	stdout-path = &uart0;
};

&cpus {
	cpu@0 {
		status = "okay";
	};
};

&gic0 {
	status = "okay";
};

&uart0 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart18_data>;
	status = "okay";
	u-boot,dm-pre-reloc;
};

&mbox0 {
	status = "okay";
	u-boot,dm-pre-reloc;
};

&tcc_sc_fw {
	status = "okay";
	mboxes = <&mbox0 0>;
	u-boot,dm-pre-reloc;

	tcc_sc_mmc {
		status = "okay";
		u-boot,dm-pre-reloc;
	};
};

/* eMMC */
&sdhc0 {
	pinctrl-names = "default";
	pinctrl-0 = <&sd0_clk>, <&sd0_cmd>, <&sd0_bus8>, <&sd0_strb>;
	bus-width = <8>;
	status = "disabled";

	tcc-mmc-taps = <0xF 0xF 0xF 0xF>;
	tcc-mmc-hs400-pos-tap = <0x6>;
	tcc-mmc-hs400-neg-tap = <0xB>;

	non-removable;
	mmc-hs200-1_8v;
	mmc-hs400-1_8v;
};

/* SD Slot */
&sdhc2 {
	pinctrl-names = "default";
	pinctrl-0 = <&sd2_clk>, <&sd2_cmd>, <&sd2_bus4>;
	status = "okay";

	mmc-pwrseq = <&sdhc2_pwrseq>;

	non-removable;
};

&i2c0 {
	pinctrl-names = "default";
	pinctrl-0 = <&i2c37_bus>;
	port-mux = <37>;
	status = "okay";
};

&i2c1 {
	pinctrl-names = "default";
	pinctrl-0 = <&i2c22_bus>;
	port-mux = <22>;
	status = "okay";

	pmic: da9131 {
		compatible	= "dlg,da9131";
		reg		= <0x68>;
	};
};

&i2c7 {
	pinctrl-names = "default";
	pinctrl-0 = <&i2c12_bus>;
	port-mux = <12>;
	status = "disabled";

	videosource0: videodecoder_adv7182 {
		compatible	= "analogdevices,adv7182";
		pinctrl-names	= "idle", "active";
		pinctrl-0	= <&cam0_idle>;
		pinctrl-1	= <&cam0_rst &cam0_clk &cam0_hsync &cam0_vsync
				   &cam0_fld &cam0_de &cam0_data>;
		rst-gpios	= <&gpb 14 GPIO_ACTIVE_HIGH>;
		reg		= <0x21>;	// 0x42 >> 1
		cifport		= <0>;
	};
#if 1
	videosource2: deserializer_max9276 {
		compatible	= "maxim,max9276";
		pinctrl-names	= "idle", "active";
		pinctrl-0	= <&cam0_idle>;
		pinctrl-1	= <&cam0_rst &cam0_clk &cam0_hsync &cam0_vsync
				   &cam0_fld &cam0_de &cam0_data>;
		rst-gpios	= <&gpb 14 GPIO_ACTIVE_HIGH>;
		reg		= <0x4A>;	// 0x94 >> 1
		cifport		= <0>;
	};
#else
	videosource2: deserializer_max9276 {
		compatible	= "maxim,max9276";
		pinctrl-names	= "idle", "active";
		pinctrl-0	= <&cam1_idle>;
		pinctrl-1	= <&cam1_rst &cam1_clk &cam1_hsync &cam1_vsync
				   &cam1_fld &cam1_de &cam1_data>;
		rst-gpios	= <&gpma 21 GPIO_ACTIVE_HIGH>;
		reg		= <0x4A>;	// 0x94 >> 1
		cifport		= <1>;
	};
#endif
};

&switch0 {
	status = "okay";

	pinctrl-names = "default";
	pinctrl-0 = <&switch_mb23>;

	switch-gpios = <&gpmb 23 1>;
	switch-active = <1>;
};

&dwc3_phy {
	status = "okay";
};

&dwc3_platform {
	status = "okay";
};

&dwc3 {
	pinctrl-names = "vbus_on", "vbus_off";
	pinctrl-0 = <&usb30_vbus_pwr_ctrl_on_sv01>;
	pinctrl-1 = <&usb30_vbus_pwr_ctrl_off_sv01>;
};

&ehci_phy {
	status = "okay";
};

&mhst_phy {
	status = "okay";
};

&ehci {
	pinctrl-names = "vbus_on", "vbus_off";
	pinctrl-0 = <&usb20_vbus_pwr_ctrl_on_sv01>;
	pinctrl-1 = <&usb20_vbus_pwr_ctrl_off_sv01>;
	status = "okay";
};

&ehci_mux {
	pinctrl-names = "vbus_on", "vbus_off";
	pinctrl-0 = <&usb20_mux_vbus_pwr_ctrl_on_sv01>;
	pinctrl-1 = <&usb20_mux_vbus_pwr_ctrl_off_sv01>;
	status = "okay";
};

&ohci {
	status = "okay";
};

&ohci_mux {
	status = "okay";
};

&dwc_otg_phy {
	status = "okay";
};

&dwc_otg {
	pinctrl-names = "vbus_on", "vbus_off";
	pinctrl-0 = <&usb20_mux_vbus_pwr_ctrl_on_sv01>;
	pinctrl-1 = <&usb20_mux_vbus_pwr_ctrl_off_sv01>;
	status= "okay";
};

&tcc_lcd_interface{
	board-type = <1>;
	status = "okay";
};

&ufs{
	status = "okay";
};
```


The following is information from the driver file developed for TOPST.

```bash
$ cd ~/TOPST/build/tcc8050-main/workspace/sources/u-boot-tcc/

$ find drivers/ | grep tcc

drivers/mmc/tcc_sdhci.c
drivers/mmc/tcc_sc_mmc.c
drivers/spi/tcc_spi.c
drivers/power/domain/tcc-power-domain.c
drivers/watchdog/tcc_wdt.c
drivers/watchdog/tcc_pmu_wdt.c
drivers/camera/splash/splash_1920x720x32_tcc803xp.img
drivers/timer_irq/tcc_timer.c
drivers/adc/tcc_adc.c
drivers/clk/tcc_clk.c
drivers/clk/tcc_dm_clk.c
drivers/phy/phy-tcc-dwc2.c
drivers/phy/phy-tcc-ehci.c
drivers/phy/phy-tcc-dwc3.c
drivers/pwm/pwm-tcc.c
drivers/pwm/pwm-tcc-dm.c
drivers/net/tcc_gmac.c
drivers/video/tcc
drivers/video/tcc/lcd_SLVDS_SAMPLE.c
drivers/video/tcc/lcd_HDMIV20.h
drivers/video/tcc/tcc_lcd_interface.c
drivers/video/tcc/tcc_hdmi_v1_4.c
drivers/video/tcc/tcc_lvds_ctrl.c
drivers/video/tcc/lcd_HDMIV20.c
drivers/video/tcc/tcc_hdmi_v2_0.c
drivers/video/tcc/tcc_fb.c
drivers/video/tcc/Kconfig
drivers/video/tcc/lcd_TM123XDHP90.c
drivers/video/tcc/lcd_FLD0800.c
drivers/video/tcc/Makefile
drivers/video/tcc/tcc_hdmi_v2_0.h
drivers/video/tcc/lcd_DPV14.c
drivers/video/tcc/lcd_SLVDS_EXT.c
drivers/i2c/tcc_i2c.c
drivers/mailbox/tcc805x_multi_mbox.c
drivers/mailbox/tcc803x_multi_mbox.c
drivers/mailbox/tcc-mbox.c
drivers/ufs/tcc-ufs.h
drivers/ufs/tcc-ufs.c
drivers/ufs/tcc_sc_ufs.c
drivers/pinctrl/pinctrl-tcc.c
drivers/gpio/tcc_legacy_gpio.c
drivers/gpio/tcc_gpio.c
drivers/firmware/tcc_sc_fw.c
drivers/firmware/tcc_sc_fw.h
drivers/tav/include/soc/tcc803x
drivers/tav/include/soc/tcc803x/tav_rdma.h
drivers/tav/include/soc/tcc803x/tav_wdma.h
drivers/tav/include/soc/tcc803x/tav_fifo.h
drivers/tav/include/soc/tcc803x/tav_wmix.h
drivers/tav/include/soc/tcc803x/tav_scaler.h
drivers/tav/include/soc/tcc803x/tav_fdly.h
drivers/tav/include/soc/tcc805x
drivers/tav/include/soc/tcc805x/tav_rdma.h
drivers/tav/include/soc/tcc805x/tav_wdma.h
drivers/tav/include/soc/tcc805x/tav_fifo.h
drivers/tav/include/soc/tcc805x/tav_wmix.h
drivers/tav/include/soc/tcc805x/tav_scaler.h
drivers/tav/include/soc/tcc805x/tav_fdly.h
```

### 1.1.2 Sub-core (CA53)

This chapter describes the recipe and source build method in Yocto for CA53.

### 1.1.2.1 U-Boot Recipe

### 1.1.2.1.1 How to locate U-Boot Recipe and Source in Yocto

**u-boot-tcc.bb** is a recipe created by adding functionality for TOPST to the U-Boot open source.

```bash
$ cd poky/meta-telechips/meta-subcore/recipes-bsp/u-boot
$ ls
$ u-boot-tcc  u-boot-tcc.bbappend
```


The following describes the features to be applied when building U-Boot in Yocto for the sub-core.

```bash
$ cat linux-telechips.inc

FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}:"

SRC_URI += " ${@bb.utils.contains('SUBCORE_APPS', 'cluster', 'file://add-dlvds.cfg', '', d)} "
```


### 1.1.2.2 Modify U-Boot Source

The following are the settings for using ***BitBake*** to modify the U-Boot source code in Yocto for the sub-core.

```bash
$ source poky/oe-init-build-env build/tcc8050-sub/

Yocto Project common targets are:
    core-image-minimal
    core-image-sato
    meta-toolchain
    adt-installer
    meta-ide-support

Telechips subcore targets are:
    meta-toolchain-telechips(Application Development Toolkit)
    telechips-subcore-image(minimal rootfs for Cortex-A53)

You can also run generated qemu images with a command like 'runqemu qemux86'
 or
You can also run generated Telechips images on Telechips EVB Boards

Other commonly useful commands are:
 - 'devtool' and 'recipetool' handle common recipe tasks
 - 'bitbake-layers' handles common layer tasks
 - 'oe-pkgdata-util' handles common target package tasks
Warning: source-mirror not exist!!
         The build to be slower or fail because will be download source code from upstream before build.
```


Use ***devtool*** to download the **u-boot-tcc** source code to workspace.

You can use the ***BitBake*** **-e** option to find out the source path as follows:

```bash
$ devtool modify u-boot-tcc

$ bitbake u-boot-tcc -e | grep ^S=
S=”/home/TOPST/build/tcc8050-sub/workspace/sources/u-boot-tcc”

$ cd /home/TOPST/build/tcc8050-sub/workspace/sources/u-boot-tcc
$ ls
api   board  common     configs  doc      dts  examples  include  Kconfig  Licenses     Makefile  oe-local-files  README   test
arch  cmd    config.mk  disk     drivers  env  fs        Kbuild   lib      MAINTAINERS  net       post            scripts  tools
```


U-boot-tcc can use the **Cleanall** or **build** function by using the ***BitBake*** **-c** option. However, the function must be defined as the task in the recipe.

```bash
$ bitbake u-boot-tcc -c cleanall

$ $ bitbake u-boot-tcc -c build
```


The following is an example that describes how to modify the source by using Git and create a patch file.

```bash
$ vi arch/arm/dts/tcc8050-subcore-ivi-TOPST_sv0.1.dts

$ git status
On branch tcc8050_linux_ivi_TOPST
Your branch is up to date with 'origin/tcc8050_linux_ivi_TOPST'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   arch/arm/dts/ tcc8050-subcore-ivi-TOPST_sv0.1.dts

$ git commit -s -m <title of your commit>
```


1. Add patch file in the **u-boot-tcc**

```bash
$ cp 001-test-sub-dts.patch poky/meta-telechips/meta-subcore/recipes-bsp/u-boot/u-boot-tcc/tcc8050-sub/

$ ls poky/meta-telechips/meta-subcore/recipes-bsp/u-boot/u-boot-tcc/tcc8050-sub/
add-dlvds.cfg  add-dp.cfg

$ vi poky/meta-telechips/meta-subcore/recipes-bsp/u-boot/u-boot-tcc/u-boot-tcc.bbappend
SRC_URI += “file://001-test-sub-dts.patch”
```


1. Re-build and execute U-Boot.

If you use the ***BitBake*** **-c build** option, the U-Boot binary is automatically saved as a path for binary downloads after compilation.

```bash
$ bitbake u-boot-tcc -c cleansstate

$ bitbake u-boot-tcc -c build

$ ls ~/TOPST/build/tcc8050-sub/workspace/sources/u-boot-tcc/tcc8050-sub/ca53_bl3.rom
ca53_bl3.rom
```


### 1.1.2.3 Configure U-Boot

The following shows how to use the U-Boot **menuconfig**.

```bash
$ make tcc805x_subcore_defconfig

$ make menuconfig
```
 
 <p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/9d934f78-e28e-46a9-b8a0-e6f47cef637c" width="750" height="400" >
</p>

```bash
$ bitbake u-boot-tcc -c build

$ ls ~/TOPST/build/tcc8050-sub/workspace/sources/u-boot-tcc/tcc8050-sub/ca53_bl3.rom
ca53_bl3.rom

```


### 1.1.2.4 Device Tree for U-Boot

Configure the U-Boot device tree.

The following is information in the device tree file for developing the U-Boot device driver.

```bash
$ grep tcc8050-subcore-ivi-TOPST poky/meta-telechips/meta-subcore/conf/machine/tcc8050-sub.conf
UBOOT_DEVICE_TREE = "tcc8050-subcore-ivi-TOPST_sv0.1"
```


The following is the device tree file in the U-Boot source code of the sub-core.

```bash
$ cd ~/TOPST/build/tcc8050-sub/workspace/sources/u-boot-tcc/

$ cat arch/arm/dts/tcc8050-subcore-ivi-TOPST_sv0.1.dts

// SPDX-License-Identifier: (GPL-2.0-or-later OR MIT)
/*
 * Copyright (C) Telechips Inc.
 */

/dts-v1/;

#include "tcc805x.dtsi"
#include "tcc8050_53-pinctrl.dtsi"

/ {
	model = "Telechips TCC8050 IVI TOPST SV0.1 Subcore";
	interrupt-parent = <&gic1>;
};

&chosen {
	stdout-path = &uart4;
};

&config {
	u-boot,mmc-env-partition = "subcore_env";
	u-boot,scsi-env-partition = "subcore_env";
};

&memory {
	reg = <0x40000000 0x20000000>;
};

&cpus {
	cpu@4 {
		status = "okay";
	};
};

&gic1 {
	status = "okay";
};

&uart4 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart9_data>;
	status = "okay";
	u-boot,dm-pre-reloc;
};

&tcc_lcd_interface {
	board-type = <1>;
	status = "okay";
};

&mbox1 {
	status = "okay";
	u-boot,dm-pre-reloc;
};

&tcc_sc_fw {
	status = "okay";
	mboxes = <&mbox1 0>;
	u-boot,dm-pre-reloc;

	tcc_sc_mmc {
		status = "okay";
		u-boot,dm-pre-reloc;
	};
};

&i2c7 {
	pinctrl-names = "default";
	//pinctrl-0 = <&i2c12_bus>; 
	pinctrl-0 = <&i2c13_bus>; 
	//port-mux = <12>;
	port-mux = <13>;
	status = "okay";

	videosource0: videodecoder_adv7182 {
		compatible	= "analogdevices,adv7182";
		pinctrl-names	= "idle", "active";
		pinctrl-0	= <&cam0_idle>;
		pinctrl-1	= <&cam0_rst &cam0_clk &cam0_hsync &cam0_vsync
				   &cam0_fld &cam0_de &cam0_data>;
		rst-gpios	= <&gpb 14 GPIO_ACTIVE_HIGH>;
		reg		= <0x21>;	// 0x42 >> 1
		cifport		= <0>;
	};
#if 1
	videosource2: deserializer_max9276 {
		compatible	= "maxim,max9276";
		pinctrl-names	= "idle", "active";
		pinctrl-0	= <&cam0_idle>;
		pinctrl-1	= <&cam0_rst &cam0_clk &cam0_hsync &cam0_vsync
				   &cam0_fld &cam0_de &cam0_data>;
		rst-gpios	= <&gpb 14 GPIO_ACTIVE_HIGH>;
		reg		= <0x4A>;	// 0x94 >> 1
		cifport		= <0>;
	};
#else
	videosource2: deserializer_max9276 {
		compatible	= "maxim,max9276";
		pinctrl-names	= "idle", "active";
		pinctrl-0	= <&cam1_idle>;
		pinctrl-1	= <&cam1_rst &cam1_clk &cam1_hsync &cam1_vsync
				   &cam1_fld &cam1_de &cam1_data>;
		rst-gpios	= <&gpma 21 GPIO_ACTIVE_HIGH>;
		reg		= <0x4A>;	// 0x94 >> 1
		cifport		= <1>;
	};
#endif
};

&switch0 {
	status = "okay";

	pinctrl-names = "default";
	pinctrl-0 = <&switch_mb23>;

	switch-gpios = <&gpmb 23 1>;
	switch-active = <1>;
};

&tcc_timer0 {
	/* disable dummy timer on Cortex-A53 subcore */
	status = "disabled";
};

&tcc_timer1 {
	/* for earlycamera */
	status = "okay";
};
```


The following is information from the driver file developed for TOPST.

```bash
$ cd ~/TOPST/build/tcc8050-sub/workspace/sources/u-boot-tcc/

$ find drivers/ | grep tcc

drivers/mmc/tcc_sdhci.c
drivers/mmc/tcc_sc_mmc.c
drivers/spi/tcc_spi.c
drivers/power/domain/tcc-power-domain.c
drivers/watchdog/tcc_wdt.c
drivers/watchdog/tcc_pmu_wdt.c
drivers/camera/splash/splash_1920x720x32_tcc803xp.img
drivers/timer_irq/tcc_timer.c
drivers/adc/tcc_adc.c
drivers/clk/tcc_clk.c
drivers/clk/tcc_dm_clk.c
drivers/phy/phy-tcc-dwc2.c
drivers/phy/phy-tcc-ehci.c
drivers/phy/phy-tcc-dwc3.c
drivers/pwm/pwm-tcc.c
drivers/pwm/pwm-tcc-dm.c
drivers/net/tcc_gmac.c
drivers/video/tcc
drivers/video/tcc/lcd_SLVDS_SAMPLE.c
drivers/video/tcc/lcd_HDMIV20.h
drivers/video/tcc/tcc_lcd_interface.c
drivers/video/tcc/tcc_hdmi_v1_4.c
drivers/video/tcc/tcc_lvds_ctrl.c
drivers/video/tcc/lcd_HDMIV20.c
drivers/video/tcc/tcc_hdmi_v2_0.c
drivers/video/tcc/tcc_fb.c
drivers/video/tcc/Kconfig
drivers/video/tcc/lcd_TM123XDHP90.c
drivers/video/tcc/lcd_FLD0800.c
drivers/video/tcc/Makefile
drivers/video/tcc/tcc_hdmi_v2_0.h
drivers/video/tcc/lcd_DPV14.c
drivers/video/tcc/lcd_SLVDS_EXT.c
drivers/i2c/tcc_i2c.c
drivers/mailbox/tcc805x_multi_mbox.c
drivers/mailbox/tcc803x_multi_mbox.c
drivers/mailbox/tcc-mbox.c
drivers/ufs/tcc-ufs.h
drivers/ufs/tcc-ufs.c
drivers/ufs/tcc_sc_ufs.c
drivers/pinctrl/pinctrl-tcc.c
drivers/gpio/tcc_legacy_gpio.c
drivers/gpio/tcc_gpio.c
drivers/firmware/tcc_sc_fw.c
drivers/firmware/tcc_sc_fw.h
drivers/tav/include/soc/tcc803x
drivers/tav/include/soc/tcc803x/tav_rdma.h
drivers/tav/include/soc/tcc803x/tav_wdma.h
drivers/tav/include/soc/tcc803x/tav_fifo.h
drivers/tav/include/soc/tcc803x/tav_wmix.h
drivers/tav/include/soc/tcc803x/tav_scaler.h
drivers/tav/include/soc/tcc803x/tav_fdly.h
drivers/tav/include/soc/tcc805x
drivers/tav/include/soc/tcc805x/tav_rdma.h
drivers/tav/include/soc/tcc805x/tav_wdma.h
drivers/tav/include/soc/tcc805x/tav_fifo.h
drivers/tav/include/soc/tcc805x/tav_wmix.h
drivers/tav/include/soc/tcc805x/tav_scaler.h
drivers/tav/include/soc/tcc805x/tav_fdly.h
```


## 1.2 Linux Kernel Installation

### 1.2.1 Main Core (CA72)

### 1.2.1.1 Linux Kernel Recipe

### 1.2.1.1.1 How to locate Linux Kernel Recipe and Source in Yocto

**linux-telechips_5.4.bb** is a recipe created by adding functionality for TOPST to the Linux Kernel open source.

```bash
$ cd poky/meta-telechips-bsp/recipes-kernel/linux/

$ ls

$ kernel-devsrc.bbappend  linux-telechips  linux-telechips_5.4.bb  linux-telechips.inc
```


The following describes the features to be applied when building Linux Kernel in Yocto for the main core.

```bash
# cat linux-telechips.inc

inherit kernel
inherit kernel-yocto

LICENSE = "GPLv2"

DEPENDS += "tc-make-image-native"

S = "${WORKDIR}/git"

KBRANCH = "${ALS_BRANCH}"

KERNEL_CC_append = " --sysroot=${WORKDIR}/recipe-sysroot "

KERNEL_IMAGETYPE = "${@bb.utils.contains("TUNE_FEATURES", "aarch64", "Image", "zImage", d)}"
KERNEL_IMAGETYPE_UNCOMPRESSED = "Image"

KERNEL_IMAGE_BASE_NAME_UNCOMPRESSED = "${KERNEL_IMAGETYPE_UNCOMPRESSED}-${PKGE}-${PKGV}-${PKGR}-${MACHINE}-${DATETIME}"
KERNEL_IMAGE_BASE_NAME_UNCOMPRESSED[vardepsexclude] = "DATETIME"
KERNEL_IMAGE_SYMLINK_NAME_UNCOMPRESSED ?= "${KERNEL_IMAGETYPE_UNCOMPRESSED}-${MACHINE}"

SRCREV_machine = "${SRCREV}"

PV = "${LINUX_VERSION}"

RAMDISK_NAME="ramdisk_dummy.rom"

BOOT_IMAGE_SUFFIX ?= "img"

BOOT_IMAGE = "${MACHINE}-tc-boot-${PV}-${PR}.${BOOT_IMAGE_SUFFIX}"
BOOT_IMAGE_BINARY = "tc-boot.${BOOT_IMAGE_SUFFIX}"
BOOT_IMAGE_SYMLINK = "tc-boot-${MACHINE}.${BOOT_IMAGE_SUFFIX}"

BOOT_IMAGE_UNCOMPRESSED = "${MACHINE}-tc-boot_uncompressed-${PV}-${PR}.${BOOT_IMAGE_SUFFIX}"
BOOT_IMAGE_BINARY_UNCOMPRESSED = "tc-boot_uncompressed.${BOOT_IMAGE_SUFFIX}"
BOOT_IMAGE_SYMLINK_UNCOMPRESSED = "tc-boot-${MACHINE}_uncompressed.${BOOT_IMAGE_SUFFIX}"

INSANE_SKIP_${PN} += "installed-vs-shipped"
PATCHTOOL = "git"

do_tc_make_image() {
        cd ${D}/${KERNEL_IMAGEDEST}

        touch ${RAMDISK_NAME}
        #compressed boot image
        ${STAGING_BINDIR_NATIVE}/tc-make-bootimg --kernel ${KERNEL_IMAGETYPE}-${KERNEL_VERSION} --ramdisk ${RAMDISK_NAME} --base ${KERNEL_BASE_ADDR} --kernel_offset ${KERNEL_OFFSET} --output ${BOOT_IMAGE} --cmdline ${CMDLINE}
        #uncompressed boot image
        ${STAGING_BINDIR_NATIVE}/tc-make-bootimg --kernel ${KERNEL_IMAGETYPE_UNCOMPRESSED}-${KERNEL_VERSION} --ramdisk ${RAMDISK_NAME} --base ${KERNEL_BASE_ADDR} --kernel_offset ${KERNEL_OFFSET} --output ${BOOT_IMAGE_UNCOMPRESSED} --cmdline ${CMDLINE}
        rm  ${RAMDISK_NAME}

        cd -
}

do_change_defconfig() {
        echo "CONFIG_STAGING=y"                                                         >> ${WORKDIR}/defconfig
        echo "CONFIG_LINUX_ANDROID=y"                                           >> ${WORKDIR}/defconfig

        if ${@bb.utils.contains('INVITE_PLATFORM', 'dispman', 'true', 'false', d)}; then
                echo "CONFIG_TCC_DISPMG=y" >> ${WORKDIR}/defconfig
        fi

        if ${@bb.utils.contains('DISTRO_FEATURES', 'multimedia', 'true', 'false', d)}; then
                echo "CONFIG_TCC_VPU_DRV=m"                                     >> ${WORKDIR}/defconfig
        fi

    if ${@bb.utils.contains('INVITE_PLATFORM', 'use-vout-vsync', 'true', 'false', d)}; then
                echo "CONFIG_VOUT_USE_VSYNC_INT=y"                              >> ${WORKDIR}/defconfig
                echo "CONFIG_VOUT_DISPLAY_LASTFRAME=y"                  >> ${WORKDIR}/defconfig
    fi

        if ${@bb.utils.contains('DISTRO_FEATURES', 'nfs', 'true', 'false', d)}; then
                echo "CONFIG_NFSD=y"                                                    >> ${WORKDIR}/defconfig
                echo "CONFIG_NFSD_V3_ACL=y"                                     >> ${WORKDIR}/defconfig
                echo "CONFIG_NFSD_V4=y"                                                 >> ${WORKDIR}/defconfig
        fi

        if ${@bb.utils.contains('DISTRO_FEATURES', 'systemd', 'true', 'false', d)}; then
                echo "CONFIG_FHANDLE=y"                                                 >> ${WORKDIR}/defconfig
        fi

# network configuration for ssh server
        if ${@bb.utils.contains('INVITE_PLATFORM', 'network', 'true', 'false', d)}; then
                echo "CONFIG_INET=y"                                            >> ${WORKDIR}/defconfig
                echo "CONFIG_IPV6=y"                                            >> ${WORKDIR}/defconfig
                echo "CONFIG_NETDEVICES=y"                                      >> ${WORKDIR}/defconfig
                echo "CONFIG_ETHERNET=y"                                        >> ${WORKDIR}/defconfig
                echo "CONFIG_PHYLIB=y"                                          >> ${WORKDIR}/defconfig
                echo "CONFIG_TCC_REALTEK_PHY=y"                         >> ${WORKDIR}/defconfig
                echo "CONFIG_STMMAC_ETH=y"                                      >> ${WORKDIR}/defconfig

                if ${@bb.utils.contains('USE_USB_TO_ETHERNET', '1', 'true', 'false', d)}; then
                        echo "CONFIG_USB_USBNET=y"                              >>  ${WORKDIR}/defconfig
                fi

                if ${@bb.utils.contains('USE_RNDIS_HOST', '1', 'true', 'false', d)}; then
                        echo "CONFIG_USB_NET_RNDIS_HOST=y"              >> ${WORKDIR}/defconfig
                fi

                if ${@bb.utils.contains('USE_IP_NETFILTER', '1', 'true', 'false', d)}; then
                        echo "CONFIG_NETFILTER=y"                                       >>  ${WORKDIR}/defconfig
                        echo "CONFIG_NF_CONNTRACK=y"                            >>  ${WORKDIR}/defconfig
                        echo "CONFIG_NF_TABLES=y"                                       >>  ${WORKDIR}/defconfig
                        echo "CONFIG_NF_TABLES_NETDEV=y"                        >>  ${WORKDIR}/defconfig
                        echo "CONFIG_NFT_CT=y"                                          >>  ${WORKDIR}/defconfig
                        echo "CONFIG_NFT_MASQ=y"                                        >>  ${WORKDIR}/defconfig
                        echo "CONFIG_NFT_NAT=y"                                         >>  ${WORKDIR}/defconfig
                        echo "CONFIG_NF_CONNTRACK_IPV4=y"                       >>  ${WORKDIR}/defconfig
                        echo "CONFIG_IP_NF_IPTABLES=y"                          >>  ${WORKDIR}/defconfig
                        echo "CONFIG_IP_NF_FILTER=y"                            >>  ${WORKDIR}/defconfig
                        echo "CONFIG_IP_NF_NAT=y"                                       >>  ${WORKDIR}/defconfig
                        echo "CONFIG_IP_NF_TARGET_MASQUERADE=y"         >>  ${WORKDIR}/defconfig
                fi
        fi

        if ${@bb.utils.contains_any('INVITE_PLATFORM', 'with-subcore', 'true', 'false', d)}; then
                echo "CONFIG_MAILBOX=y"                                                 >> ${WORKDIR}/defconfig
                echo "CONFIG_TCC_MULTI_MAILBOX=y"                               >> ${WORKDIR}/defconfig
                echo "CONFIG_PROC_MBOX=y"                               >> ${WORKDIR}/defconfig
        fi

        if ${@bb.utils.contains('INVITE_PLATFORM', 'gpu-vz', 'true', 'false', d)}; then
                echo "CONFIG_POWERVR_VZ=y"                                              >> ${WORKDIR}/defconfig
        fi

# GPU/Graphic Configuration
    if ${@bb.utils.contains('INVITE_PLATFORM', 'drm', 'true', 'false', d)}; then
                echo "CONFIG_FB=n"                                                      >> ${WORKDIR}/defconfig

                echo "CONFIG_DRM_TCC=y"                     >> ${WORKDIR}/defconfig
                echo "CONFIG_DRM_TCC_LCD=y"                 >> ${WORKDIR}/defconfig
                echo "CONFIG_DRM_TCC_VIC_MAX=1026"                      >> ${WORKDIR}/defconfig
                echo "CONFIG_DRM_TCC_LCD_VIC=1024"                      >> ${WORKDIR}/defconfig
                echo "CONFIG_DRM_TCC_KEEP_LOGO=y"                               >> ${WORKDIR}/defconfig

        if ${@bb.utils.contains_any('INVITE_PLATFORM', 'hud-display', 'true', 'false', d)}; then
                        echo "CONFIG_DRM_TCC_EXT=y"                             >> ${WORKDIR}/defconfig
                        echo "CONFIG_DRM_TCC_EXT_VIC=1024"              >> ${WORKDIR}/defconfig
                fi

                echo "CONFIG_DRM_TCC_DPI=y"                     >> ${WORKDIR}/defconfig

                if ${@bb.utils.contains_any('TCC_MACHINE_FAMILY', 'tcc805x-main tcc805x-cluster', 'true', 'false', d)}; then
                        echo "CONFIG_DRM_PANEL_LVDS_TCC=y"                      >> ${WORKDIR}/defconfig
                        echo "CONFIG_DRM_PANEL_DPV14_TCC=y"                     >> ${WORKDIR}/defconfig
                        echo "CONFIG_DRM_PANEL_MAX968XX=y"                      >> ${WORKDIR}/defconfig
                fi
        else
                echo "CONFIG_DRM_TCC=n"                                         >> ${WORKDIR}/defconfig

                echo "CONFIG_FB=y"                                                      >> ${WORKDIR}/defconfig
                echo "CONFIG_FB_NEW=y"                                          >> ${WORKDIR}/defconfig
                echo "CONFIG_FB_PANEL_LVDS_TCC=y"                       >> ${WORKDIR}/defconfig

                if ${@bb.utils.contains_any('TCC_MACHINE_FAMILY', 'tcc805x-main tcc805x-cluster', 'true', 'false', d)}; then
                        echo "CONFIG_POWERVR_ROGUE=y"                           >> ${WORKDIR}/defconfig
                        echo "CONFIG_POWERVR_DC_FBDEV=y"                        >> ${WORKDIR}/defconfig
                fi
        fi

# for disable 4k
        if ${@bb.utils.contains('INVITE_PLATFORM', 'support-4k-video', 'false', 'true', d)}; then
                echo "# CONFIG_SUPPORT_TCC_HEVC_4K is not set"                          >> ${WORKDIR}/defconfig
                echo "# CONFIG_SUPPORT_TCC_VP9_4K is not set"                           >> ${WORKDIR}/defconfig
        fi

# for TOPST board
        echo "CONFIG_I2C_TCC_V3=y"                                      >> ${WORKDIR}/defconfig
        echo "CONFIG_REGULATOR_DA9121=y"                                >> ${WORKDIR}/defconfig
        echo "CONFIG_REGULATOR_DA9210=y"                                >> ${WORKDIR}/defconfig
        echo "CONFIG_PM_TCC805X_DA9131_SW_WORKAROUND=y"                         >> ${WORKDIR}/defconfig
        echo "CONFIG_REGULATOR_DA9062=y"                                >> ${WORKDIR}/defconfig
        echo "# CONFIG_TOUCHSCREEN_INIT_SERDES is not set"              >> ${WORKDIR}/defconfig

    if ${@bb.utils.contains('INVITE_PLATFORM', 'dp2hdmi', 'true', 'false', d)}; then
                # DP to HDMI (1920x1080)
                echo "CONFIG_DRM_TCC_LCD_VIC=16"                        >> ${WORKDIR}/defconfig
        else
                # DP mode
                echo "CONFIG_DRM_TCC_LCD_VIC=0"                         >> ${WORKDIR}/defconfig
        fi
}

do_change_defconfig_append_tcc803x() {
        if ${@bb.utils.contains('DISTRO_FEATURES', 'opengl', 'true', 'false', d)}; then
                echo "CONFIG_MALI_MIDGARD=y"                    >> ${WORKDIR}/defconfig
        fi
}

addtask change_defconfig before do_configure after do_kernel_metadata
addtask tc_make_image before do_deploy after do_install

do_configure_prepend() {
        cp ${WORKDIR}/defconfig ${B}/.config
}

kernel_do_install_append() {
        if ${@bb.utils.contains("TUNE_FEATURES", "aarch64", "false", "true", d)}; then
                install -m 0644 ${KERNEL_OUTPUT_DIR}/${KERNEL_IMAGETYPE_UNCOMPRESSED} ${D}/${KERNEL_IMAGEDEST}/${KERNEL_IMAGETYPE_UNCOMPRESSED}-${KERNEL_VERSION}
        fi
        rm -f $kerneldir/mkbootimg
}

do_deploy_append() {
        install -m 0644 ${D}/${KERNEL_IMAGEDEST}/${KERNEL_IMAGETYPE_UNCOMPRESSED}-${KERNEL_VERSION} ${DEPLOYDIR}/${KERNEL_IMAGE_BASE_NAME_UNCOMPRESSED}.bin
        install -m 0644 ${D}/${KERNEL_IMAGEDEST}/${BOOT_IMAGE} ${DEPLOYDIR}/
        install -m 0644 ${D}/${KERNEL_IMAGEDEST}/${BOOT_IMAGE_UNCOMPRESSED} ${DEPLOYDIR}/

        cd ${DEPLOYDIR}

        ln -sf ${KERNEL_IMAGE_BASE_NAME_UNCOMPRESSED}.bin ${DEPLOYDIR}/${KERNEL_IMAGE_SYMLINK_NAME_UNCOMPRESSED}.bin
        ln -sf ${KERNEL_IMAGE_BASE_NAME_UNCOMPRESSED}.bin ${DEPLOYDIR}/${KERNEL_IMAGETYPE_UNCOMPRESSED}

    rm -f ${BOOT_IMAGE_BINARY} ${BOOT_IMAGE_SYMLINK}
    ln -sf ${BOOT_IMAGE} ${BOOT_IMAGE_SYMLINK}
    ln -sf ${BOOT_IMAGE} ${BOOT_IMAGE_BINARY}

        rm -f ${BOOT_IMAGE_BINARY_UNCOMPRESSED} ${BOOT_IMAGE_SYMLINK_UNCOMPRESSED}
        ln -sf ${BOOT_IMAGE_UNCOMPRESSED} ${BOOT_IMAGE_SYMLINK_UNCOMPRESSED}
        ln -sf ${BOOT_IMAGE_UNCOMPRESSED} ${BOOT_IMAGE_BINARY_UNCOMPRESSED}

        cd -
}

PACKAGE_KERNEL_BOOT_IMAGE = " \
        /${KERNEL_IMAGEDEST}/${BOOT_IMAGE} \
        /${KERNEL_IMAGEDEST}/${BOOT_IMAGE_BINARY} \
"
PACKAGE_KERNEL_BOOT_IMAGE_UNCOMPRESSED = " \
        /${KERNEL_IMAGEDEST}/${BOOT_IMAGE_UNCOMPRESSED} \
        /${KERNEL_IMAGEDEST}/${BOOT_IMAGE_BINARY_UNCOMPRESSED} \
"
FILES_${KERNEL_PACKAGE_NAME}-image += " \
        /${KERNEL_IMAGEDEST}/${KERNEL_IMAGETYPE_UNCOMPRESSED}* \
        ${PACKAGE_KERNEL_BOOT_IMAGE} \
        ${PACKAGE_KERNEL_BOOT_IMAGE_UNCOMPRESSED} \
"

FILES_${KERNEL_PACKAGE_NAME}-modules += " \
        /lib/modules/${KERNEL_VERSION}/modules.builtin \
        /lib/modules/${KERNEL_VERSION}/modules.order \
"

RDEPENDS_kernel-modules += "${@bb.utils.contains('DISTRO_FEATURES', 'multimedia', 'kernel-modules-vpu', '', d)}"

# disable unneeded tasks
do_uboot_mkimage[noexec] = "1"

pkg_postinst_kernel-image_append () {
        update-alternatives --install /${KERNEL_IMAGEDEST}/${KERNEL_IMAGETYPE_UNCOMPRESSED} ${KERNEL_IMAGETYPE_UNCOMPRESSED} /${KERNEL_IMAGEDEST}/${KERNEL_IMAGETYPE_UNCOMPRESSED}-${KERNEL_VERSION} ${KERNEL_PRIORITY} || true
}
```

The following is the **linux-telechips_5.4.bb** recipe:

```bash
$ cat linux-telechips_5.4.bb

require linux-telechips.inc

SRC_URI = "${TELECHIPS_AUTOMOTIVE_BSP_GIT}/kernel-5.4.git;protocol=${ALS_GIT_PROTOCOL};branch=${ALS_BRANCH}"
SRCREV = "${AUTOREV}"

LIC_FILES_CHKSUM = "file://COPYING;md5=bbea815ee2795b2f4230826c0c6b8814"

LINUX_VERSION = "5.4.159"
COMPATIBLE_MACHINE = "(tcc803x|tcc805x)"
KERNEL_EXTRA_ARGS_arm_append = " ARCH=arm"
KERNEL_EXTRA_ARGS_aarch64_append = " ARCH=arm64"

KERNEL_OFFSET_arm = "0x8000"
KERNEL_OFFSET_aarch64 = "0x80000"

do_change_defconfig_append() {
        if ${@bb.utils.contains_any('INVITE_PLATFORM', 'with-subcore', 'true', 'false', d)}; then
                echo "CONFIG_CAMIPC=y"                                          >> ${WORKDIR}/defconfig
        fi
}

kernel_do_install_append() {
        if [ -e "${D}${KERNEL_SRC_PATH}/tools/gator/daemon/escape" ]; then
                rm ${D}${KERNEL_SRC_PATH}/tools/gator/daemon/escape
        fi

        rm -rf ${D}/lib/modules/${KERNEL_VERSION}/kernel/drivers/char/vpu/vpu_hevc_enc_lib*
        rm -rf ${D}/lib/modules/${KERNEL_VERSION}/kernel/drivers/char/vpu/vpu_lib*
        rm -rf ${D}/lib/modules/${KERNEL_VERSION}/kernel/drivers/char/vpu/vpu_4k_d2_lib*
        rm -rf ${D}/lib/modules/${KERNEL_VERSION}/kernel/drivers/char/vpu/jpu_lib*
}
```


### 1.2.1.2 Modify Linux Kernel Source

The following are the settings for using ***BitBake*** to modify Linux Kernel source code in Yocto for the main core.

```bash
$ source poky/oe-init-build-env build/tcc8050-main/

Yocto Project common targets are:
    core-image-minimal
    core-image-sato
    meta-toolchain
    adt-installer
    meta-ide-support

Telechips common targets are:
    telechips-ivi-image-minimal
    telechips-ivi-image-gstreamer(minimal + GStreamer)
    telechips-ivi-image-qt(minimal + Qt)
    telechips-ivi-image(minimal + GStreamer + Qt)

    meta-toolchain-telechips(Application Development Toolkit)
    meta-toolchain-telechips-ivi(Application Development Toolkit include GStreamer)
    meta-toolchain-telechips-qt5(Application Development Toolkit include GStreamer and Qt5)

Telechips Automotive Linux Platform targets are:
    automotive-linux-platform-image(telechips-ivi-image + demo applications)

You can also run generated qemu images with a command like 'runqemu qemux86'
 or
You can also run generated Telechips images on Telechips EVB Boards

Other commonly useful commands are:
 - 'devtool' and 'recipetool' handle common recipe tasks
 - 'bitbake-layers' handles common layer tasks
 - 'oe-pkgdata-util' handles common target package tasks
Warning: source-mirror not exist!!
         The build to be slowler or fail because will be download source code from upstream before build.
```


Use ***devtool*** to download the Linux Kernel source code to workspace.

You can use the ***BitBake*** **-e** option to find out the source path as follows:

```bash
$ devtool modify virtual/kernel

$ bitbake virtual/kernel -e | grep ^S=
S=”/home/TOPST/build/tcc8050-main/workspace/sources/linux-telechips”

$ cd /home/TOPST/build/tcc8050-main/workspace/sources/linux-telechips
$ ls
android                            build.config.gki.aarch64          certs          kernel                  poky
arch                               build.config.gki-debug.aarch64    COPYING        lib                     README
block                              build.config.gki-debug.x86_64     CREDITS        LICENSES                README.md
build.config.aarch64               build.config.gki_kasan            crypto         MAINTAINERS             samples
build.config.allmodconfig          build.config.gki_kasan.aarch64    Documentation  Makefile                scripts
build.config.allmodconfig.aarch64  build.config.gki_kasan.x86_64     drivers        mklinuxmtdimg_64bit.sh  security
build.config.allmodconfig.arm      build.config.gki_kprobes          fs             mklinuxmtdimg.sh        sound
build.config.allmodconfig.x86_64   build.config.gki_kprobes.aarch64  include        mm                      tcc805x_customized_gpu_configuration.sh
build.config.arm                   build.config.gki_kprobes.x86_64   init           net                     tools
build.config.common                build.config.gki.x86_64           ipc            oe-logs                 usr
build.config.db845c                build.config.hikey960             Kbuild         oe-workdir              virt
build.config.gki                   build.config.x86_64               Kconfig        OWNERS
```


The kernel can use the **Cleanall** or **build** function by using the ***BitBake*** **-c** option. However, the function must be defined as the task in the recipe.

```bash
$ bitbake virtual/kernel -c cleanall

$ bitbake virtual/kernel -c build

$ ls -al oe-workdir 
oe-workdir -> /home/TOPST /build/tcc8050-main/tmp/work/tcc8050_sub-telechips-linux/linux-telechips/5.4.159-r0
```


The following is an example that describes how to modify the source by using Git and create a patch file.

```bash
$ vi arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts

$ git status
On branch tcc8050_linux_ivi_TOPST
Your branch is up to date with 'origin/tcc8050_linux_ivi_TOPST'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified: arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts

$ git commit -s -m <title of your commit>
```


1. Add patch file in the Linux Kernel recipe.

```bash
$ cp 001-test-dts.patch poky/meta-telechips-bsp/recipes-kernel/linux/linux-telechips/

$ ls poky/meta-telechips-bsp/recipes-kernel/linux/linux-telechips/
0101-perf-Make-perf-able-to-build-with-latest-libbfd.patch 001-test-dts.patch

$ vi poky/meta-telechips-bsp/recipes-kernel/linux/linux-telechips_5.4.bb
SRC_URI += “file://001-test-dts.patch”
```


1. Re-build and execute Linux Kernel.

If you use the ***BitBake*** **-c build** option, the Linux Kernel binary is automatically saved as a path for binary downloads after compilation.

```bash
$ bitbake virtual/kernel -c cleansstate
$ bitbake virtual/kernel -c build
```


### 1.2.1.3 Configure Linux Kernel

The following shows how to use the Kernel **menuconfig**.

```bash
$ bitbake virtual/kernel -c menuconfig
```
<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/b7fd7468-02ee-435a-9c54-db1a85648faf" width="750" height="400" >
</p>

```bash
$ bitbake u-boot-tcc -c build
```


### 1.2.1.4 Device Tree for Linux Kernel

Configure the U-Boot device tree.

The following is information in the device tree file for developing the Linux Kernel device driver.

```bash
$ grep tcc8050-linux-ivi-TOPST poky/meta-telechips-bsp/conf/machine/tcc8050-main.conf
KERNEL_DEVICETREE ?= "tcc/tcc8050-linux-ivi-TOPST_sv0.1.dtb"
UPDATE_DTB_NAME = "tcc8050-linux-ivi-TOPST_sv0.1.dtb"
```


The following is the device tree file in the kernel source code of the main core.

```bash
$ cd ~/TOPST/build/tcc8050-main/workspace/sources/linux-telechips/
$ cat arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts

// TOPST BOARD
/*
 * Copyright (C) Telechips Inc.
 */

#ifndef DT_OVERLAY
/dts-v1/;
#endif

#include "tcc8050-linux-ivi.dtsi"

/* Definition for H/W LCD ports */
#define LCD_PORT1_PWR "gpmc-19"
#define LCD_PORT1_RST "gpmc-20"
#define LCD_PORT1_BLK "gph-6"

#define LCD_PORT2_PWR "gpc-8"
#define LCD_PORT2_RST "gpc-9"
#define LCD_PORT2_BLK "gph-7"

#define LCD_PORT3_PWR "gpmc-2"
#define LCD_PORT3_RST "gpmc-18"
#define LCD_PORT3_BLK "gpe-15"

#define LCD_PORT4_PWR "gpmb-25"
#define LCD_PORT4_RST "gpmb-19"
#define LCD_PORT4_BLK "gpmc-21"

#include "tcc8050-linux-ivi-display.dtsi"

/ {
	switch0:switch0 {
		compatible = "telechips,switch";
		status = "okay";

#if 0	/* for softswitch */
		pinctrl-names = "default";
		pinctrl-0 = <&switch_mb23>;

		switch-gpios = <&gpmb 23 1>;
		switch-active = <0>;
#endif
	};

	lcd_port1_bl: lcd_port1_bl {
                gpios = <&gph 6 0>;
        };

        lcd_port2_bl: lcd_port2_bl {
                gpios = <&gph 7 0>;
        };

        lcd_port3_bl: lcd_port3_bl {
                gpios = <&gpe 15 0>;
        };

        lcd_port4_bl: lcd_port4_bl {
                gpios = <&gpmc 21 0>;
        };

	vqmmc_emmc: vqmmc_emmc {
		compatible = "regulator-fixed";
		regulator-name = "vqmmc_emmc";
		regulator-min-microvolt = <1800000>;
		regulator-max-microvolt = <1800000>;
		regulator-always-on;
	};

	sdhc2_pwrseq: sdhc2_pwrseq {
		compatible = "mmc-pwrseq-simple";
		reset-gpios = <&gpsd1 8 GPIO_ACTIVE_LOW>;
	};

	vqmmc_sdio: vqmmc_sdio {
		compatible = "regulator-fixed";
		regulator-name = "vqmmc_sdio";
		regulator-min-microvolt = <1800000>;
		regulator-max-microvolt = <1800000>;
		regulator-always-on;
	};

	tcc_sc_mmc: tcc_sc_mmc {
		compatible = "telechips,tcc805x-sc-mmc";
		status = "okay";

		max-frequency = <200000000>;
		bus-width = <8>;

		sc-firmware = <&tcc_sc_fw>;

		no-sdio;
		no-sd;
		non-removable;
		keep-power-in-suspend;
		disable-wp;

		mmc-hs400-1_8v;
		vqmmc-supply = <&vqmmc_emmc>;
	};

	tcc_sc_ufs: tcc_sc_ufs {
		compatible = "telechips,tcc805x-sc-ufs";
		status = "disabled";
		sc-firmware = <&tcc_sc_fw>;
	};

	vbus_supply_dwc2: vbus_supply_dwc2 {
		gpios = <&gpmc 6 0x0>;
	};

	vbus_supply_dwc3: vbus_supply_dwc3 {
		gpios = <&gpmc 8 0x0>;
	};

	vbus_supply_enable: vbus_supply_enable {
		compatible = "regulator-fixed";
		regulator-name = "vbus_supply_enable";
		regulator-min-microvolt = <5000000>;
		regulator-max-microvolt = <5000000>;
		gpios = <&gpmc 5 0x0>;
		enable-active-high;
		regulator-always-on;
	}; 

	aux_detect {
		status = "disabled";

		aux-gpios = <&gpmb 13 1>;
	};

	tcc_gpio_sample {
		compatible = "gpio_sample";
		label = "TCC GPIO";
		interrupt-parent = <&gpa>;
		interrupts = <1 IRQ_TYPE_EDGE_BOTH>;
		gpios = <&gpa 2 0>;
	};

	fb0: fb@0 {
		status = "disabled";
	};

	/* Panel for Framebuffer driver */
	fb_tm123xdhp90:fb_tm123xdhp90 {
		status = "disabled";
		pinctrl-0 = <&lcd_port_default3>;
		pinctrl-1 = <&lcd_port_power_on3>;
		pinctrl-2 = <&lcd_port_reset_off3>;
		pinctrl-3 = <&lcd_port_blk_on3>;
		pinctrl-4 = <&lcd_port_blk_off3>;
		pinctrl-5 = <&lcd_port_reset_on3 &lcd_port_power_off3>;
		backlight = <&lcd_port3_bl>;
	};
};

&a72_sc_mbox {
	status = "okay";
};

&tcc_sc_fw {
	status = "okay";
	mboxes = <&a72_sc_mbox 0>;
};

&i2c0 {
	status = "disabled";
	port-mux = <22>;
	pinctrl-names = "default";
	pinctrl-0 = <&i2c22_bus>;

	/* pmic da9131-D0 */
	da9131_D0: pmic@68 {
		compatible = "dlg,da9131";
		reg = <0x68>;

		regulators {
			DA9131_D0_BUCK1: buck1 {
				regulator-name = "core_0p8";
				regulator-always-on;
				regulator-boot-on;
			};

			DA9131_D0_BUCK2: buck2 {
				regulator-name = "gb_0p8";
				regulator-always-on;
				regulator-boot-on;
			};
		};
	}; /* pmic da9131-D0 */

	/* pmic da9131-D2 */
	da9131_D2: pmic@69 {
		compatible = "dlg,da9131";
		reg = <0x69>;

		regulators {
			DA9131_D2_BUCK1: buck1 {
				regulator-name = "cpu_1p0";
				regulator-always-on;
				regulator-boot-on;
			};

			DA9131_D2_BUCK2: buck2 {
				regulator-name = "io_3p3";
				regulator-always-on;
				regulator-boot-on;
			};
		};
	}; /* pmic da9131-D2 */

	/* pmic da9062 */
	da9062: pmic@58 {
		compatible = "dlg,da9062";
		reg = <0x58>;
		rst-gpios = <&gpma 12 0>;

		da9062_gpio: gpio {
			compatible = "dlg,da9062-gpio";
			gpio-controller;
			#gpio-cells = <2>;
		};

		regulators {
			DA9062_BUCK1: buck1 {
				regulator-name = "memq_1p1";
				regulator-always-on;
				regulator-boot-on;
			};

			DA9062_BUCK2: buck2 {
				regulator-name = "cpu_0p9v";
				regulator-always-on;
				regulator-boot-on;
			};

			DA9062_BUCK3: buck3 {
				regulator-name = "io_1p8";
				regulator-always-on;
				regulator-boot-on;
			};

			DA9062_BUCK4: buck4 {
				regulator-name = "memq_0p6";
				regulator-always-on;
				regulator-boot-on;
			};
#if 0
			/* Temp block for LDO2. Block min-microvolt of LDO2 */
			DA9062_LDO2: ldo2 {
				regulator-name = "mem_1p8";
				regulator-always-on;
				regulator-boot-on;
			};
#endif
			DA9062_LDO3: ldo3 {
				regulator-name = "mipi_1p2";
				regulator-always-on;
				regulator-boot-on;
			};

			DA9062_LDO4: ldo4 {
				regulator-name = "sdio_pwr";
				regulator-always-on;
				regulator-boot-on;
				regulator-min-microvolt = <1800000>;
				regulator-max-microvolt = <3300000>;
			};
		};
	}; /* pmic da9062 */
};

 /* eMMC */
&sdhc0 {
	bus-width = <8>;
	pinctrl-names = "default";
	pinctrl-0 = <&sd0_clk &sd0_cmd &sd0_bus8 &sd0_strb>;
	status = "disabled";

	sdhci-caps-mask = <0x7 0x0>;

	non-removable;
	keep-power-in-suspend;
	disable-wp;
	cap-mmc-hw-reset;

	mmc-hs400-1_8v;
	mmc-hs400-enhanced-strobe;
	tcc-mmc-taps = <0xF 0xF 0xF 0xF>;
	tcc-mmc-hs400-pos-tap = <0x6>;
	tcc-mmc-hs400-neg-tap = <0xB>;

	tcc-mmc-reset = <&gpsd0 14 0>;

	vqmmc-supply = <&vqmmc_emmc>;
};

/* SDIO  */
&sdhc1 {
	status = "disabled";
	pinctrl-names = "default";
	pinctrl-0 = <&sd1_clk &sd1_cmd &sd1_bus4 &out0_clk>;

	keep-power-in-suspend;
	cap-sdio-irq;

	vqmmc-supply = <&vqmmc_sdio>;

	cd-force-presence-change;

	tcm38xx {
		compatible = "telechips,tcm38xx";
		wifi_pwr-gpio = <&gpmd 10 0>; /* BT_WIFI_VBAT_EN */
		wlreg_on-gpio = <&gpmd 11 0>; /* WIFI_REG_ON */
		//wifi_pwr_1p8v-gpio = <&gpsd1 6 0>;
		//wifi_pwr_3p3v-gpio = <&gpsd1 7 0>;
	};
};

&sd1_clk {
	status = "disabled";
	telechips,pull-up;
};

&sd1_cmd {
	status = "disabled";
	telechips,pull-up;
};

&sd1_bus4 {
	status = "disabled";
	telechips,pull-up;
};

/* SD slot */

&sdhc2 {
	/*
	 * In order to reduce boot time,
	 * we use the driver as module not built-in.
	 */

	compatible = "telechips,tcc805x-sdhci,module-only";
	status = "disabled";
	pinctrl-names = "default";
	pinctrl-0 = <&sd2_clk &sd2_cmd &sd2_bus4 &sd1_bus4_wp_cd &sd1_pwr>;

	broken-cd;
	cd-inverted;
	keep-power-in-suspend;

	wp-gpios = <&gpsd1 7 0>;
	cd-gpios = <&gpsd1 6 0>;

	vqmmc-supply = <&DA9062_LDO4>;
	mmc-pwrseq = <&sdhc2_pwrseq>;
};

&sd2_clk {
	status = "disabled";
	telechips,no-pull;
};

&sd2_cmd {
	status = "disabled";
	telechips,no-pull;
};

&sd2_bus4 {
	status = "disabled";
	telechips,no-pull;
};

&i2c1 {
	status = "disabled";
	port-mux = <10>; /* [0]SCL [1]SDA */
	pinctrl-names = "default";
	pinctrl-0 = <&i2c10_bus>;
};

&i2c2 {
	status = "disabled";
	port-mux = <30>; /* [0]SCL [1]SDA */
	pinctrl-names = "default";
	pinctrl-0 = <&i2c30_bus>;
	ak4601: ak4601@10 {
		compatible = "akm,ak4601";
		ak4601,pdn-gpio = <&gpmb 29 0>;
		//cmute-gpios = <&gpmc 5 1>;     //CODEC_MUTE    0: active high, 1: active low
		//amute-gpios = <&gpma 1 1>;       //AMP_MUTE      0: active high, 1: active low
		//stanby-gpios = <&gpma 2 0>;     //AMP_STBY      0: active high, 1: active low
		reg = <0x10>;
	};
};

&i2c3 {
	status = "disabled";
	port-mux = <11>;
	pinctrl-names = "default";
	pinctrl-0 = <&i2c11_bus>;

	/* CarPlay require ack timeout count: 500 */
	ack-timeout = <500>;
};

&i2c4 {
	status = "disabled";
	port-mux = <6>;
	pinctrl-names = "default";
	pinctrl-0 = <&i2c6_bus>;
	mxt_touch@4b {
		compatible = "atmel,maxtouch";
		status = "okay";
		reg = <0x4b>;
		pinctrl-names = "default";
		pinctrl-0 = <&tsc0_default>;

		interrupt-parent = <&gpsd1>;
		interrupts = <9 IRQ_TYPE_EDGE_FALLING>;
		interrupt-controller;
		reset-gpio = <&gpb 16 0>;
	};
};

&i2c5 {
	status = "disabled";
	port-mux = <33>;
	pinctrl-names = "default";
	pinctrl-0 = <&i2c33_bus>;

	mxt_touch@4d {
		compatible = "atmel,maxtouch";
		status = "disabled";
		reg = <0x4d>;
		pinctrl-names = "default";
		pinctrl-0 = <&tsc1_sv01>;

		interrupt-parent = <&gpa>;
		interrupts = <21 IRQ_TYPE_EDGE_FALLING>;
		reset-gpio = <&gpmd 8 0>;
	};

	mxt_touch@4f {
		compatible = "atmel,maxtouch";
		status = "disabled";
		reg = <0x4f>;
		pinctrl-names = "default";
		pinctrl-0 = <&tsc2_sv01>;

		interrupt-parent = <&gpc>;
		interrupts = <13 IRQ_TYPE_EDGE_FALLING>;
		reset-gpio = <&gpc 12 0>;
	};
};

&i2c6 {
	status = "disabled";
	port-mux = <37>;
	pinctrl-names = "default";
	pinctrl-0 = <&i2c37_bus>;

	dp_serializer:max96851@60 {
		compatible	= "maxim,serdes";
		reg		= <0x60>;
	};

	dp_deserializer0:max96878@48 {
		compatible	= "maxim,serdes";
		reg		= <0x48>;
	};

	dp_deserializer1:max96878@4a {
		compatible	= "maxim,serdes";
		reg		= <0x4a>;
	};

	dp_deserializer2:max96878@4c {
		compatible	= "maxim,serdes";
		reg		= <0x4c>;
	};

	dp_deserializer3:max96878@68 {
		compatible	= "maxim,serdes";
		reg		= <0x68>;
	};
};

&max968xx_config {
	compatible = "telechips,max968xx_configuration";
	max968xx_evb_type = <2>;/* 0: TCC8059 EVB, 1: TCC8050/3 sv0.1, 2:TCC8050/3 sv1.0 */
	max96851_lane_02_13_swap = <1>;

	pinctrl-names = "default", "default";
	pinctrl-0 = <&serdes_intb_sv10>;
	pinctrl-1 = <&serdes_lock>;
	status = "disabled";
};

&aux_detect_pin {
	telechips,pins = "gpmb-13";
	status = "disabled";
};

/* bluetooth */
&tcc_bluetooth {
	compatible = "telechips, tcc_bluetooth";
	bt_power-gpio = <&gpmd 10 0>; /* BTWIFI_VBAT_EN */
	bt_reg_on-gpio = <&gpmd 9 0>;
	bt_hwake-gpio = <&gpk 2 0>;
	status = "disabled";
};
```

The following is information from the driver file developed for TOPST.

```bash
$ cd ~/TOPST/build/tcc8050-main/workspace/sources/linux-telechips/

$ find drivers/ | grep tcc

drivers/gpu/drm/tcc
drivers/gpu/drm/tcc/tcc_drm_edid.h
drivers/gpu/drm/tcc/tcc_drm_screen_share.c
drivers/gpu/drm/tcc/vioc
drivers/gpu/drm/tcc/tcc_drm_fb.h
drivers/gpu/drm/tcc/tcc_drm_lcd_ext.c
drivers/gpu/drm/tcc/tcc_drm_drv.c
drivers/gpu/drm/tcc/tcc_drm_lcd_third.c
drivers/gpu/drm/tcc/tcc_drm_address.h
drivers/gpu/drm/tcc/tcc_drm_dpi.c
drivers/gpu/drm/tcc/tcc_drm_address.c
drivers/gpu/drm/tcc/tcc_drm_core.c
drivers/gpu/drm/tcc/tcc_drm_plane.h
drivers/gpu/drm/tcc/tcc_drm_gem.c
drivers/gpu/drm/tcc/tcc_drm_gem.h
drivers/gpu/drm/tcc/tcc_drm_lcd_fourth.c
drivers/gpu/drm/tcc/tcc_drm_lcd_base.c
drivers/gpu/drm/tcc/tcc_drm_drv.h
drivers/gpu/drm/tcc/tcc_drm_lcd_base.h
drivers/gpu/drm/tcc/Kconfig
drivers/gpu/drm/tcc/tcc_drm_fbdev.h
drivers/gpu/drm/tcc/tcc_drm_plane.c
drivers/gpu/drm/tcc/tcc_drm_edid.c
drivers/gpu/drm/tcc/tcc_drm_dpi.h
drivers/gpu/drm/tcc/Makefile
drivers/gpu/drm/tcc/tcc_drm_lcd_primary.c
drivers/gpu/drm/tcc/tcc_drm_fb.c
drivers/gpu/drm/tcc/tcc_drm_crtc.c
drivers/gpu/drm/tcc/tcc_drm_crtc.h
drivers/gpu/drm/tcc/tcc_drm_fbdev.c
drivers/gpu/drm/panel/panel-tcc.h
drivers/gpu/drm/panel/panel-tcc-dpv14.h
drivers/gpu/drm/panel/panel-tcc-dpv14.c
drivers/gpu/drm/panel/panel-lvds-tcc.c
drivers/gpu/imgtec/tcc_9xtp
drivers/gpu/imgtec/tcc_9xtp/sysconfig.h
drivers/gpu/imgtec/tcc_9xtp/sysconfig.c
drivers/gpu/imgtec/tcc_9xtp/tcc_9xtp_init.h
drivers/gpu/imgtec/tcc_9xtp/sysinfo.h
drivers/gpu/imgtec/tcc_9xtp/tcc_9xtp_init.c
drivers/gpu/imgtec/tcc_9xtp/Kconfig
drivers/gpu/imgtec/tcc_9xtp/Makefile
drivers/gpu/imgtec/pvrsrvkm/dc_fbdev_tcc.c
drivers/gpu/arm/utgard/mali/platform/tcc-m400
drivers/gpu/arm/utgard/mali/platform/tcc-m400/arm_core_scaling.c
drivers/gpu/arm/utgard/mali/platform/tcc-m400/mali_platform_dvfs.c
drivers/gpu/arm/utgard/mali/platform/tcc-m400/mali_platform.c
drivers/gpu/arm/utgard/mali/platform/tcc-m400/arm_core_scaling.h
drivers/gpu/arm/midgard/platform/tcc-mali-g51
drivers/gpu/arm/midgard/platform/tcc-mali-g51/mali_kbase_config_tcc.c
drivers/gpu/arm/midgard/platform/tcc-mali-g51/mali_kbase_config_platform.h
drivers/gpu/arm/midgard/platform/tcc-mali-g51/Kbuild
drivers/dma/tcc-dma.c
drivers/mmc/host/sdhci-tcc-mod.c
drivers/mmc/host/sdhci-tcc.h
drivers/mmc/host/sdhci-tcc.c
drivers/mmc/host/tcc_sc_mmc.c
drivers/spi/spi-tcc.c
drivers/spi/spi-tcc.h
drivers/spi/tcc_gpsb_tsif.c
drivers/tty/serial/tcc_serial.h
drivers/tty/serial/tcc_serial.c
drivers/power/reset/tcc-reboot-mode.c
drivers/watchdog/tcc805x_cbus_wdt.c
drivers/watchdog/tcc_pmu_wdt.c
drivers/watchdog/tcc803x_cbus_wdt.c
drivers/char/tcc_lut_3d_d1.c
drivers/char/tcc_lut.c
drivers/char/tcc_wdma.c
drivers/char/tcc_dxb_ctrl
drivers/char/tcc_dxb_ctrl/dxb_ctrl_gpio.h
drivers/char/tcc_dxb_ctrl/tcc_dxb_control.h
drivers/char/tcc_dxb_ctrl/amfmtuner.c
drivers/char/tcc_dxb_ctrl/dxb_ctrl_defs.h
drivers/char/tcc_dxb_ctrl/Kconfig
drivers/char/tcc_dxb_ctrl/dxb_ctrl_gpio.c
drivers/char/tcc_dxb_ctrl/Makefile
drivers/char/tcc_dxb_ctrl/amfmtuner.h
drivers/char/tcc_dxb_ctrl/dxb_ctrl.c
drivers/char/tcc_lut_3d_d0.c
drivers/char/tcc_aux_detect
drivers/char/tcc_aux_detect/aux_detect.c
drivers/char/tcc_aux_detect/Kconfig
drivers/char/tcc_aux_detect/Makefile
drivers/char/tcc_bluetooth.c
drivers/char/tcc_ipc
drivers/char/tcc_ipc/tcc_ipc_ctl.h
drivers/char/tcc_ipc/tcc_ipc_ctl.c
drivers/char/tcc_ipc/tcc_ipc_cmd.c
drivers/char/tcc_ipc/tcc_ipc_cmd.h
drivers/char/tcc_ipc/tcc_ipc_mbox.h
drivers/char/tcc_ipc/Kconfig
drivers/char/tcc_ipc/tcc_ipc_typedef.h
drivers/char/tcc_ipc/tcc_ipc_buffer.h
drivers/char/tcc_ipc/Makefile
drivers/char/tcc_ipc/tcc_ipc_os.h
drivers/char/tcc_ipc/tcc_ipc_mbox.c
drivers/char/tcc_ipc/tcc_ipc_buffer.c
drivers/char/tcc_ipc/tcc_ipc.c
drivers/char/tcc_ipc/tcc_ipc_os.c
drivers/char/tcc_lut.h
drivers/char/tcc_screen_share.h
drivers/char/hw_random/tcc-rng.h
drivers/char/hw_random/tcc-rng.c
drivers/char/tcc_thsm
drivers/char/tcc_thsm/tcc_thsm_cmd.c
drivers/char/tcc_thsm/Kconfig
drivers/char/tcc_thsm/tcc_thsm.c
drivers/char/tcc_thsm/tcc_thsm_log.h
drivers/char/tcc_thsm/Makefile
drivers/char/tcc_thsm/tcc_thsm_cmd.h
drivers/char/tcc_cp.c
drivers/char/tcc_ecid.c
drivers/char/tcc_screen_share.c
drivers/char/tcc_snor_updater
drivers/char/tcc_snor_updater/tcc_snor_updater_crc8.h
drivers/char/tcc_snor_updater/tcc_snor_updater.h
drivers/char/tcc_snor_updater/tcc_snor_updater_cmd.h
drivers/char/tcc_snor_updater/tcc_snor_updater_mbox.h
drivers/char/tcc_snor_updater/tcc_snor_updater_cmd.c
drivers/char/tcc_snor_updater/tcc_snor_updater_mbox.c
drivers/char/tcc_snor_updater/tcc_snor_updater_crc8.c
drivers/char/tcc_snor_updater/Kconfig
drivers/char/tcc_snor_updater/Makefile
drivers/char/tcc_snor_updater/tcc_snor_updater.c
drivers/char/tcc_snor_updater/tcc_snor_updater_dev.c
drivers/char/tcc_snor_updater/tcc_snor_updater_typedef.h
drivers/char/tcc_hsm
drivers/char/tcc_hsm/tcc803x
drivers/char/tcc_hsm/tcc803x/tcc_hsm_log.h
drivers/char/tcc_hsm/tcc803x/tcc_hsm_sp_cmd.c
drivers/char/tcc_hsm/tcc803x/tcc_hsm.c
drivers/char/tcc_hsm/tcc803x/tcc_hsm_sp_cmd.h
drivers/char/tcc_hsm/tcc805x
drivers/char/tcc_hsm/tcc805x/tcc_hsm.h
drivers/char/tcc_hsm/tcc805x/tcc_hsm_log.h
drivers/char/tcc_hsm/tcc805x/tcc_hsm_cmd.c
drivers/char/tcc_hsm/tcc805x/tcc_hsm_cmd.h
drivers/char/tcc_hsm/tcc805x/tcc_hsm.c
drivers/char/tcc_hsm/Kconfig
drivers/char/tcc_hsm/Makefile
drivers/reset/reset-tcc.c
drivers/thermal/tcc
drivers/thermal/tcc/tccxxxx_thermal.c
drivers/thermal/tcc/tcc_thermal.h
drivers/thermal/tcc/tcc805x_thermal.c
drivers/thermal/tcc/thermal_common.c
drivers/thermal/tcc/thermal_common.h
drivers/thermal/tcc/Kconfig
drivers/thermal/tcc/tcc_thermal.c
drivers/thermal/tcc/Makefile
drivers/media/platform/tcc-mipi-csi2
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-csi2.h
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-cfg-reg.h
drivers/media/platform/tcc-mipi-csi2/Kconfig
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-csi2-reg.h
drivers/media/platform/tcc-mipi-csi2/Makefile
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-csi2.c
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-ckc-reg.h
drivers/media/platform/tccvin2
drivers/media/platform/tccvin2/basic_operation.h
drivers/media/platform/tccvin2/tccvin_diagnostics.h
drivers/media/platform/tccvin2/tccvin_driver.c
drivers/media/platform/tccvin2/tccvin_v4l2.c
drivers/media/platform/tccvin2/tccvin_diagnostics.c
drivers/media/platform/tccvin2/tccvin_video.h
drivers/media/platform/tccvin2/Kconfig
drivers/media/platform/tccvin2/Makefile
drivers/media/platform/tccvin2/tccvin_subdev.c
drivers/media/platform/tccvin2/tccvin_video.c
drivers/media/platform/tccvout
drivers/media/platform/tccvout/tcc_vout_attr.h
drivers/media/platform/tccvout/tcc_vout.h
drivers/media/platform/tccvout/tcc_vout_core.c
drivers/media/platform/tccvout/tcc_vout_v4l2.c
drivers/media/platform/tccvout/tcc_vout_core.h
drivers/media/platform/tccvout/tcc_vout_dbg.h
drivers/media/platform/tccvout/Kconfig
drivers/media/platform/tccvout/Makefile
drivers/media/platform/tccvout/tcc_vout_dbg.c
drivers/media/platform/tccvout/tcc_vout_attr.c
drivers/media/platform/tcc-isp
drivers/media/platform/tcc-isp/tcc-isp.h
drivers/media/platform/tcc-isp/tcc-isp-settings.h
drivers/media/platform/tcc-isp/Kconfig
drivers/media/platform/tcc-isp/Makefile
drivers/media/platform/tcc-isp/tcc-isp-reg.h
drivers/media/platform/tcc-isp/tcc-isp-iommu.c
drivers/media/platform/tcc-isp/tcc-isp.c
drivers/rtc/tcc_rtc.c
drivers/rtc/tcc
drivers/rtc/tcc/tca_alarm.h
drivers/rtc/tcc/tca_rtc.c
drivers/rtc/tcc/tca_alarm.c
drivers/rtc/tcc/Makefile
drivers/rtc/tcc/tca_rtc.h
drivers/misc/tcc
drivers/misc/tcc/gpio-tcc-sample.c
drivers/misc/tcc/scaler_drv.c
drivers/misc/tcc/wmixer_drv.c
drivers/misc/tcc/tcc_grp.c
drivers/misc/tcc/tcc_shared_mem.c
drivers/misc/tcc/Kconfig
drivers/misc/tcc/tcc_grp.h
drivers/misc/tcc/Makefile
drivers/misc/tcc/tcc_sdr
drivers/misc/tcc/tcc_sdr/tcc_sdr_hw.h
drivers/misc/tcc/tcc_sdr/tcc_sdr.h
drivers/misc/tcc/tcc_sdr/tcc_sdr_dai.h
drivers/misc/tcc/tcc_sdr/tcc_sdr.c
drivers/misc/tcc/tcc_sdr/tcc_sdr_adma.h
drivers/misc/tcc/tccmisc_drv.c
drivers/clk/tcc
drivers/clk/tcc/clk-tcc803x-a7s.c
drivers/clk/tcc/clk-tcc897x.h
drivers/clk/tcc/clk-tcc803x-a7s-reg.h
drivers/clk/tcc/clk.c
drivers/clk/tcc/clk-tcc803x.h
drivers/clk/tcc/clk-tcc897x.c
drivers/clk/tcc/Makefile
drivers/clk/tcc/clk-tcc803x.c
drivers/clk/tcc/clk-tcc805x.h
drivers/clk/tcc/clk-tcc805x.c
drivers/pwm/pwm-tcc.c
drivers/pci/controller/dwc/pci-tcc.c
drivers/net/ethernet/tcc_virt_eth.c
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-ecid.c
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-debugfs.c
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-ecid.h
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-v2.h
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-v2.c
drivers/net/phy/tcc_realtek.c
drivers/net/phy/tcc_marvell.c
drivers/scsi/ufs/tcc_sc_ufs.c
drivers/video/fbdev/tcc-fb
drivers/video/fbdev/tcc-fb/vioc
drivers/video/fbdev/tcc-fb/vioc/vioc_lvds.c
drivers/video/fbdev/tcc-fb/vioc/vioc_lvds_t.c
drivers/video/fbdev/tcc-fb/vioc/vioc_fifo.c
drivers/video/fbdev/tcc-fb/vioc/vioc_scaler.c
drivers/video/fbdev/tcc-fb/vioc/vioc_lut_3d.c
drivers/video/fbdev/tcc-fb/vioc/vioc_timer.c
drivers/video/fbdev/tcc-fb/vioc/vioc_pxdemux.c
drivers/video/fbdev/tcc-fb/vioc/vioc_wdma.c
drivers/video/fbdev/tcc-fb/vioc/vioc_vin.c
drivers/video/fbdev/tcc-fb/vioc/vioc_pvric_fbdc.c
drivers/video/fbdev/tcc-fb/vioc/vioc_mc.c
drivers/video/fbdev/tcc-fb/vioc/vioc_intr.c
drivers/video/fbdev/tcc-fb/vioc/vioc_lut.c
drivers/video/fbdev/tcc-fb/vioc/vioc_outcfg.c
drivers/video/fbdev/tcc-fb/vioc/vioc_rdma.c
drivers/video/fbdev/tcc-fb/vioc/vioc_config.c
drivers/video/fbdev/tcc-fb/vioc/tca_map_converter.c
drivers/video/fbdev/tcc-fb/vioc/vioc_wmix.c
drivers/video/fbdev/tcc-fb/vioc/Kconfig
drivers/video/fbdev/tcc-fb/vioc/vioc_afbcdec.c
drivers/video/fbdev/tcc-fb/vioc/vioc_deintls.c
drivers/video/fbdev/tcc-fb/vioc/Makefile
drivers/video/fbdev/tcc-fb/vioc/vioc_ddicfg.c
drivers/video/fbdev/tcc-fb/vioc/vioc_disp.c
drivers/video/fbdev/tcc-fb/vioc/vioc_viqe.c
drivers/video/fbdev/tcc-fb/g2d
drivers/video/fbdev/tcc-fb/g2d/tca_gre2d_api.c
drivers/video/fbdev/tcc-fb/g2d/Kconfig
drivers/video/fbdev/tcc-fb/g2d/Makefile
drivers/video/fbdev/tcc-fb/g2d/tcc_gre2d.c
drivers/video/fbdev/tcc-fb/fb.c
drivers/video/fbdev/tcc-fb/tcc_overlay.c
drivers/video/fbdev/tcc-fb/viqe
drivers/video/fbdev/tcc-fb/viqe/tcc_vioc_viqe.c
drivers/video/fbdev/tcc-fb/viqe/tcc_vioc_viqe_interface.c
drivers/video/fbdev/tcc-fb/viqe/tcc_vioc_viqe.h
drivers/video/fbdev/tcc-fb/viqe/viqe.c
drivers/video/fbdev/tcc-fb/viqe/viqe.h
drivers/video/fbdev/tcc-fb/viqe/Makefile
drivers/video/fbdev/tcc-fb/viqe/tcc_vioc_viqe_interface.h
drivers/video/fbdev/tcc-fb/Kconfig
drivers/video/fbdev/tcc-fb/panel
drivers/video/fbdev/tcc-fb/panel/panel-tcc.h
drivers/video/fbdev/tcc-fb/panel/Kconfig
drivers/video/fbdev/tcc-fb/panel/panel_helper.c
drivers/video/fbdev/tcc-fb/panel/Makefile
drivers/video/fbdev/tcc-fb/panel/panel_helper.h
drivers/video/fbdev/tcc-fb/panel/panel-lvds-tcc.c
drivers/video/fbdev/tcc-fb/Makefile
drivers/i2c/busses/i2c-tcc-v3.c
drivers/i2c/busses/i2c-slave-tcc-chdrv.c
drivers/i2c/busses/i2c-slave-tcc-chdrv.h
drivers/i2c/busses/i2c-tcc.c
drivers/i2c/busses/i2c-tcc-v2.c
drivers/input/tc_touch_share/tcc_touch_receive.c
drivers/input/tc_touch_share/tcc_touch_cmd.c
drivers/input/tc_touch_share/tcc_touch_bridge.c
drivers/input/tc_touch_share/tcc_touch_cmd.h
drivers/input/keyboard/tcc_keypad.c
drivers/input/touchscreen/serdes/tcc_tsc_serdes.c
drivers/mailbox/tcc_ipc_log.h
drivers/mailbox/tcc_sec_ipc.c
drivers/mailbox/tcc803x_multi_mailbox
drivers/mailbox/tcc803x_multi_mailbox/tcc_multi_mbox.c
drivers/mailbox/tcc803x_multi_mailbox/Makefile
drivers/mailbox/mailbox-tcc-sc.c
drivers/mailbox/tcc805x_multi_mailbox
drivers/mailbox/tcc805x_multi_mailbox/tcc_multi_mbox.c
drivers/mailbox/tcc805x_multi_mailbox/tcc_multi_mbox_test.c
drivers/mailbox/tcc805x_multi_mailbox/Makefile
drivers/usb/dwc3/dwc3-tcc.h
drivers/usb/dwc3/dwc3-tcc.c
drivers/usb/phy/phy-tcc-ehci.h
drivers/usb/phy/phy-tcc-ehci.c
drivers/usb/phy/phy-tcc-dwc_otg.c
drivers/usb/phy/phy-tcc-dwc_otg.h
drivers/usb/phy/phy-tcc-dwc3.h
drivers/usb/phy/phy-tcc-dwc3.c
drivers/usb/host/ehci-tcc.c
drivers/usb/host/ehci-tcc.h
drivers/usb/host/ohci-tcc.h
drivers/usb/host/ohci-tcc.c
drivers/pinctrl/tcc
drivers/pinctrl/tcc/Kconfig
drivers/pinctrl/tcc/Makefile
drivers/pinctrl/tcc/pinctrl-tcc.c
drivers/soc/tcc
drivers/soc/tcc/pm-tcc805x.c
drivers/soc/tcc/irq-pic.c
drivers/soc/tcc/timer.c
drivers/soc/tcc/chipinfo.c
drivers/soc/tcc/bootstage.c
drivers/soc/tcc/Kconfig
drivers/soc/tcc/Makefile
drivers/gpio/gpio-tcc.c
drivers/iio/adc/tcc_adc.h
drivers/iio/adc/tcc_adc.c
drivers/firmware/tcc_pmi.c
drivers/firmware/tcc_sc_fw.c
```


### 1.2.2 Sub-core (CA53)

### 1.2.2.1 Linux Kernel Recipe

### 1.2.2.1.1 How to locate Linux Kernel Recipe and Source in Yocto

**linux-telechips_5.4.bb** is a recipe created by adding functionality for TOPST to the Linux Kernel open source.

```bash
$ cd poky/meta-telechips/meta-subcore/recipes-kernel/linux
$ ls
$ linux-telechips_%.bbappend
```

The following describes the features to be applied when building Linux Kernel in Yocto for the sub-core.

```bash
# cat linux-telechips_%.bbappend

do_change_defconfig() {
	if ${@bb.utils.contains_any('SUBCORE_APPS', 'rvc svm', 'true', 'false', d)}; then
		echo "CONFIG_MEDIA_SUPPORT=y"							>> ${WORKDIR}/defconfig
		echo "CONFIG_MEDIA_CAMERA_SUPPORT=y"					>> ${WORKDIR}/defconfig
		echo "CONFIG_V4L_PLATFORM_DRIVERS=y"					>> ${WORKDIR}/defconfig

		echo "CONFIG_MEDIA_CONTROLLER=y"						>> ${WORKDIR}/defconfig
		echo "CONFIG_VIDEO_V4L2_SUBDEV_API=y"					>> ${WORKDIR}/defconfig
		echo "CONFIG_VIDEO_TCCVIN2=y"							>> ${WORKDIR}/defconfig
		echo "# CONFIG_MEDIA_SUBDRV_AUTOSELECT is not set"		>> ${WORKDIR}/defconfig
		#echo "CONFIG_VIDEO_ADV7182=y"							>> ${WORKDIR}/defconfig
		echo "CONFIG_VIDEO_ISL79988=y"							>> ${WORKDIR}/defconfig
		echo "CONFIG_VIDEO_ARXXXX=y"							>> ${WORKDIR}/defconfig
		echo "CONFIG_VIDEO_MAX96701=y"							>> ${WORKDIR}/defconfig
		echo "CONFIG_VIDEO_MAX9286=y"							>> ${WORKDIR}/defconfig

		echo "CONFIG_MAILBOX=y"									>> ${WORKDIR}/defconfig
		echo "CONFIG_TCC_MULTI_MAILBOX=y"						>> ${WORKDIR}/defconfig
		echo "# CONFIG_VIDEO_TCC_VOUT is not set"				>> ${WORKDIR}/defconfig
		echo "CONFIG_SWITCH_REVERSE=y"							>> ${WORKDIR}/defconfig
		echo "CONFIG_FB_TCC_OVERLAY=y"							>> ${WORKDIR}/defconfig

		if ${@bb.utils.contains_any('SUBCORE_APPS', 'rvc', 'true', 'false', d)}; then
			echo "CONFIG_OVERLAY_PGL=y"							>> ${WORKDIR}/defconfig
		fi
	fi
		
	# TOPST : Raspberry Camera
	echo "CONFIG_VIDEO_IMX219=y"							>> ${WORKDIR}/defconfig
	echo "CONFIG_VIDEOBUF2_VMALLOC=y"						>> ${WORKDIR}/defconfig

	if ${@bb.utils.contains('SUBCORE_APPS', 't-sound', 'true', 'false', d)}; then
		echo "CONFIG_SOUND=y"									>>  ${WORKDIR}/defconfig
		echo "CONFIG_SND=y"										>>  ${WORKDIR}/defconfig
		echo "CONFIG_SND_SOC=y"									>>  ${WORKDIR}/defconfig
		echo "CONFIG_SND_SOC_TCC=m"								>>  ${WORKDIR}/defconfig
		echo "CONFIG_SND_SOC_AK4601=m "							>>  ${WORKDIR}/defconfig
		echo "CONFIG_TCC_MULTI_MAILBOX_AUDIO=y"					>>  ${WORKDIR}/defconfig
	fi

# network configuration for ssh server
	if ${@bb.utils.contains('INVITE_PLATFORM', 'network', 'true', 'false', d)}; then
		echo "CONFIG_INET=y"									>> ${WORKDIR}/defconfig
		echo "CONFIG_IPV6=y"									>> ${WORKDIR}/defconfig
		echo "CONFIG_NETDEVICES=y"								>> ${WORKDIR}/defconfig
		echo "CONFIG_ETHERNET=y"								>> ${WORKDIR}/defconfig
		echo "CONFIG_PHYLIB=y"									>> ${WORKDIR}/defconfig
		echo "# CONFIG_WLAN is not set"							>> ${WORKDIR}/defconfig
		echo "# CONFIG_WIRELESS is not set"						>> ${WORKDIR}/defconfig
		echo "CONFIG_USB_USBNET=y"								>>  ${WORKDIR}/defconfig

		if ${@bb.utils.contains('USE_RNDIS_HOST', '1', 'true', 'false', d)}; then
			echo "CONFIG_USB_NET_RNDIS_HOST=y"					>> ${WORKDIR}/defconfig
		fi
	fi

	if ${@bb.utils.contains('INVITE_PLATFORM', 'gpu-vz', 'true', 'false', d)}; then
		echo "CONFIG_POWERVR_ROGUE=y"							>> ${WORKDIR}/defconfig
		echo "CONFIG_POWERVR_TCC9XTP=y"							>> ${WORKDIR}/defconfig
		echo "CONFIG_POWERVR_VZ=y"								>> ${WORKDIR}/defconfig

		if ${@bb.utils.contains('INVITE_PLATFORM', 'drm', 'true', 'false', d)}; then
			echo "CONFIG_DRM_TCC=y"                             >> ${WORKDIR}/defconfig
			echo "CONFIG_DRM_TCC_LCD=y"                         >> ${WORKDIR}/defconfig
		else
			echo "CONFIG_POWERVR_DC_FBDEV=y"                    >> ${WORKDIR}/defconfig
		fi
	fi

	if ${@bb.utils.contains_any('SUBCORE_APPS', 'rvc svm', 'true', 'false', d)}; then
		echo "CONFIG_CAMIPC=y"								>> ${WORKDIR}/defconfig
	fi
}

do_change_defconfig_append_tcc803x() {
	if ${@bb.utils.contains('SUBCORE_APPS', 'cluster', 'true', 'false', d)}; then
		echo "CONFIG_FB_NEW_DISP1=y"							>> ${WORKDIR}/defconfig
		sed -i 's%\(^.*\)22M\(.*\)%\1${EXPECTED_ROOTFS_SIZE}\2%g'  ${S}/arch/arm/boot/dts/tcc/tcc803x-subcore.dtsi
	fi
}
```

### 1.2.2.2 Modify Linux Kernel Source

The following are the settings for using ***BitBake*** to modify Linux Kernel source code in Yocto for the sub-core.

```bash
$ source poky/oe-init-build-env build/tcc8050-sub/

Yocto Project common targets are:
    core-image-minimal
    core-image-sato
    meta-toolchain
    adt-installer
    meta-ide-support

Telechips subcore targets are:
    meta-toolchain-telechips(Application Development Toolkit)
    telechips-subcore-image(minimal rootfs for Cortex-A53)

You can also run generated qemu images with a command like 'runqemu qemux86'
 or
You can also run generated Telechips images on Telechips EVB Boards

Other commonly useful commands are:
 - 'devtool' and 'recipetool' handle common recipe tasks
 - 'bitbake-layers' handles common layer tasks
 - 'oe-pkgdata-util' handles common target package tasks
Warning: source-mirror not exist!!
         The build to be slower or fail because will be download source code from upstream before build.
```


Use ***devtool*** to download the Linux Kernel source code to workspace.

You can use the ***BitBake*** **-e** option to find out the source path as follows:

```bash
$ devtool modify virtual/kernel

$ bitbake virtual/kernel -e | grep ^S=
S=”/home/TOPST/build/tcc8050-sub/workspace/sources/linux-telechips”

$ cd /home/TOPST/build/tcc8050-sub/workspace/sources/linux-telechips
$ ls
android                            build.config.gki.aarch64          certs          kernel                  poky
arch                               build.config.gki-debug.aarch64    COPYING        lib                     README
block                              build.config.gki-debug.x86_64     CREDITS        LICENSES                README.md
build.config.aarch64               build.config.gki_kasan            crypto         MAINTAINERS             samples
build.config.allmodconfig          build.config.gki_kasan.aarch64    Documentation  Makefile                scripts
build.config.allmodconfig.aarch64  build.config.gki_kasan.x86_64     drivers        mklinuxmtdimg_64bit.sh  security
build.config.allmodconfig.arm      build.config.gki_kprobes          fs             mklinuxmtdimg.sh        sound
build.config.allmodconfig.x86_64   build.config.gki_kprobes.aarch64  include        mm                      tcc805x_customized_gpu_configuration.sh
build.config.arm                   build.config.gki_kprobes.x86_64   init           net                     tools
build.config.common                build.config.gki.x86_64           ipc            oe-logs                 usr
build.config.db845c                build.config.hikey960             Kbuild         oe-workdir              virt
build.config.gki                   build.config.x86_64               Kconfig        OWNERS
```


The kernel can use the **Cleanall** or **build** function  by using the ***BitBake*** **-c** option. However, the function must be defined as the task in the recipe.

```bash
$ bitbake virtual/kernel -c cleanall

$ bitbake virtual/kernel -c build

$ ls -al oe-workdir 
oe-workdir -> /home/TOPST /build/tcc8050-sub/tmp/work/tcc8050_sub-telechips-linux/linux-telechips/5.4.159-r0
```


The following describes how to modify the source by using Git and create a patch file.

The following is an example of modifying a DTS file.

```bash
$ vi arch/arm64/boot/dts/tcc/tcc8050-linux-subcore-ivi-TOPST_sv0.1.dts

$ git status
On branch tcc8050_linux_ivi_TOPST
Your branch is up to date with 'origin/tcc8050_linux_ivi_TOPST'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified: arch/arm64/boot/dts/tcc/tcc8050-linux-subcore-ivi-TOPST_sv0.1.dts

$ git commit -s -m <title of your commit>
```


Add patch file in the Linux Kernel recipe.

```bash
$ cp 001-test-sub-dts.patch poky/meta-telechips/meta-subcore/recipes-kernel/linux/

$ ls poky/meta-telechips/meta-subcore/recipes-kernel/linux/
001-test-sub-dts.patch

$ vi poky/meta-telechips/meta-subcore/recipes-kernel/linux/linux-telechips_%.bbappend
SRC_URI += “file://001-test-sub-dts.patch”
```


Re-build and execute Linux Kernel.

If you use the ***BitBake*** **-c build** option, the Linux Kernel binary is automatically saved as a path for binary downloads after compilation.

```bash
$ bitbake virtual/kernel -c cleansstate

$ bitbake virtual/kernel -c build
```


### 1.2.2.3 Configure Linux Kernel

The following shows how to use the Linux Kernel **menuconfig**.

```bash
$ bitbake virtual/kernel -c menuconfig
```

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/b1e7e6b8-3116-4ae5-9952-7f3e50607ddf" width="750" height="400" >
</p>

```bash
$ bitbake u-boot-tcc -c build
```


### 1.2.2.4 Device Tree for Linux Kernel

Configure the Linux Kernel device tree.

The following is information in the device tree file for developing the Linux Kernel device driver.

```bash
$ grep tcc8050-linux-subcore-ivi-TOPST poky/meta-telechips/meta-subcore/conf/machine/tcc8050-sub.conf
KERNEL_DEVICETREE ?= "${@bb.utils.contains('INVITE_PLATFORM', 'drm', 'tcc/tcc8050-linux-subcore_sv1.0_drm.dtb', 'tcc/tcc8050-linux-subcore-ivi-TOPST_sv0.1.dtb', d)}"
UPDATE_DTB_NAME = "tcc8050-linux-subcore-ivi-TOPST_sv1.0.dtb" 
```


The following is the device tree file in the kernel source code of the sub-core.

```bash
$ cd ~/TOPST/build/tcc8050-sub/workspace/sources/linux-telechips/
$ cat arch/arm64/boot/dts/tcc/tcc8050-linux-subcore-ivi-TOPST_sv1.0.dts

// SPDX-License-Identifier: (GPL-2.0-or-later OR MIT)
/*
 * Copyright (C) Telechips Inc.
 */

/dts-v1/;

/* Definition for H/W LCD ports */
#define LCD_PORT1_PWR "gpmc-19"
#define LCD_PORT1_RST "gpmc-20"
#define LCD_PORT1_BLK "gph-6"

#define LCD_PORT2_PWR "gpc-8"
#define LCD_PORT2_RST "gpc-9"
#define LCD_PORT2_BLK "gph-7"

#define LCD_PORT3_PWR "gpmc-2"
#define LCD_PORT3_RST "gpmc-18"
#define LCD_PORT3_BLK "gpe-15"

#define LCD_PORT4_PWR "gpmb-25"
#define LCD_PORT4_RST "gpmb-19"
#define LCD_PORT4_BLK "gpmc-21"

#include "tcc8050-subcore.dtsi"
#include <dt-bindings/pmap/tcc805x/pmap-tcc805x-linux-ivi-subcore-a53q.h>

/ {
	vqmmc_emmc: vqmmc_emmc {
		compatible = "regulator-fixed";
		regulator-name = "vqmmc_emmc";
		regulator-min-microvolt = <1800000>;
		regulator-max-microvolt = <1800000>;
		regulator-always-on;
	};

	tcc_sc_mmc: tcc_sc_mmc {
		compatible = "telechips,tcc805x-sc-mmc";
		status = "okay";

		max-frequency = <200000000>;
		bus-width = <8>;

		sc-firmware = <&tcc_sc_fw>;

		no-sdio;
		no-sd;
		non-removable;
		keep-power-in-suspend;
		disable-wp;

		mmc-hs400-1_8v;
		vqmmc-supply = <&vqmmc_emmc>;
	};

	tcc_sc_ufs: tcc_sc_ufs {
		compatible = "telechips,tcc805x-sc-ufs";
		status = "okay";
		sc-firmware = <&tcc_sc_fw>;
	};

	tcc_shared_memory: tcc_shared_memory {
		compatible = "telechips,tcc_shmem";
		reg = <0x0 0x38000000 0x0 0x8000000>;
		reg-names = "tcc_shmem";
		interrupts = <GIC_SPI 50 IRQ_TYPE_LEVEL_HIGH>;
		lable = "TCC shared memory";
		memory-region = <&pmap_tcc_shmem>;
		dist_regs = <0x17301000 0x1000 0x17C01000 0x1000>;
		core-num = <1>;
	};

	tcc_virt_eth: tcc_virt_eth {
		compatible = "telechips,tcc_virt_eth";
		status = "okay";
	};

        lcd_port3_bl: lcd_port3_bl {
                status = "okay";
                compatible = "gpio-backlight";
                gpios = <&gpe 15 0>;
                default-on;
        };

        fb_tm123xdhp90:fb_tm123xdhp90 {
                status = "okay";
                pinctrl-names = "default", "pwr_on_1", "pwr_on_2", "blk_on",
                                        "blk_off", "power_off";
                pinctrl-0 = <&lcd_port_default3>;
                pinctrl-1 = <&lcd_port_power_on3>;
                pinctrl-2 = <&lcd_port_reset_off3>;
                pinctrl-3 = <&lcd_port_blk_on3>;
                pinctrl-4 = <&lcd_port_blk_off3>;
                pinctrl-5 = <&lcd_port_reset_on3 &lcd_port_power_off3>;
                backlight = <&lcd_port3_bl>;
        };

	vbus_supply_ehci: vbus_supply_ehci {
		gpios = <&gpmc 7 0x0>;
	};
	
	vbus_supply_enable: vbus_supply_enable {
		compatible = "regulator-fixed";
		regulator-name = "vbus_supply_enable";
		regulator-min-microvolt = <5000000>;
		regulator-max-microvolt = <5000000>;
		gpios = <&gpmc 5 0x0>;
		enable-active-high;
		regulator-always-on;
	}; 
	
	tcc_gpio_sample {
		compatible = "gpio_sample";
		label = "TCC GPIO";
		interrupt-parent = <&gpa>;
		interrupts = <1 IRQ_TYPE_EDGE_BOTH>;
		gpios = <&gpa 2 0>;
	};

	imx219_mipi1_pd_enable: imx219_mipi1_pd_enable {
		compatible = "regulator-fixed";
		regulator-name = "imx219_mipi1_pd_enable";
		regulator-min-microvolt = <5000000>;
		regulator-max-microvolt = <5000000>;
		gpios = <&gpg 5 0x0>;
		enable-active-high;
		regulator-always-on;
		status = "disabled";
	}; 
};

&reserved_memory {
	pmap_tcc_shmem: tcc_shmem {
		reg = <0x0 0x38000000 0x0 0x8000000>;
		no-map;
		status = "disabled";
	};

	/*-----------------------------------------------------------
	 * Camera Memory
	 *-----------------------------------------------------------
	 */
	pmap_parking_gui: pmap_parking_gui {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PGL_BASE 0x0 PMAP_SIZE_CAMERA_PGL>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera_viqe: pmap_rearcamera_viqe {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_VIQE_BASE 0x0 PMAP_SIZE_CAMERA_VIQE>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera: pmap_rearcamera {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PREVIEW_BASE 0x0 PMAP_SIZE_CAMERA_PREVIEW0>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera1: pmap_rearcamera1 {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PREVIEW1_BASE 0x0 PMAP_SIZE_CAMERA_PREVIEW1>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera2: pmap_rearcamera2 {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PREVIEW2_BASE 0x0 PMAP_SIZE_CAMERA_PREVIEW2>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera3: pmap_rearcamera3 {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PREVIEW3_BASE 0x0 PMAP_SIZE_CAMERA_PREVIEW3>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera4: pmap_rearcamera4 {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PREVIEW4_BASE 0x0 PMAP_SIZE_CAMERA_PREVIEW4>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera5: pmap_rearcamera5 {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PREVIEW5_BASE 0x0 PMAP_SIZE_CAMERA_PREVIEW5>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera6: pmap_rearcamera6 {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PREVIEW6_BASE 0x0 PMAP_SIZE_CAMERA_PREVIEW6>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_rearcamera7: pmap_rearcamera7 {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_PREVIEW7_BASE 0x0 PMAP_SIZE_CAMERA_PREVIEW7>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_vin_lastframe: pmap_vin_lastframe {
		compatible = "shared-dma-pool";
		reg = <0x0 CAMERA_LASTFRAME_BASE 0x0 PMAP_SIZE_CAMERA_LASTFRAME>;
		alignment = <0x1000>;
		no-map;
	};

	/*-----------------------------------------------------------
	 * Secure Area 1 (CPU X, VPU X, GPU W, VIOC R)
	 *-----------------------------------------------------------
	 */
	pmap_fb_video: fb_video {
		compatible = "pmap,fb_video";
		pmap-name = "fb_video";
		reg = <0x0 FB_VIDEO_BASE 0x0 PMAP_SIZE_FB_VIDEO>;
		alignment = <0x1000>;
		no-map;
	};

	pmap_fb1_video: fb1_video {
		compatible = "pmap,fb1_video";
		pmap-name = "fb1_video";
		alloc-ranges = <0x0 RESERVED_HEAP_BASE 0x0 RESERVED_HEAP_SIZE>;
		size = <0x0 PMAP_SIZE_FB1_VIDEO>;
		no-map;
	};

	pmap_fb2_video: fb2_video {
		compatible = "pmap,fb2_video";
		pmap-name = "fb2_video";
		alloc-ranges = <0x0 RESERVED_HEAP_BASE 0x0 RESERVED_HEAP_SIZE>;
		size = <0x0 PMAP_SIZE_FB2_VIDEO>;
		no-map;
	};

	pmap_fb3_video: fb3_video {
		compatible = "pmap,fb3_video";
		pmap-name = "fb3_video";
		alloc-ranges = <0x0 RESERVED_HEAP_BASE 0x0 RESERVED_HEAP_SIZE>;
		size = <0x0 PMAP_SIZE_FB3_VIDEO>;
		no-map;
	};

	pmap_overlay_rot: overlay_rot {
		compatible = "pmap,overlay_rot";
		pmap-name = "overlay_rot";
		alloc-ranges = <0x0 RESERVED_HEAP_BASE 0x0 RESERVED_HEAP_SIZE>;
		size = <0x0 PMAP_SIZE_OVERLAY_ROT>;
		no-map;
	};
};

&a53_sc_mbox {
	status = "okay";
};

&tcc_sc_fw {
	status = "okay";
	mboxes = <&a53_sc_mbox 0>;
};

&cam_ipc0 {
	status = "okay";
};

&cam_ipc1 {
	status = "okay";
};

&switch0 {
	compatible = "telechips,switch";
	status = "okay";

	pinctrl-names = "default";
	pinctrl-0 = <&switch_mb23>;

	switch-gpios = <&gpmb 23 1>;
	switch-active = <0>;
};

&uart4 {
        pinctrl-0 = <&uart9_data>;
        status = "okay";
};

#include "tcc8050_53-videoinput-sv0.1.dtsi"
// #include "tcc805x-videoinput-parallel-sd.dtsi"
// #include "tcc805x-videoinput-mipi0-hd.dtsi"
#include "tcc805x-videoinput-mipi1-hd.dtsi"
//#include "tcc805x-videoinput-mipi1-hd-ispless.dtsi"
```

The following is information from the driver file developed for TOPST.

```bash
$ cd ~/TOPST/build/tcc8050-sub/workspace/sources/linux-telechips/

$ find drivers/ | grep tcc

drivers/gpu/drm/tcc
drivers/gpu/drm/tcc/tcc_drm_edid.h
drivers/gpu/drm/tcc/tcc_drm_screen_share.c
drivers/gpu/drm/tcc/vioc
drivers/gpu/drm/tcc/tcc_drm_fb.h
drivers/gpu/drm/tcc/tcc_drm_lcd_ext.c
drivers/gpu/drm/tcc/tcc_drm_drv.c
drivers/gpu/drm/tcc/tcc_drm_lcd_third.c
drivers/gpu/drm/tcc/tcc_drm_address.h
drivers/gpu/drm/tcc/tcc_drm_dpi.c
drivers/gpu/drm/tcc/tcc_drm_address.c
drivers/gpu/drm/tcc/tcc_drm_core.c
drivers/gpu/drm/tcc/tcc_drm_plane.h
drivers/gpu/drm/tcc/tcc_drm_gem.c
drivers/gpu/drm/tcc/tcc_drm_gem.h
drivers/gpu/drm/tcc/tcc_drm_lcd_fourth.c
drivers/gpu/drm/tcc/tcc_drm_lcd_base.c
drivers/gpu/drm/tcc/tcc_drm_drv.h
drivers/gpu/drm/tcc/tcc_drm_lcd_base.h
drivers/gpu/drm/tcc/Kconfig
drivers/gpu/drm/tcc/tcc_drm_fbdev.h
drivers/gpu/drm/tcc/tcc_drm_plane.c
drivers/gpu/drm/tcc/tcc_drm_edid.c
drivers/gpu/drm/tcc/tcc_drm_dpi.h
drivers/gpu/drm/tcc/Makefile
drivers/gpu/drm/tcc/tcc_drm_lcd_primary.c
drivers/gpu/drm/tcc/tcc_drm_fb.c
drivers/gpu/drm/tcc/tcc_drm_crtc.c
drivers/gpu/drm/tcc/tcc_drm_crtc.h
drivers/gpu/drm/tcc/tcc_drm_fbdev.c
drivers/gpu/drm/panel/panel-tcc.h
drivers/gpu/drm/panel/panel-tcc-dpv14.h
drivers/gpu/drm/panel/panel-tcc-dpv14.c
drivers/gpu/drm/panel/panel-lvds-tcc.c
drivers/gpu/imgtec/tcc_9xtp
drivers/gpu/imgtec/tcc_9xtp/sysconfig.h
drivers/gpu/imgtec/tcc_9xtp/sysconfig.c
drivers/gpu/imgtec/tcc_9xtp/tcc_9xtp_init.h
drivers/gpu/imgtec/tcc_9xtp/sysinfo.h
drivers/gpu/imgtec/tcc_9xtp/tcc_9xtp_init.c
drivers/gpu/imgtec/tcc_9xtp/Kconfig
drivers/gpu/imgtec/tcc_9xtp/Makefile
drivers/gpu/imgtec/pvrsrvkm/dc_fbdev_tcc.c
drivers/gpu/arm/utgard/mali/platform/tcc-m400
drivers/gpu/arm/utgard/mali/platform/tcc-m400/arm_core_scaling.c
drivers/gpu/arm/utgard/mali/platform/tcc-m400/mali_platform_dvfs.c
drivers/gpu/arm/utgard/mali/platform/tcc-m400/mali_platform.c
drivers/gpu/arm/utgard/mali/platform/tcc-m400/arm_core_scaling.h
drivers/gpu/arm/midgard/platform/tcc-mali-g51
drivers/gpu/arm/midgard/platform/tcc-mali-g51/mali_kbase_config_tcc.c
drivers/gpu/arm/midgard/platform/tcc-mali-g51/mali_kbase_config_platform.h
drivers/gpu/arm/midgard/platform/tcc-mali-g51/Kbuild
drivers/dma/tcc-dma.c
drivers/mmc/host/sdhci-tcc-mod.c
drivers/mmc/host/sdhci-tcc.h
drivers/mmc/host/sdhci-tcc.c
drivers/mmc/host/tcc_sc_mmc.c
drivers/spi/spi-tcc.c
drivers/spi/spi-tcc.h
drivers/spi/tcc_gpsb_tsif.c
drivers/tty/serial/tcc_serial.h
drivers/tty/serial/tcc_serial.c
drivers/power/reset/tcc-reboot-mode.c
drivers/watchdog/tcc805x_cbus_wdt.c
drivers/watchdog/tcc_pmu_wdt.c
drivers/watchdog/tcc803x_cbus_wdt.c
drivers/char/tcc_lut_3d_d1.c
drivers/char/tcc_lut.c
drivers/char/tcc_wdma.c
drivers/char/tcc_dxb_ctrl
drivers/char/tcc_dxb_ctrl/dxb_ctrl_gpio.h
drivers/char/tcc_dxb_ctrl/tcc_dxb_control.h
drivers/char/tcc_dxb_ctrl/amfmtuner.c
drivers/char/tcc_dxb_ctrl/dxb_ctrl_defs.h
drivers/char/tcc_dxb_ctrl/Kconfig
drivers/char/tcc_dxb_ctrl/dxb_ctrl_gpio.c
drivers/char/tcc_dxb_ctrl/Makefile
drivers/char/tcc_dxb_ctrl/amfmtuner.h
drivers/char/tcc_dxb_ctrl/dxb_ctrl.c
drivers/char/tcc_lut_3d_d0.c
drivers/char/tcc_aux_detect
drivers/char/tcc_aux_detect/aux_detect.c
drivers/char/tcc_aux_detect/Kconfig
drivers/char/tcc_aux_detect/Makefile
drivers/char/tcc_bluetooth.c
drivers/char/tcc_ipc
drivers/char/tcc_ipc/tcc_ipc_ctl.h
drivers/char/tcc_ipc/tcc_ipc_ctl.c
drivers/char/tcc_ipc/tcc_ipc_cmd.c
drivers/char/tcc_ipc/tcc_ipc_cmd.h
drivers/char/tcc_ipc/tcc_ipc_mbox.h
drivers/char/tcc_ipc/Kconfig
drivers/char/tcc_ipc/tcc_ipc_typedef.h
drivers/char/tcc_ipc/tcc_ipc_buffer.h
drivers/char/tcc_ipc/Makefile
drivers/char/tcc_ipc/tcc_ipc_os.h
drivers/char/tcc_ipc/tcc_ipc_mbox.c
drivers/char/tcc_ipc/tcc_ipc_buffer.c
drivers/char/tcc_ipc/tcc_ipc.c
drivers/char/tcc_ipc/tcc_ipc_os.c
drivers/char/tcc_lut.h
drivers/char/tcc_screen_share.h
drivers/char/hw_random/tcc-rng.h
drivers/char/hw_random/tcc-rng.c
drivers/char/tcc_thsm
drivers/char/tcc_thsm/tcc_thsm_cmd.c
drivers/char/tcc_thsm/Kconfig
drivers/char/tcc_thsm/tcc_thsm.c
drivers/char/tcc_thsm/tcc_thsm_log.h
drivers/char/tcc_thsm/Makefile
drivers/char/tcc_thsm/tcc_thsm_cmd.h
drivers/char/tcc_cp.c
drivers/char/tcc_ecid.c
drivers/char/tcc_screen_share.c
drivers/char/tcc_snor_updater
drivers/char/tcc_snor_updater/tcc_snor_updater_crc8.h
drivers/char/tcc_snor_updater/tcc_snor_updater.h
drivers/char/tcc_snor_updater/tcc_snor_updater_cmd.h
drivers/char/tcc_snor_updater/tcc_snor_updater_mbox.h
drivers/char/tcc_snor_updater/tcc_snor_updater_cmd.c
drivers/char/tcc_snor_updater/tcc_snor_updater_mbox.c
drivers/char/tcc_snor_updater/tcc_snor_updater_crc8.c
drivers/char/tcc_snor_updater/Kconfig
drivers/char/tcc_snor_updater/Makefile
drivers/char/tcc_snor_updater/tcc_snor_updater.c
drivers/char/tcc_snor_updater/tcc_snor_updater_dev.c
drivers/char/tcc_snor_updater/tcc_snor_updater_typedef.h
drivers/char/tcc_hsm
drivers/char/tcc_hsm/tcc803x
drivers/char/tcc_hsm/tcc803x/tcc_hsm_log.h
drivers/char/tcc_hsm/tcc803x/tcc_hsm_sp_cmd.c
drivers/char/tcc_hsm/tcc803x/tcc_hsm.c
drivers/char/tcc_hsm/tcc803x/tcc_hsm_sp_cmd.h
drivers/char/tcc_hsm/tcc805x
drivers/char/tcc_hsm/tcc805x/tcc_hsm.h
drivers/char/tcc_hsm/tcc805x/tcc_hsm_log.h
drivers/char/tcc_hsm/tcc805x/tcc_hsm_cmd.c
drivers/char/tcc_hsm/tcc805x/tcc_hsm_cmd.h
drivers/char/tcc_hsm/tcc805x/tcc_hsm.c
drivers/char/tcc_hsm/Kconfig
drivers/char/tcc_hsm/Makefile
drivers/reset/reset-tcc.c
drivers/thermal/tcc
drivers/thermal/tcc/tccxxxx_thermal.c
drivers/thermal/tcc/tcc_thermal.h
drivers/thermal/tcc/tcc805x_thermal.c
drivers/thermal/tcc/thermal_common.c
drivers/thermal/tcc/thermal_common.h
drivers/thermal/tcc/Kconfig
drivers/thermal/tcc/tcc_thermal.c
drivers/thermal/tcc/Makefile
drivers/media/platform/tcc-mipi-csi2
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-csi2.h
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-cfg-reg.h
drivers/media/platform/tcc-mipi-csi2/Kconfig
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-csi2-reg.h
drivers/media/platform/tcc-mipi-csi2/Makefile
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-csi2.c
drivers/media/platform/tcc-mipi-csi2/tcc-mipi-ckc-reg.h
drivers/media/platform/tccvin2
drivers/media/platform/tccvin2/basic_operation.h
drivers/media/platform/tccvin2/tccvin_diagnostics.h
drivers/media/platform/tccvin2/tccvin_driver.c
drivers/media/platform/tccvin2/tccvin_v4l2.c
drivers/media/platform/tccvin2/tccvin_diagnostics.c
drivers/media/platform/tccvin2/tccvin_video.h
drivers/media/platform/tccvin2/Kconfig
drivers/media/platform/tccvin2/Makefile
drivers/media/platform/tccvin2/tccvin_subdev.c
drivers/media/platform/tccvin2/tccvin_video.c
drivers/media/platform/tccvout
drivers/media/platform/tccvout/tcc_vout_attr.h
drivers/media/platform/tccvout/tcc_vout.h
drivers/media/platform/tccvout/tcc_vout_core.c
drivers/media/platform/tccvout/tcc_vout_v4l2.c
drivers/media/platform/tccvout/tcc_vout_core.h
drivers/media/platform/tccvout/tcc_vout_dbg.h
drivers/media/platform/tccvout/Kconfig
drivers/media/platform/tccvout/Makefile
drivers/media/platform/tccvout/tcc_vout_dbg.c
drivers/media/platform/tccvout/tcc_vout_attr.c
drivers/media/platform/tcc-isp
drivers/media/platform/tcc-isp/tcc-isp.h
drivers/media/platform/tcc-isp/tcc-isp-settings.h
drivers/media/platform/tcc-isp/Kconfig
drivers/media/platform/tcc-isp/Makefile
drivers/media/platform/tcc-isp/tcc-isp-reg.h
drivers/media/platform/tcc-isp/tcc-isp-iommu.c
drivers/media/platform/tcc-isp/tcc-isp.c
drivers/rtc/tcc_rtc.c
drivers/rtc/tcc
drivers/rtc/tcc/tca_alarm.h
drivers/rtc/tcc/tca_rtc.c
drivers/rtc/tcc/tca_alarm.c
drivers/rtc/tcc/Makefile
drivers/rtc/tcc/tca_rtc.h
drivers/misc/tcc
drivers/misc/tcc/gpio-tcc-sample.c
drivers/misc/tcc/scaler_drv.c
drivers/misc/tcc/wmixer_drv.c
drivers/misc/tcc/tcc_grp.c
drivers/misc/tcc/tcc_shared_mem.c
drivers/misc/tcc/Kconfig
drivers/misc/tcc/tcc_grp.h
drivers/misc/tcc/Makefile
drivers/misc/tcc/tcc_sdr
drivers/misc/tcc/tcc_sdr/tcc_sdr_hw.h
drivers/misc/tcc/tcc_sdr/tcc_sdr.h
drivers/misc/tcc/tcc_sdr/tcc_sdr_dai.h
drivers/misc/tcc/tcc_sdr/tcc_sdr.c
drivers/misc/tcc/tcc_sdr/tcc_sdr_adma.h
drivers/misc/tcc/tccmisc_drv.c
drivers/clk/tcc
drivers/clk/tcc/clk-tcc803x-a7s.c
drivers/clk/tcc/clk-tcc897x.h
drivers/clk/tcc/clk-tcc803x-a7s-reg.h
drivers/clk/tcc/clk.c
drivers/clk/tcc/clk-tcc803x.h
drivers/clk/tcc/clk-tcc897x.c
drivers/clk/tcc/Makefile
drivers/clk/tcc/clk-tcc803x.c
drivers/clk/tcc/clk-tcc805x.h
drivers/clk/tcc/clk-tcc805x.c
drivers/pwm/pwm-tcc.c
drivers/pci/controller/dwc/pci-tcc.c
drivers/net/ethernet/tcc_virt_eth.c
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-ecid.c
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-debugfs.c
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-ecid.h
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-v2.h
drivers/net/ethernet/stmicro/stmmac/dwmac-tcc-v2.c
drivers/net/phy/tcc_realtek.c
drivers/net/phy/tcc_marvell.c
drivers/scsi/ufs/tcc_sc_ufs.c
drivers/video/fbdev/tcc-fb
drivers/video/fbdev/tcc-fb/vioc
drivers/video/fbdev/tcc-fb/vioc/vioc_lvds.c
drivers/video/fbdev/tcc-fb/vioc/vioc_lvds_t.c
drivers/video/fbdev/tcc-fb/vioc/vioc_fifo.c
drivers/video/fbdev/tcc-fb/vioc/vioc_scaler.c
drivers/video/fbdev/tcc-fb/vioc/vioc_lut_3d.c
drivers/video/fbdev/tcc-fb/vioc/vioc_timer.c
drivers/video/fbdev/tcc-fb/vioc/vioc_pxdemux.c
drivers/video/fbdev/tcc-fb/vioc/vioc_wdma.c
drivers/video/fbdev/tcc-fb/vioc/vioc_vin.c
drivers/video/fbdev/tcc-fb/vioc/vioc_pvric_fbdc.c
drivers/video/fbdev/tcc-fb/vioc/vioc_mc.c
drivers/video/fbdev/tcc-fb/vioc/vioc_intr.c
drivers/video/fbdev/tcc-fb/vioc/vioc_lut.c
drivers/video/fbdev/tcc-fb/vioc/vioc_outcfg.c
drivers/video/fbdev/tcc-fb/vioc/vioc_rdma.c
drivers/video/fbdev/tcc-fb/vioc/vioc_config.c
drivers/video/fbdev/tcc-fb/vioc/tca_map_converter.c
drivers/video/fbdev/tcc-fb/vioc/vioc_wmix.c
drivers/video/fbdev/tcc-fb/vioc/Kconfig
drivers/video/fbdev/tcc-fb/vioc/vioc_afbcdec.c
drivers/video/fbdev/tcc-fb/vioc/vioc_deintls.c
drivers/video/fbdev/tcc-fb/vioc/Makefile
drivers/video/fbdev/tcc-fb/vioc/vioc_ddicfg.c
drivers/video/fbdev/tcc-fb/vioc/vioc_disp.c
drivers/video/fbdev/tcc-fb/vioc/vioc_viqe.c
drivers/video/fbdev/tcc-fb/g2d
drivers/video/fbdev/tcc-fb/g2d/tca_gre2d_api.c
drivers/video/fbdev/tcc-fb/g2d/Kconfig
drivers/video/fbdev/tcc-fb/g2d/Makefile
drivers/video/fbdev/tcc-fb/g2d/tcc_gre2d.c
drivers/video/fbdev/tcc-fb/fb.c
drivers/video/fbdev/tcc-fb/tcc_overlay.c
drivers/video/fbdev/tcc-fb/viqe
drivers/video/fbdev/tcc-fb/viqe/tcc_vioc_viqe.c
drivers/video/fbdev/tcc-fb/viqe/tcc_vioc_viqe_interface.c
drivers/video/fbdev/tcc-fb/viqe/tcc_vioc_viqe.h
drivers/video/fbdev/tcc-fb/viqe/viqe.c
drivers/video/fbdev/tcc-fb/viqe/viqe.h
drivers/video/fbdev/tcc-fb/viqe/Makefile
drivers/video/fbdev/tcc-fb/viqe/tcc_vioc_viqe_interface.h
drivers/video/fbdev/tcc-fb/Kconfig
drivers/video/fbdev/tcc-fb/panel
drivers/video/fbdev/tcc-fb/panel/panel-tcc.h
drivers/video/fbdev/tcc-fb/panel/Kconfig
drivers/video/fbdev/tcc-fb/panel/panel_helper.c
drivers/video/fbdev/tcc-fb/panel/Makefile
drivers/video/fbdev/tcc-fb/panel/panel_helper.h
drivers/video/fbdev/tcc-fb/panel/panel-lvds-tcc.c
drivers/video/fbdev/tcc-fb/Makefile
drivers/i2c/busses/i2c-tcc-v3.c
drivers/i2c/busses/i2c-slave-tcc-chdrv.c
drivers/i2c/busses/i2c-slave-tcc-chdrv.h
drivers/i2c/busses/i2c-tcc.c
drivers/i2c/busses/i2c-tcc-v2.c
drivers/input/tc_touch_share/tcc_touch_receive.c
drivers/input/tc_touch_share/tcc_touch_cmd.c
drivers/input/tc_touch_share/tcc_touch_bridge.c
drivers/input/tc_touch_share/tcc_touch_cmd.h
drivers/input/keyboard/tcc_keypad.c
drivers/input/touchscreen/serdes/tcc_tsc_serdes.c
drivers/mailbox/tcc_ipc_log.h
drivers/mailbox/tcc_sec_ipc.c
drivers/mailbox/tcc803x_multi_mailbox
drivers/mailbox/tcc803x_multi_mailbox/tcc_multi_mbox.c
drivers/mailbox/tcc803x_multi_mailbox/Makefile
drivers/mailbox/mailbox-tcc-sc.c
drivers/mailbox/tcc805x_multi_mailbox
drivers/mailbox/tcc805x_multi_mailbox/tcc_multi_mbox.c
drivers/mailbox/tcc805x_multi_mailbox/tcc_multi_mbox_test.c
drivers/mailbox/tcc805x_multi_mailbox/Makefile
drivers/usb/dwc3/dwc3-tcc.h
drivers/usb/dwc3/dwc3-tcc.c
drivers/usb/phy/phy-tcc-ehci.h
drivers/usb/phy/phy-tcc-ehci.c
drivers/usb/phy/phy-tcc-dwc_otg.c
drivers/usb/phy/phy-tcc-dwc_otg.h
drivers/usb/phy/phy-tcc-dwc3.h
drivers/usb/phy/phy-tcc-dwc3.c
drivers/usb/host/ehci-tcc.c
drivers/usb/host/ehci-tcc.h
drivers/usb/host/ohci-tcc.h
drivers/usb/host/ohci-tcc.c
drivers/pinctrl/tcc
drivers/pinctrl/tcc/Kconfig
drivers/pinctrl/tcc/Makefile
drivers/pinctrl/tcc/pinctrl-tcc.c
drivers/soc/tcc
drivers/soc/tcc/pm-tcc805x.c
drivers/soc/tcc/irq-pic.c
drivers/soc/tcc/timer.c
drivers/soc/tcc/chipinfo.c
drivers/soc/tcc/bootstage.c
drivers/soc/tcc/Kconfig
drivers/soc/tcc/Makefile
drivers/gpio/gpio-tcc.c
drivers/iio/adc/tcc_adc.h
drivers/iio/adc/tcc_adc.c
drivers/firmware/tcc_pmi.c
drivers/firmware/tcc_sc_fw.c

```

## 1.3 Feature Instructions

### 1.3.1 SD

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/09a478fe-394f-4bed-ae67-d61fecef467b" width="300" height="350" >
	<img src="https://github.com/Topst-Dev/Documentation/assets/144076415/a9bb5ccd-3eed-44df-93ae-f01c94e1e18b" width="650" height="400" >
</p>

### 1.3.1.1 U-Boot Configuration

**Note**: TOPST SD Host Controller 0 uses Storage Share Framework (SSF) by default. TOPST SD Host Controller 0 should not use SD Host Controller Interface (SDHCI) and Storage Share Framework (SSF) at the same time.

### 1.3.1.1.1 Device Tree Files

```bash
<Bootloader>arch/arm64/boot/dts/tcc/tcc805x.dtsi
<Bootloader>/arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi
<Bootloader>/arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts
```


The following shows pin configuration.

```bash
<Bootloader>/arch/arm/dts/tcc805x.dtsi

        sdhc0: sdhc@1D2A0000 {
                compatible = "telechips,tcc805x-sdhci";
                reg = <0x1D2A0000 0x100>,
                      <0x1D2D1000 0x50>,
                      <0x1D2D10FC 0x30>,
                      <0x1D2D112C 0x24>;
                max-frequency = <200000000>;
                peri-clock-frequency = <2600000000>;
                bus-width = <4>;
‘                controller-id = <0>;
                status = "disabled";
        };

        sdhc1: sdhc@16450000 {
                compatible = "telechips,tcc805x-sdhci";
                reg = <0x16450000 0x100>,
                      <0x16470050 0x50>,
                      <0x164700FC 0x30>,
                      <0x16470150 0x24>;
                max-frequency = <50000000>;
                bus-width = <4>;
                controller-id = <1>;
                status = "disabled";
        };

        sdhc2: sdhc@16460000 {
                compatible = "telechips,tcc805x-sdhci";
                reg = <0x16460000 0x100>,
                      <0x164700A0 0x50>,
                      <0x164700FC 0x30>,
                      <0x16470174 0x24>;
                max-frequency = <50000000>;
                bus-width = <4>;
                controller-id = <2>;
                status = "disabled";
        };
```


### 1.3.1.1.2 Properties

<p align="center"><strong>Table 1.1 Required Properties for SDHCI Driver in U-Boot</strong></p>

<table align="center">
	<tr>
		<th>Property</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>compatible</td>
		<td>Must be "telechips, < chipset >-sdhci". <br> <strong>Example:</strong><br> <li>"telechips, tcc8050-sdhci" for TCC8050</td>
	</tr>
	<tr>
		<td>reg</td>
		<td>The first one is for the SDHCI registers themselves.<br> The second one is for the channel control configuration registers.<br> The third one is for the channel clock delay configuration registers.<br> The last one is for the channel command and data delay registers.</td>
	</tr>
	<tr>
		<td>max-frequency</td>
		<td>Maximum operating clock frequency</td>
	</tr>
	<tr>
		<td>bus-width</td>
		<td>The number of data lines can be < 1 >, < 4 >, or < 8 >. </td>
	</tr>
	</tr>
		<td>controller-id</td>
		<td>Specify a controller ID.</td>
	</tr>
</table>

<p align="center"><strong>Table 1.2 Optional Properties for SDHCI Driver in U-Boot</strong></p>

<table align="center">
	<tr>
		<th>Property</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>mmc-pwrseq</td>
		<td>phandle to the MMC power sequence node</td>
	</tr>
	<tr>
		<td>non-removable</td>
		<td>Notify that a slot type is non-removable to the driver.</td>
	</tr>
	<tr>
		<td>mmc-hs200-1_8v</td>
		<td>Set the voltage as 1.8V for HS200.</td>
	</tr>
	<tr>
		<td>mmc-hs400-1_8v</td>
		<td>Set the voltage as 1.8V for HS400.</td>
	</tr>
	</tr>
		<td>tcc-mmc-taps</td>
		<td>Array of delay tap register settings:<br> <li>The first value is for OTAPDLYSEL field in TAPDLYn register. <li>The second value is for SDn_CMD_OUT_DLY, SDn_CMD_OEN_DLY, and SDn_CMD_IN_DLY fields in SDn_CMD_DLY register. <li>The third value is for SDn_DATAm_OUT_DLY, SDn_DATAm_OEN_DLY, and SDn_DATAm_IN_DLY fields in SDn_DATAm_DLY register. <li>The last value is for TX_CLK_DLY_n field in TX_CLK_DLY register.</td>
	</tr>
	<tr>
		<td>tcc-mmc-hs400-pos-tap</td>
		<td>Detection timing control value on positive edge of DQS signal for HS400</td>
	</tr>
	<tr>
		<td>tcc-mmc-hs400-neg_tap </td>
		<td>Detection timing control value on negative edge of DQS signal for HS400</td>
	</tr>
</table>

### 1.3.1.1.3 Configuration

```bash
$ make  menuconfig
Device Drivers ---> 
        MMC Host controller Support ---> 
                [*] MMC/SD/SDIO card support 
                -*- Enable MMC controllers using Driver Model 
                [*] enable HS400 support -*- enable HS200 support 
                [*] Secure Digital Host Controller Interface support 
                [*] Support SDHCI SDMA 
                [*] SDHCI support for the Telechips SD/MMC Controller 
                [ ] No retry the initialization
```


### 1.3.1.1.4 U-Boot MMC Command

You can test the driver by using **mmc** command. Refer to the following help messages of **mmc** command.

```bash
TCC # mmc
mmc - MMC sub system

Usage:
mmc info - display info of the current MMC device
mmc read addr blk# cnt
mmc write addr blk# cnt
mmc erase blk# cnt
mmc rescan
mmc part - lists available partition on current mmc device
mmc dev [dev] [part] - show or set current mmc device [partition]
mmc list - lists available devices
mmc hwpartition [args...] - does hardware partitioning
  arguments (sizes in 512-byte blocks):
    [user [enh start cnt] [wrrel {on|off}]] - sets user data area attributes
    [gp1|gp2|gp3|gp4 cnt [enh] [wrrel {on|off}]] - general purpose partition
    [check|set|complete] - mode, complete set partitioning completed
  WARNING: Partitioning is a write-once setting once it is set to complete.
  Power cycling is required to initialize partitions after set to complete.
mmc setdsr <value> - set DSR register value
```


### 1.3.1.2 Kernel Configuration

### 1.3.1.2.1 Kernel Configuration

```bash
$ make  menuconfig

Device Drivers --->
	<*> 	MMC/SD/SDIO card support --->
	<*> 	Secure Digital Host Controller Interface support
		<*> 	SDHCI platform and OF driver helper
		<*> 	SDHCI support for the Telechips SDHC controller
```


### 1.3.2 USB

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/b6574be4-d59c-4fc8-9ab0-09663aeb1a54" width="300" height="300">
	<img src="https://github.com/Topst-Dev/Documentation/assets/144076415/726793c6-0288-4ebd-888a-84cb032bf7ca" width="600" height="450">
</p>

### 1.3.2.1 Features

### 1.3.2.1.1 Hardware Overview

- USB 2.0 Host
- USB 2.0 host (EHCI/OHCI)
- USB 2.0 Dual-role device (DRD)
- OTG MUX (USB 2.0 host (EHCI/OHCI))
- USB 2.0 device mode
- USB 3.0 Dual-role device
- USB 3.0 host (XHCI/EHCI/OHCI)
- USB 3.0 device mode

### 1.3.2.1.2  Linux USB Stack Architecture

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/769e11da-f364-4934-a9ea-80b5cf52cf20" width="700" height="550" >
</p>

### 1.3.2.2 USB 2.0 Host

USB 2.0 Host Controller supports Enhanced Host Controller Interface (EHCI) specification and Open Host Controller Interface (OHCI) specification.

### 1.3.2.2.1 Device Tree

```bash
<Bootloader>arch/arm64/boot/dts/tcc/tcc805x.dtsi

        ehci: ehci@11A00000 {
                compatible = "telechips,tcc-ehci";
                reg = <0x11A00000 0x108>;
                phys = <&ehci_phy>;
                status = "disabled";
        };

        ehci_mux: ehci_mux@11900000 {
                compatible = "telechips,tcc-ehci";
                reg = <0x11900000 0x108>;
                phys = <&mhst_phy>;
                status = "disabled";
        };

        ohci: ohci@11A80000 {
                compatible = "telechips,tcc-ohci";
                reg = <0x11A80000 0x60>;
                status = "disabled";
        };

        ohci_mux: ohci_mux@11940000 {
                compatible = "telechips,tcc-ohci";
                reg = <0x11940000 0x60>;
                status = "disabled";
        };
```


### 1.3.2.2.2 Driver Source File

```bash
<kernel>/drivers/usb/host/ehci*.c
<kernel>/drivers/usb/host/ohci*.c
```


### 1.3.2.2.3 Configuration

```bash
Device Drivers  ---> 
	[*] USB support  ---> 
		<*>	EHCI HCD (USB 2.0) support 
		[*]     	Improved Transaction Translator scheduling 
		<*>     	Support for Telechips on-chip EHCI USB controller 
		-*-     	Generic EHCI driver for a platform device  
		… 
		<*>   	OHCI HCD (USB 1.1) support 
		<*>     	Support for Telechips on-chip OHCI USB controller 
		<*>     	OHCI support for PCI-bus USB controllers 
		-*-     	Generic OHCI driver for a platform device
```


### 1.3.2.3 USB 2.0 DRD Driver

USB 2.0 Dual-Role Device (DRD) controller that provides both device and host functions is supported. It can be also configured as a Host-only or Device-only controller that is fully compliant to the USB 2.0 Specification. The USB 2.0 configurations support high-speed (HS, 480 Mbps), full-speed (FS, 12 Mbps), and low-speed (LS, 1.5 Mbps) transfers. Additionally, it can be configured as USB 1.1 full-speed/low-speed DRD.

You can set the USB 2.0 DRD driver to host-only mode or set it to DRD mode to provide USB function role switching.

### 1.3.2.3.1 Device Tree

```bash
<Bootloader>arch/arm64/boot/dts/tcc/tcc805x.dtsi

        dwc_otg_phy: dwc_otg_phy@11DA0100 {
                compatible = "telechips,tcc_dwc_otg_phy";
                reg = <0x11DA0100 0x30>;
                #phy-cells = <0>;
                status = "disabled";
        };

        dwc_otg: dwc_otg@11980000 {
                compatible = "telechips,dwc2";
                reg = <0x11980000 0xcfff>;
                phys = <&dwc_otg_phy>;
                phy-names = "usb2-phy";
                dr_mode = "peripheral";
                g-np-tx-fifo-size = <16>;
                g-rx-fifo-size = <275>;
                g-tx-fifo-size = <256 128 128 64 64 32>;
                g-use-dma;
                status = "disabled";
        };
        
        mhst_phy: mux_host_phy@11DA00DC {
                compatible = "telechips,tcc_ehci_phy";
                reg = <0x11DA00DC 0x20>;
                #phy-cells = <0>;
                mux-port;
                //support-bc12;
                //support-line-state;
                status = "disabled";
        };
```


### 1.3.2.3.2 Driver Source Files

```bash
<kernel>/drivers/usb/dwc2/*
<kernel>/drivers/usb/host/ehci*.c
<kernel>/drivers/usb/host/ohci*.c
```

### 1.3.2.3.3 Configuration

The following is an example for host-only mode.

```bash
Device Drivers --->
	[*] USB support --->
		<*> 	DesignWare USB2 DRD Core Support
			DWC2 Mode Selection (Host only mode) --->
		[*] 	Enable TCC SoC dwc2
		[*] 	DWC2 Host Mux mode

```


The following is an example for DRD mode.

```bash
Device Drivers --->
	[*]	USB support --->
		<*> 	DesignWare USB2 DRD Core Support
			DWC2 Mode Selection (Dual Role mode) --->
		[*] 	Enable TCC SoC dwc2
		[*] 	DWC2 Host Mux mode
			DWC2 First Role Mode Selection (Enable TCC SoC dwc2 first host) --->
```


### 1.3.2.4 USB 3.0 DRD Driver

USB3.0 DRD controller that provides both device and host functions is supported. USB 3.0 DRD supports the following speed modes:

- USB host
- SuperSpeed
- High-speed
- Full-speed
- Low-speed
- USB device
- SuperSpeed
- High-speed
- Full-speed

You can set the USB 3.0 DRD driver to host-only mode or set it to DRD mode to provide USB function role switching.

### 1.3.2.4.1 Device Tree

```bash
<Bootloader>arch/arm64/boot/dts/tcc/tcc805x.dtsi

        dwc3_phy: dwc3_phy@11D90000 {
                compatible = "telechips,tcc_dwc3_phy";
                reg = <0x11D90000 0xB8>;
                #phy-cells = <0>;
                //support-bc12;
                //support-line-state;
                status = "disabled";
        };

        dwc3_platform: dwc3_platform {
                compatible = "telechips,tcc-dwc3";
                #address-cells = <1>;
                #size-cells = <1>;
                status = "disabled";
                ranges;

                dwc3: dwc3 {
                        compatible = "snps,dwc3";
                        reg = <0x11B00000 0xCFFF>;
                        phys = <&dwc3_phy>;
                        dr_mode = "host";
                        maximum-speed = "super-speed";
                };
        };
```


### 1.3.2.4.2 Driver Source Files

```bash
<kernel>/drivers/usb/dwc3/*
<kernel>/drivers/usb/host/xhci*.c
```


### 1.3.2.4.3 Configuration

The following is an example for host-only mode.

```bash
Device Drivers --->
	[*] USB support --->
		<*> 	xHCI HCD (USB 3.0) support
		<*> 	DesignWare USB3 DRD Core Support
			DWC3 Mode Selection (Host only mode) --->

```


The following is an example for DRD mode.

```bash
Device Drivers --->
	[*] USB support --->
		<*> 	USB Gadget Support --->
		<*> 	xHCI HCD (USB 3.0) support
		<*> 	DesignWare USB3 DRD Core Support
			DWC3 Mode Selection (Dual Role mode) --->
			DWC3 OTG Dual-role Initial Mode (Host mode) --->
```


### 1.3.3 Display

There are two types of DP-to-HDMI adapters: passive and active. TOPST supports only active adapters.

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/5175e774-6e5d-4124-a768-e08fca13887e" width="600" height="450" >
</p>

TOPST uses DisplayPort by default. This feature is supported by the main core.

Select DisplayPort or DP2HDMI by using the **dp2hdmi** feature in “Local.conf”.

When the DisplayPort settings are set in 'Local.conf' settings, they are applied in U-Boot and Kernel as shown below.

```bash
$ cat build/tcc8050-main/conf/local.conf

# DP to HDMI (1920x1080)
# if you are not defined, set the DP mode 
#INVITE_PLATFORM += "dp2hdmi"
```


The following is the settings for using **dp2hdmi** in U-Boot.

```bash
$ cat poky/meta-telechips-bsp/recipes-bsp/u-boot/u-boot-tcc.bb 

# DP to HDMI (1920x1080)
SRC_URI += "${@bb.utils.contains('INVITE_PLATFORM', 'dp2hdmi', 'file://add-dp2hdmi.cfg', '', d)}"
```


The following is the settings for using **DP** or **dp2hdmi** in the Kernel.

```bash
$ cat poky/meta-telechips-bsp/recipes-kernel/linux/linux-telechips.inc 

if ${@bb.utils.contains('INVITE_PLATFORM', 'dp2hdmi', 'true', 'false', d)}; then
		# DP to HDMI (1920x1080)
		echo "CONFIG_DRM_TCC_LCD_VIC=16"			>> ${WORKDIR}/defconfig
	else
		# DP mode
		echo "CONFIG_DRM_TCC_LCD_VIC=0"				>> ${WORKDIR}/defconfig
	fi
}
```


The following is the setting for the 1920x1080 60Hz enabled monitor in the Kernel.

For monitors with different resolutions, the following must be changed to suit those monitors.

```bash
$ cat arch/arm64/boot/dts/tcc/tcc805x-display.dtsi 

display-timings {
			native-mode = <&timing_dpv14_0>;
			timing_dpv14_0: 1920x1080p60 {
				clock-frequency = <148500000>;
				hactive = <1920>;
				vactive = <1080>;
				hfront-porch = <88>;
				hback-porch = <148>;
				hsync-len = <44>;
				hsync-active = <1>;
				vback-porch = <36>;
				vfront-porch = <4>;
				vsync-len = <5>;
				vsync-active = <1>;
				de-active = <1>;
				pixelclk-active = <1>;
			};
		};
```


### 1.3.3.1 Use DisplayPort

If **dp2hdmi** feature is set, disable the function as shown below.

```bash
$ cat build/tcc8050-main/conf/local.conf

# DP to HDMI (1920x1080)
# if you are not defined, set the DP mode 
#INVITE_PLATFORM += "dp2hdmi"
```


Rebuild the U-Boot and Kernel sources and apply the image to the TOPST D3 (Open platform board).

```bash
$ source poky/oe-init-build-env build/tcc8050-main/

$ Bitbake u-boot-tcc -c clean
$ Bitbake u-boot-tcc -c build

$ Bitbake virtual-kernel -c clean
$ Bitbake virtual-kernel -c build
```


### 1.3.3.2 Use DisplayPort-to-HDMI Active Adapter

Activate the **dp2hdmi** function as shown below.

```bash
$ cat build/tcc8050-main/conf/local.conf

# DP to HDMI (1920x1080)
# if you are not defined, set the DP mode 
INVITE_PLATFORM += "dp2hdmi"
```


Rebuild the U-Boot and Kernel sources and apply the image to the TOPST D3 (Open platform board).

```bash
$ source poky/oe-init-build-env build/tcc8050-main/

$ Bitbake u-boot-tcc -c clean
$ Bitbake u-boot-tcc -c build

$ Bitbake virtual-kernel -c clean
$ Bitbake virtual-kernel -c build
```

### 1.3.4 PCIe

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/e892503b-d493-4177-aaf6-e780ff56e7d1" width="750" height="550" >
</p>

### 1.3.4.1 Device Tree

```bash
<Kernel>/arch/arm64/boot/dts/tcc/tcc805x.dtsi
```


### 1.3.4.2 Driver Source Files

```bash
<Kernel>/arch/arm64/boot/dts/tcc/tcc805x.dtsi

pcie: pcie@11000000 {
        compatible = "telechips,pcie", "snps,dw-pcie";
        reg = <0 0x11000000 0 0x410000
               0 0x11100000 0 0x10000
               0 0x11120000 0 0x10000
               /* 5 0x00000000 0 0x100000 */
               5 0x00700000 0 0x100000
               0 0x11110000 0 0x10000>;
        reg-names = "dbi", "phy", "cfg", "config" , "clk_cfg";
        interrupts = <GIC_SPI 138 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&clk_peri PERI_PCIE0_PHY_CR_CLK>, <&clk_fbus FBUS_PCIe0>;
        clock-names = "pcie_phy", "fbus_pci0";
        resets = <&pmu_rst RESET_PCIE>;
        #address-cells = <3>;
        #size-cells = <2>;
        device_type = "pci";
        ranges = <0x81000000 0 0x00000000 5 0x00600000 0 0x00100000  /* downstream I/O */
                  0x82000000 0 0x00000000 5 0x0        0 0x00600000>;   /* non-prefetchable memory*/
        #interrupt-cells = <1>;
        interrupt-map-mask = <0 0 0 0>;
        interrupt-map = <0 0 0 0 &gic GIC_SPI 138 IRQ_TYPE_LEVEL_HIGH>;
        num-lanes = <1>;
        num-viewport = <4>;
        pci_gen = <3>;
        using_ext_ref_clk = <0>;
        for_si_test = <0>;
        ref_clk_pms = <0x0504C80C>;
        status = "disabled";
    };

```


### 1.3.4.3 Configuration

Kernel configuration requires the following kernel definitions:

```bash
# Networking support > Wireless
CONFIG_CFG80211=m
CONFIG_NL80211_TESTMODE=y
CONFIG_MAC80211=m
CONFIG_RFKILL=y

# Device Drivers > PCI
CONFIG_PCI=y
CONFIG_PCI_DOLPHIN3=y

# Intel wireless
CONFIG_IPW2100=m
CONFIG_IPW2100_MONITOR=y
CONFIG_IPW2100_DEBUG=y
CONFIG_IPW2200=m
CONFIG_IPW2200_RADIOTAP=y
CONFIG_IPW2200_PROMISCUOUS=y
CONFIG_IPW2200_QOS=y
CONFIG_IPW2200_DEBUG=y
CONFIG_LIBIPW_DEBUG=y
CONFIG_IWL49651=m
CONFIG_IWL39451=m
CONFIG_IWLEGACY_DEBUG=y
CONFIG_IWLWIFI=m
CONFIG_IWLDVM=m
CONFIG_IWLMVM=m
CONFIG_IWLWIFI_BCAST_FILTERING=y
CONFIG_IWLWIFI_DEVICE_TRACING=y
CONFIG_IWLWIFI_DEBUG=y

```


**Important:** If DMA is allocated for an area that exceeds the 32-bit area in endpoint (EP), the PCI may not operate. In this case, you should set **DMA_BIT_MASK(32)**.

### 1.3.5 Audio

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/46f69442-4f76-4cec-b96f-e926cae17f6c" width="500" height="250" >
</p>
<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/14e7d7f9-aeab-45ec-ae24-5e075b3f2b89" width="900" height="500" >
</p>

### 1.3.5.1 Feature (TBD)

### 1.3.5.2 Device Tree (TBD)

### 1.3.5.3 Configuration (TBD)

### 1.3.6 I2C Master

I2C is the master-slave protocol where communication takes place between a host adapter (or host controller) and client devices (or slaves), and it is also has two lines for clock and bidirectional data transfer. The corresponding lines are called Serial Clock (SCL) and Serial Data (SDA). The I2C master device nodes (such as /dev/i2c-x) are created for user space access.

### 1.3.6.1 U-Boot Configuration

### 1.3.6.1.1 Device Tree Files

```bash
<Bootloader>/arch/arm/dts/tcc805x.dtsi
<Bootloader>/arch/arm/dts/tcc8050_53-pinctrl.dtsi
<Bootloader>/arch/arm/dts/tcc8050-ivi-tost_sv0.1.dts
```


The following shows pin configuration:

```bash
<Bootloader>/arch/arm/dts/tcc805x.dtsi

        i2c0: i2c@16300000 {
                compatible = "telechips,tcc805x-i2c";
                reg = <0x16300000 0x28 0x163c0000 0x014>;
                #address-cells = <1>;
                #size-cells = <0>;
                clocks = <&clk_peri PERI_I2C0 &clk_io IOBUS_I2C_M0
                          &clk_fbus FBUS_IO>;
                clock-frequency = <400000>;
                port-mux = <0xFF>;      /* default value */
                status = "disabled";
        };

        i2c1: i2c@16310000 {
                compatible = "telechips,tcc805x-i2c";
                reg = <0x16310000 0x28 0x163c0000 0x14>;
                #address-cells = <1>;
                #size-cells = <0>;
                clocks = <&clk_peri PERI_I2C1 &clk_io IOBUS_I2C_M1
                          &clk_fbus FBUS_IO>;
                clock-frequency = <400000>;
                port-mux = <0xFF>;      /* default value */
                status = "disabled";
        };

        i2c2: i2c@16320000 {
                compatible = "telechips,tcc805x-i2c";
                reg = <0x16320000 0x28 0x163c0000 0x14>;
                #address-cells = <1>;
                #size-cells = <0>;
                clocks = <&clk_peri PERI_I2C2 &clk_io IOBUS_I2C_M2
                          &clk_fbus FBUS_IO>;
                clock-frequency = <400000>;
                port-mux = <0xFF>;      /* default value */
                status = "disabled";
        };

        i2c3: i2c@16330000 {
                compatible = "telechips,tcc805x-i2c";
                reg = <0x16330000 0x28 0x163c0000 0x14>;
                #address-cells = <1>;
                #size-cells = <0>;
                clocks = <&clk_peri PERI_I2C3 &clk_io IOBUS_I2C_M3
                          &clk_fbus FBUS_IO>;
                clock-frequency = <400000>;
                port-mux = <0xFF>;      /* default value */
                status = "disabled";
        };

        i2c4: i2c@16340000 {
                compatible = "telechips,tcc805x-i2c";
                reg = <0x16340000 0x28 0x163c0000 0x14>;
                #address-cells = <1>;
                #size-cells = <0>;
                clocks = <&clk_peri PERI_I2C0 &clk_io IOBUS_I2C_M4
                          &clk_fbus FBUS_IO>;
                clock-frequency = <400000>;
                port-mux = <0xFF>;      /* default value */
                status = "disabled";
        };

        i2c5: i2c@16350000 {
                compatible = "telechips,tcc805x-i2c";
                reg = <0x16350000 0x28 0x163c0000 0x14>;
                #address-cells = <1>;
                #size-cells = <0>;
                clocks = <&clk_peri PERI_I2C1 &clk_io IOBUS_I2C_M5
                          &clk_fbus FBUS_IO>;
                clock-frequency = <400000>;
                port-mux = <0xFF>;      /* default value */
                status = "disabled";
        };

        i2c6: i2c@16360000 {
                compatible = "telechips,tcc805x-i2c";
                reg = <0x16360000 0x28 0x163c0000 0x14>;
                #address-cells = <1>;
                #size-cells = <0>;
                clocks = <&clk_peri PERI_I2C2 &clk_io IOBUS_I2C_M6
                          &clk_fbus FBUS_IO>;
                clock-frequency = <400000>;
                port-mux = <0xFF>;      /* default value */
                status = "disabled";
        };

        i2c7: i2c@16370000 {
                compatible = "telechips,tcc805x-i2c";
                reg = <0x16370000 0x100 0x163c0000 0x14>;
                #address-cells = <1>;
                #size-cells = <0>;
                clocks = <&clk_peri PERI_I2C3 &clk_io IOBUS_I2C_M7
                          &clk_fbus FBUS_IO>;
                clock-frequency = <400000>;
                port-mux = <0xFF>;      /* default value */
                status = "disabled";
        };

        /omit-if-no-ref/
        i2c37_bus: i2c37_bus {
                telechips,pins = "gpmc-16", "gpmc-17";
                telechips,pin-function = <8>;
                telechips,input-enable;
                telechips,no-pull;
        };

        /omit-if-no-ref/
        i2c22_bus: i2c22_bus {
                telechips,pins = "gph-4", "gph-5";
                telechips,pin-function = <9>;
                telechips,input-enable;
                telechips,no-pull;
        };

        /omit-if-no-ref/
        i2c6_bus: i2c6_bus {
                telechips,pins = "gpa-22", "gpa-23";
                telechips,pin-function = <8>;
                telechips,input-enable;
                telechips,no-pull;
        };

        /omit-if-no-ref/
        i2c11_bus: i2c11_bus {
                telechips,pins = "gpb-19", "gpb-20";
                telechips,pin-function = <8>;
                telechips,input-enable;
                telechips,no-pull;
        };

        /omit-if-no-ref/
        i2c12_bus: i2c12_bus {
                telechips,pins = "gpb-21", "gpb-22";
                telechips,pin-function = <8>;
                telechips,input-enable;
                telechips,no-pull;
        };

<Bootloader>/arch/arm/dts/tcc8050-ivi-tost_sv0.1.dts

&i2c0 {
pinctrl-names = "default";
	       pinctrl-0 = <&i2c37_bus>;
	       port-mux = <37>;
	       status = "okay";
};
```


### 1.3.6.1.2 Properties

Properties for I2C master driver in device tree consist of required properties and optional properties.

**Note 1**: The same I2C master should not be enabled on AP and sub-core.

**Note 2**: **port-mux** numbers should not overlap. The default value is an exception.

<p align="center"><strong>Table 1.3 Required Properties of I2C Master Driver in Device Tree</strong></p>

<table align="center">
	<tr>
		<th>Property</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>compatible</td>
		<td>Name for matching with the drivers depending on chipset<br> "telechips,< chipset >- i2c"</td>
	</tr>
	<tr>
		<td>reg</td>
		<td>Specifies the physical base address and size of the registers</td>
	</tr>
	<tr>
		<td>#address-cells</td>
		<td>Should be < 1 ></td>
	</tr>
	<tr>
		<td>#size-cells</td>
		<td>Should be < 0 ></td>
	</tr>
	</tr>
		<td>clocks</td>
		<td>Array of clocks for I2C Master </td>
	</tr>
	<tr>
		<td>clock-frequency</td>
		<td>Desired I2C SCL speed in Hz</td>
	</tr>
	<tr>
		<td>pinctrl-names</td>
		<td>Should be “default”</td>
	</tr>
	<tr>
		<td>port-mux </td>
		<td>I2C port number</td>
	</tr>
	<tr>
		<td>pinctrl-0</td>
		<td>Pin control group to be used for I2C</td>
	</tr>
</table>

<p align="center"><strong>Table 1.4 Optional Properties of I2C Master Driver in Device Tree</strong></p>

<table align="center">
	<tr>
		<th>Property</th>
		<th>Default Value</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>pulse-width-high</td>
		<td>2</td>
		<td>Clock high width</td>
	</tr>
	<tr>
		<td>pulse-width-low</td>
		<td>3</td>
		<td>Clock low width</td>
	</tr>
	<tr>
		<td>noise_filter</td>
		<td>0</td>
		<td>Noise filter counter load value<br> <li>0: Disables noise filter</td>
	</tr>
	<tr>
		<td>status</td>
		<td>"okay"</td>
		<td>Activation status<br> "okay": Activates the device<br> "disabled": Deactivates the device</td>
	</tr>
</table>

### 1.3.6.1.3 Driver Source File

```bash
<Bootloader>/arch/arm /mach-telechips/tcc805x/include/i2c.h
<Bootloader>/drivers/i2c/tcc_i2c.c

```


### 1.3.6.1.4 Configuration

```bash
Device Driver --->
	<*> 	I2C support --->
		[*] 	Enable Driver Model for I2C drivers
		<*> 	Telechips I2C Master driver

```


### 1.3.6.1.5 Driver Test

For I2C master driver test, you can use the following **i2c** commands in U-Boot.

```bash
$ i2c
i2c - I2C sub-system
Usage:
i2c speed [speed] - show or set I2C bus speed
i2c dev [dev] - show or set current I2C bus
i2c md chip address[.0, .1, .2] [# of objects] - read from I2C device
i2c mm chip address[.0, .1, .2] - write to I2C device (auto-incrementing)
i2c mw chip address[.0, .1, .2] value [count] - write to I2C device (fill)
i2c nm chip address[.0, .1, .2] - write to I2C device (constant address)
i2c crc32 chip address[.0, .1, .2] count - compute CRC32 checksum
i2c probe - show devices on the I2C bus
i2c reset - re-init the I2C Controller
i2c loop chip address[.0, .1, .2] [# of objects] - looping read of device
```


### 1.3.6.2 Kernel Configuration

### 1.3.6.3 Device Tree

```bash
<Kernel>/arch/arm64/boot/dts/tcc/tcc805x.dtsi
<Kernel>/arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi
<Kernel>/arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts
```


### 1.3.6.3.1 Driver Source File

```bash
<Kernel>/drivers/i2c/busses/i2c-tcc.c
<Kernel>/drivers/i2c/busses/i2c-tcc-v2.c
<Kernel>/drivers/i2c/busses/i2c-tcc-v3.c
```


### 1.3.6.3.2 Configuration

```bash
$ make menuconfig ARCH=arm

Device Driver --->
	<*> 	I2C support --->
		[*] 	Enable compatibility bits for old user-space
		<*> 	I2C device interface
		[*] 	Autoselect pertinent helper modules
			I2C Hardware Bus support -->
			<*> 	Telechips I2C Master driver
```


### 1.3.6.3.3 Driver Test

You can test I2C driver by using ***i2c-tools***. Refer to i2c.wiki.kernel.org/index.php/I2C_Tool.

```bash
$ i2cdetect -l
$ i2cdetect <bus>
$ i2cdump <bus> <chipid>
$ i2cget <bus> <chip> <register>
$ i2cset <bus> <chip> <register> <value>

```

### 1.3.7 SPI Master

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/a14ee70c-1046-417d-814a-076da4124707" width="750" height="550" >
</p>

### 1.3.7.1 Device Tree

### 1.3.7.1.1 Device Tree Files

The following is the Device Tree information for U-Boot of the main core (CA72).

```bash
<Bootloader>/arch/arm64/boot/dts/tcc/tcc805x.dtsi
<Bootloader>/arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi
<Bootloader>/arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts
```

The following shows Pin configuration:

```bash
<Bootloader>/arch/arm64/boot/dts/tcc/tcc805x.dtsi

    gpsb0: spi@16900000 {
        compatible = "telechips,tcc805x-spi";
        reg = <0x0 0x16900000 0x0 0x38>, /* base address */
              <0x0 0x16960000 0x0 0x20>, /* Port Configuration register */
              <0x0 0x16980000 0x0 0x20>; /* GPSB_0_AC */
        interrupts = <GIC_SPI 55 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&clk_peri PERI_GPSB0 &clk_io IOBUS_GPSB0>;
        gpsb-id = <0>;
        status = "disabled";
    };

    gpsb2: spi@16920000 {
        compatible = "telechips,tcc805x-tsif";
        reg = <0x0 0x16920000 0x0 0x38>, /* base address */
              <0x0 0x16960000 0x0 0x20>, /* Port Configuration register */
              <0x0 0x16980040 0x0 0x20>; /* GPSB_2_AC */
        interrupts = <GIC_SPI 57 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&clk_peri PERI_GPSB2 &clk_io IOBUS_GPSB2>;
        gpsb-id = <1>;
        status = "disabled";
    };

<Bootloader>/arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi

	gpsb15_bus_idle: gpsb15_bus_idle {
		telechips,pins = "gpc-0", "gpc-1", "gpc-3", "gpc-2";
		telechips,pin-function = <0>;
		telechips,output-low;
		telechips,input_buffer_enable;
	};

	gpsb15_bus_spi: gpsb15_bus_spi {
		telechips,pins = "gpc-0", "gpc-1", "gpc-3", "gpc-2";
		telechips,pin-function = <6>;
		telechips,input_buffer_enable;
	};        

    gpsb22_bus_idle: gpsb22_bus_idle {
        telechips,pins = "gpg-7", "gpg-8", "gpg-10", "gpg-9";
        telechips,pin-function = <0>;
        telechips,output-low;
        telechips,input_buffer_enable;
    };

    gpsb22_bus_spi: gpsb22_bus_spi {
        telechips,pins = "gpg-7", "gpg-8", "gpg-10", "gpg-9";
        telechips,pin-function = <6>;
        telechips,input_buffer_enable;
    };

<Bootloader>/arch/arm/dts/tcc8050-ivi-tost_sv0.1.dts

&gpsb0 {
	       status = "okay";
	       gpsb-port = <15>;
	       pinctrl-names = "idle", "active";
	       pinctrl-0 = <&gpsb15_bus_idle>;
	       pinctrl-1 = <&gpsb15_bus_spi>;

	       /* cs-gpios */
	       cs-gpios = <&gpc 1 0>,<&gpc 6 0>;
};

&gpsb2 {
	       status = "okay";
	       compatible = "telechips,tcc805x-spi";
	       gpsb-port = <22>;
	       pinctrl-names = "idle", "active";
	       pinctrl-0 = <&gpsb22_bus_idle>;
	       pinctrl-1 = <&gpsb22_bus_spi>;

	       cs-gpios = <&gpg 8 0>;

	       #address-cells = <1>;
	       #size-cells = <0>;
	       spidev@0 {
			compatible = "rohm,dh2228fv";
			reg = <0>;
			spi-max-frequency = <20000000>;
	       };
};
```

### 1.3.7.1.2 Properties

Properties for SPI master driver in device tree consists of required properties and optional properties.

<p align="center"><strong>Table 1.5 Required Properties of SPI Master Driver in Device Tree</strong></p>

<table align="center">
	<tr>
		<th>Property</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>compatible</td>
		<td>Name for matching with the drivers depending on chipset<br> “telechips,< chipset >-spi” </td>
	</tr>
	<tr>
		<td>reg</td>
		<td>Specify the physical base address and size of the registers</td>
	</tr>
	<tr>
		<td>interrupts</td>
		<td>Interrupt specifier</td>
	</tr>
	<tr>
		<td>clocks</td>
		<td>Array of clocks for GPSB</td>
	</tr>
	<tr>
		<td>pinctrl-names</td>
		<td>Should be “idle”, “active”</td>
	</tr>
	<tr>
		<td>pinctrl-0</td>
		<td>Pin control group to be used for SPI in idle state</td>
	</tr>
	<tr>
		<td>pinctrl-1</td>
		<td>Pin control group to be used for SPI in active state</td>
	</tr>
	<tr>
		<td>gpsb-id</td>
		<td>Bus ID of SPI master device</td>
	</tr>
	<tr>
		<td>gpsb-port</td>
		<td>SPI port number</td>
	</tr>
</table>

<p align="center"><strong>Table 1.6 Optional Properties of SPI Master Driver in Device Tree</strong></p>

<table align="center">
	<tr>
		<th>Property</th>
		<th>Default Value</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>spi-max-frequency</td>
		<td>20 MHz</td>
		<td>Maximum speed</td>
	</tr>
	<tr>
		<td>ctf-mode-disable</td>
		<td>0</td>
		<td><li>0: Continuous transfer mode <li>1: Single mode </td>
	</tr>
	<tr>
		<td>status</td>
		<td>"okay"</td>
		<td>Activation status<br> “okay”: Activates the device.<br> “disabled”: Deactivates the device.</td>
	</tr>
	<tr>
		<td>access-control0<br> access-control1<br> access-control2<br> access-control3</td>
		<td>0x00000000 to 0xFFFFFFFF</td>
		<td>Address filtering<br><br> <strong>Note 1:</strong>DMA can access only a region of addresses.<br> <strong>Note 2:</strong> If the region of DMA addresses is out of the range that you set in device tree, GPSB sends 0x00 and cannot read the data.</td>
	</tr>
</table>

### 1.3.7.2 Kernel Configuration

### 1.3.7.2.1 Driver Source File

The driver file is located in the following path:

```bash
<Kernel>/drivers/spi/spi-tcc.c
<Kernel>/drivers/spi/spidev.c

```

### 1.3.7.2.2 Configuration

To set SPI driver activity in kernel, execute the following kernel configuration tool:

```bash
$ bitbake virtual/kernel -c menuconfig

Device Driver ---> 
	<*>	SPI support ---> 
	<*> 	Telechips SPI(GPSB) Master controller 
	<*> 	User mode SPI device driver support

```

If **User mode SPI device driver support** is enabled, compile drivers/spi/spidev.c. You can find more detailed information in <Kernel>/Documentation/spi/spidev.

If you want to use SPI with GDMA, set the menu as follows:

```bash
Device Driver ---> 
	[*] 	DMA Engine support ---> 
	[*] 	Telechips DMA support 

```

After system booting, you can find the device nodes through the command below.

```bash
root@jammy:~# ls /dev/spidev0.*
/dev/spidev0.0  /dev/spidev0.1
```

### 1.3.8 GPIO

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/6ee529df-9d4e-42ec-835f-fe323be8d7af" width="750" height="550" >
</p>

### 1.3.8.1 U-Boot Configuration

### 1.3.8.1.1 Device Tree Files

```bash
Telechips device tree files are located in <Kernel>/arch/arm64/boot/dts/tcc/

tcc8050.dtsi
tcc805x-linux.dtsi
tcc8050_53-pinctrl.dtsi
tcc8050-linux-ivi.dtsi
```

The following show pin configurations:

```bash
u-boot-tcc/arch/arm/dts/tcc805x.dtsi

gpa: gpa- {
		compatible = "telechips,tcc805x-gpio";
		gpio-controller;
		#gpio-cells = <2>;
		reg = <0x0 32>;
	};

	gpb: gpb- {
		compatible = "telechips,tcc805x-gpio";
		gpio-controller;
		#gpio-cells = <2>;
		reg = <0x40 29>;
	};

	gpc: gpc- {
		compatible = "telechips,tcc805x-gpio";
		gpio-controller;
		#gpio-cells = <2>;
		reg = <0x80 30>;
	};

arch/arm/dts/tcc8050_53-pinctrl.dtsi

&pinctrl {

	u-boot,dm-pre-reloc;

	uart0_data: uart0_data {
		telechips,pins = "gpsd0-11", "gpc-12";
		telechips,pin-function = <7>;
		telechips,input-enable;
		u-boot,dm-pre-reloc;
	};

	uart1_data: uart0_data {
		telechips,pins = "gpsd1-0", "gpsd1-1";
		telechips,pin-function = <7>;
		telechips,input-enable;
		u-boot,dm-pre-reloc;
	};

```

### 1.3.8.1.2 Properties

<p align="center"><strong>Table 1.7 Pin Control Options</strong></p>

<table align="center">
  <tr>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>GPIO_ACTIVE_HIGH</td>
    <td>If GPIO data is 1, the GPIO is output in high state.</td>
  </tr>
  <tr>
    <td>GPIO_ACTIVE_LOW</td>
    <td>If GPIO data is 1, the GPIO is output in low state.</td>
  </tr>
  <tr>
    <td>telechips,drive-strength</td>
    <td>Set port driver strength</td>
  </tr>
  <tr>
    <td>telechips,no-pull</td>
    <td>Disable pull-up/down</td>
  </tr>
  <tr>
    <td>telechips,pull-up</td>
    <td rowspan="2">Select pull-up/down and enables pull-up/down</td>
  </tr>
  <tr>
    <td>Telechips,pull-down</td>
  </tr>
  <tr>
    <td>telechips,input-enable</td>
    <td>Enable input buffer</td>
  </tr>
  <tr>
    <td>telechips,output-low</td>
    <td rowspan="2">Enable output and set to low/high</td>
  </tr>
  <tr>
    <td>telechips,output-high</td>
  </tr>
  <tr>
    <td>telechips,schmitt-input</td>
    <td rowspan="2">Set to schmitt input type or CMOS input type</td>
  </tr>
  <tr>
    <td>telechips,cmos-input</td>
  </tr>
  <tr>
    <td rowspan="2">telechips,slow-slew</td>
    <td>Select slew rate to slow</td>
  </tr>
  <tr>
    <td>telechips,fast-slew</td>
  </tr>
</table>

### 1.3.8.1.3 Driver Source File

```bash
<U-boot-tcc>/drivers/gpio/tcc_legacy_gpio.c
<U-boot-tcc>/drivers/gpio/tcc_gpio.c
<U-boot-tcc>drivers/pinctrl/pinctrl-tcc.c
```

### 1.3.8.1.4 Configuration

```bash
$ make  menuconfig

Device Driver ---> 
	GPIO support ---> 
	<*> 	Telechips GPIO driver
	<*> 	Enable Driver Model for GPIO drivers
	<*> 	Telechips GPIO driver model

```

### 1.3.8.1.5 Example for Driver

The following is an example of changing the state of **GPIO_B_14** to high.

```bash
u-boot-tcc/arch/arm/dts/tcc8050-ivi-TOPST_sv0.1.dts

videosource0: videodecoder_adv7182 {
	compatible	= "analogdevices,adv7182";
	pinctrl-names	= "idle", "active";
	pinctrl-0	= <&cam0_idle>;
	pinctrl-1	= <&cam0_rst &cam0_clk &cam0_hsync &cam0_vsync
			   &cam0_fld &cam0_de &cam0_data>;
	rst-gpios	= <&gpb 14 GPIO_ACTIVE_HIGH>;
	reg		= <0x21>;	// 0x42 >> 1
	cifport		= <0>;
};

u-boot-tcc/drivers/camera/earlycam/dev/videosource_if.c

static int videosource_ofdata_to_platdata(struct udevice *dev)
{
	/* coverity[misra_c_2012_rule_11_5_violation : FALSE] */
	struct vs_gpio		*plat	= dev_get_platdata(dev);
	int			ret	= 0;

	(void)gpio_request_by_name(dev,
		"rst-gpios", 0, &plat->rst_port, GPIOD_IS_OUT);

	return ret;
}
```

### 1.3.8.2 Kernel Configuration

### 1.3.8.2.1 Device Tree

GPIO group information is as follows:

```bash
arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi
	
&gpio {

    gpa: gpa {
		reg-offset = <0x0>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
		source-num = <1 0 1 32>;
		label = "gpioa";
		gpio-ranges = <&pinctrl 0 0 32>;
	    };

	    gpb: gpb {
		reg-offset = <0x40>;
		gpio-controller;
		#gpio-cells = <2>;
		interrupt-controller;
		#interrupt-cells = <2>;
		source-num = <1 0 33 29>;
		label = "gpiob";
		gpio-ranges = <&pinctrl 0 32 29>;
	    };

&pinctrl {
    pinctrl_gpio {
	gpa {
		pin-info = <0x0 0 32>;
		source-num = <1 0 1 32>;
	};

	gpb {
		pin-info = <0x40 0 29>;
		source-num = <1 0 33 29>;
	};

arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi

	i2c3_bus: i2c3_bus {
		telechips,pins = "gpa-4", "gpa-5";
		telechips,pin-function = <8>;
		telechips,input_buffer_enable;
		telechips,no-pull;
	};

arch/arm64/boot/dts/tcc/tcc805x.dtsi

vbus_supply_ehci: vbus_supply_ehci {
	compatible = "regulator-gpio";
	regulator-name = "vbus_supply_ehci";
	regulator-type = "voltage";
	regulator-min-microvolt = <0000001>;
	regulator-max-microvolt = <5000000>;
	gpios-states = <0x0>;
	states = <0000001 0x0 5000000 0x1>;
	status = "disabled";
};

arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts

	tcc_gpio_sample {
		compatible = "gpio_sample";
		label = "TCC GPIO";
		interrupt-parent = <&gpa>;
		interrupts = <1 IRQ_TYPE_EDGE_BOTH>;
		gpios = <&gpa 2 0>;
	};

```

**Example**:

```bash
- &gpx: x represents small letter of GPIO group such as A, B, C, D, etc. The following example applies to GPIO D group. 
	- arg 1: GPIO number 
	- arg 2: Option 

- gpio-states = <arg1 arg2>; 
	- arg 1: GPIO D 25 is set to low (0). 
	- arg 2: GPIO D 24 is set to high (1). 

regulators { 
gpios = <&gpd 25 0x4 &gpd 24 0x4>; 
gpios-states = <0 1>; 
}

```

### 1.3.8.2.2 GPIO sysfs Interface

The kernel provides an interface via **sysfs** to use GPIO in the user space. The documentation of how to use GPIO **sysfs interface** is located in “Documentation/gpio/sysfs.txt” file in the kernel.

To include **sysfs**, add the following in **menuconfig**.

```bash
$ bitbake virtual/kernel -c menuconfig

Device Drivers ---> 
GPIO Support ---> 
[*] /sys/class/gpio/… (sysfs interface)
```

- GPIO Number

A pinctrl file is available for the chip you are using. If your platform is TCC8050, you should use “tcc8050-pinctrl.dtsi”. This file defines GPIO to use **gpio a/b/c/d/e** and so on. The **reg** value determines the value in **sysfs**.

For example, the value of **gpio a** is 32, and is defined in the **reg** value. **gpio a** is the first of GPIO, so it can be input to **sysfs** from 0 to 31.

If you pass **gpio b, gpio b 0** will start from 32 because **gpio a** uses 0 to 31.

If you use **gpio b 3** as an example, you should enter **sysfs** as 32 + 3.

The GPIO number assigned to each GPIO is shown in red below. The GPIO increases the GPIO value from 0 to 31 and inputs it into **sysfs**.

```bash
gpa: gpa { 
gpio-controller; 
#gpio-cells = <2>; 
reg = <0x0 0 32>; 
}; 

gpb: gpb { 
gpio-controller; 
#gpio-cells = <2>; 
reg = <0x40 0 32>; 
}; 

gpc: gpc { 
gpio-controller; 
#gpio-cells = <2>; 
reg = <0x80 0 32>; 
};

```

### 1.3.8.2.3 GPIO dev Interface

- Preparation of GPIO dev Interface (**libgpiod**)

GPIO sysfs interface is deprecated and a new GPIO dev interface is merged. The GPIO dev interface requires libgpiody, which is a C library and tools for GPIO chardev, and the GPIO dev interface uses the device nodes named gpiochip0, gpiochip1, ghiochip2, and so on in /dev directory.

The following example is simple and only covers the basic functions of controlling GPIO.

- Example for Raspberry GPIO Pin (**RPI GPIO_C22 PIN7**)

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

// RPI GPIO_C22 PIN7

int main(int argc, char *argv[])
{
    int fd;
    char buf[64];
    int pin = 22; // choose a GPIO pin
    int value = 1; // choose a value (0 or 1)

    // open GPIO chardev number 0. GPIO group C. 
    snprintf(buf, sizeof(buf), "/dev/gpiochip2");
    fd = open(buf, O_RDWR);
    if (fd < 0) {
        perror("Failed to open gpiochip0");
        exit(1);
    }

// get the desired line of GPIO chip. GPIO_C22. 
    snprintf(buf, sizeof(buf), "gpio%d/direction", pin);
    if (write(fd, "out", 4) < 0) {
        perror("Failed to set direction of gpio pin");
        exit(1);
    }

    // set the value of the GPIO pin. GPIO_C22.
    snprintf(buf, sizeof(buf), "gpio%d/value", pin);
    if (write(fd, value ? "1" : "0", 2) < 0) {
        perror("Failed to set value of gpio pin");
        exit(1);
    }

    // close the GPIO chip device
    close(fd);

    return 0;
}

```

### 1.3.8.2.4 GPIO Pin Control

GPIO configurations such as pull-up/down, schmitt-input, and drive-strength can be configured with DT and applied in device driver. The GPIO state can be changed within the range from pinctrl-0 to pinctrl-N. The pinctrl-N is mapped to pinctrl-names in sequential order.

The following example shows how the GPIO key device driver changes the GPIO state according to the suspended or resumed state. The resumed state (**pinctrl-names: "default"**) is set based on the information of **pinctrl-0**. The suspended state (**pinctrl-names: sleep**) is set based on the information of **pinctrl-1**.

```bash
arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi

	uart20_data: uart20_data {
		telechips,pins = "gpc-16", "gpc-17";
		telechips,pin-function = <7>;
		telechips,input_buffer_enable;
		telechips,pull-up;
	};

drivers/pinctrl/tcc/pinctrl-tcc.c

static struct tcc_pinconf tcc_pin_configs[] = {
	{"telechips,drive-strength", TCC_PINCONF_PARAM_DRIVE_STRENGTH},
	{"telechips,no-pull", TCC_PINCONF_PARAM_NO_PULL},
	{"telechips,pull-up", TCC_PINCONF_PARAM_PULL_UP},
	{"telechips,pull-down", TCC_PINCONF_PARAM_PULL_DOWN},
	{"telechips,input-enable", TCC_PINCONF_PARAM_INPUT_ENABLE},
	{"telechips,output-low", TCC_PINCONF_PARAM_OUTPUT_LOW},
	{"telechips,output-high", TCC_PINCONF_PARAM_OUTPUT_HIGH},
	{"telechips,input_buffer_enable",
		TCC_PINCONF_PARAM_INPUT_BUFFER_ENABLE},
	{"telechips,input_buffer_disable",
		TCC_PINCONF_PARAM_INPUT_BUFFER_DISABLE},
	{"telechips,schmitt-input", TCC_PINCONF_PARAM_SCHMITT_INPUT},
	{"telechips,cmos-input", TCC_PINCONF_PARAM_CMOS_INPUT},
	{"telechips,slow-slew", TCC_PINCONF_PARAM_SLOW_SLEW},
	{"telechips,fast-slew", TCC_PINCONF_PARAM_FAST_SLEW},
	{"telechips,eclk-sel", TCC_PINCONF_PARAM_ECLK_SEL},
};

static const struct of_device_id tcc_pinctrl_of_match[] = {
	{
		.compatible = "telechips,tcc-pinctrl",},
	{ },
};

```

### 1.3.8.2.5 Properties

<p align="center"><strong>Table 1.8 Pin Control Options</strong></p>

<table align="center">
  <tr>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>telechips,drive-strength</td>
    <td>Set port driver strength.</td>
  </tr>
  <tr>
    <td>telechips,no-pull</td>
    <td>Disable pull-up/down.</td>
  </tr>
  <tr>
    <td>telechips,pull-up</td>
    <td rowspan="2">Select pull-up/down and enable pull-up/down.</td>
  </tr>
  <tr>
    <td>Telechips,pull-down</td>
  </tr>
  <tr>
    <td>telechips,input-enable</td>
    <td>Disable output.</td>
  </tr>
  <tr>
    <td>telechips,output-low</td>
    <td rowspan="2">Enable output and set to low/high.</td>
  </tr>
  <tr>
    <td>telechips,output-high</td>
  </tr>
  <tr>
    <td>telechips,input_buffer_enable</td>
    <td rowspan="2">Enable/disable input buffer.</td>
  </tr>
  <tr>
    <td>telechips,input_buffer_disable</td>
  </tr>
  <tr>
    <td>telechips,Schmitt-input</td>
    <td rowspan="2">Set to Schmitt input type or CMOS input type.</td>
  </tr>
  <tr>
    <td>telechips,cmos-input</td>
  </tr>
  <tr>
    <td>telechips,slow-slew</td>
    <td rowspan="2">Select slew rate to fast/slow.</td>
  </tr>
  <tr>
    <td>telechips,fast-slew</td>
  </tr>
  <tr>
    <td>telechips,eclk-sel</td>
    <td>Select external input clock.</td>
  </tr>
</table>


### 1.3.8.2.6 Example for Sample Driver

The GPIO property should be added to device node.

```bash
u-boot-tcc/arch/arm/dts/tcc8050-ivi-TOPST_sv0.1.dts
tcc-gpio-sample { 
compatible = "gpio-sample"; 
leble = "TCC GPIO"; 
interrupt-parent = <&gpa>; 
interrupts = <1 IRQ_TYPE_EDGE_BOTH>; 
}; 

```

The following example shows how to set external interrupt with sample driver:

```bash
drivers/misc/tcc/gpio-tcc-sample.c

static int gpio_sample_probe(struct platform_device *pdev)
{
...

isr = gpio_sample_isr;
irq = platform_get_irq(pdev, 0);
irq_type = irq_get_trigger_type(irq);

if(irq <= 0) {
debug_gpio("%s: irq null\n", __func__);
error = -EINVAL;
goto err_remove_group;
}

if ((irq_type == IRQF_TRIGGER_RISING)||
(irq_type == IRQF_TRIGGER_FALLING)||
(irq_type == (IRQF_TRIGGER_RISING|IRQF_TRIGGER_FALLING))) {

error = request_any_context_irq(irq, isr, 0, "sample_gpio_interrupt", dev);

// 1st : irq number, 2nd : interrupt service routine,
// 3rd : irq type(if '0', the type defined at device tree is used),
// 4th : name, 5th : parameter to isr.

}

static struct platform_driver gpio_sample_device_driver = {
	.probe		= gpio_sample_probe,
	.remove		= gpio_sample_remove,
	.driver		= {
		.name	= "gpio-tcc-sample",
		.pm	= &gpio_sample_pm_ops,
		.of_match_table = of_match_ptr(gpio_sample_of_match),
	}
};

```

### 1.3.9 UART

This chapter describes how to set up PL011 UART device by providing information on UART controller architecture and registers. This chapter also describes PIN multiplexing information and source code for port configuration.

### 1.3.9.1 Features

Eight port channels (Port 0 to Port 7)

- Hardware flow control
- Maximum speed: 3 Mbps
- Default UART console is configured with the following settings:
- Speed: 115200 bps
- Data bits: 8 bits
- Parity bits: No parity
- Stop bits: 1 bit
- Flow control: No flow control

### 1.3.9.2 Device Tree

### 1.3.9.2.1 Device Tree Files

```bash
<Bootloader>/arch/arm64/boot/dts/tcc/tcc805x.dtsi
<Bootloader>/arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi
<Bootloader>/arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts

```

### 1.3.9.2.2 Properties

<p align="center"><strong>Table 1.9 Node Data of UART Device Tree</strong></p>

<table align="center">
  <tr>
    <th>Parameter</th>
    <th>Format</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>node</td>
    <td>&lt;name&gt;[@&lt;unit-address&gt;]</td>
    <td>&lt;name&gt; is a simple ASCII string and can be up to 31 characters in length. 
    &lt;unit-address&gt; is the primary address used to access the device.</td>
  </tr>
  <tr>
    <td>compatible</td>
    <td>"&lt;manufacturer&gt;,&lt;model&gt;"</td>
    <td>Compatible is a list of strings. The first string in the list specifies the exact device which the node represents. 
    The following strings represent other devices which the device is compatible with.</td>
  </tr>
  <tr>
    <td>reg</td>
    <td>&lt;address length&gt;</td>
    <td>An address value is a list of one or more 32-bit integers called cells. 
    The length value as one register range with length 0 x 1000.</td>
  </tr>
  <tr>
    <td>interrupts</td>
    <td>&lt;int_type int_num trigger&gt;</td>
    <td>The parameter should contain the UART interrupt numbers.</td>
  </tr>
  <tr>
    <td>clocks</td>
    <td>&lt;peri_clk_label peri_idx&gt;, &lt;iobus_clk_label iobus_idx&gt;</td>
    <td>The first clock corresponds to the clock named UARTCLK on the IP block. 
    The second clock corresponds to the PCLK.</td>
  </tr>
  <tr>
    <td>clock-names</td>
    <td>"uart_clk_name", "io_bus_clk name"</td>
    <td>The listed first clock must be named “uartclk” and the listed second clock must be named “apb_pclk”</td>
  </tr>
  <tr>
    <td>status</td>
    <td>A node is enabled if:<br>"ok" or "okay"<br>A node is disabled if:<br>"disabled"</td>
    <td>Activation status: <br>“okay”: Activates the device.<br>“disabled”: Deactivates the device.</td>
  </tr>
</table>

### 1.3.9.2.3 Pin Configurations for Raspberry PIN Map

```bash
<Bootloader>/arch/arm64/boot/dts/tcc/tcc805x.dtsi

	uart2: serial@16620000 {
		compatible = "arm,pl011", "arm,primecell";
		reg = <0x0 0x16620000 0x0 0x1000>;
		config-reg = <0x16689000 0x8 9>;
		arm,primecell-periphid = <0x001feff7>;
		interrupts = <GIC_SPI 90 IRQ_TYPE_LEVEL_HIGH>;
		clocks = <&clk_peri PERI_UART2>, <&clk_io IOBUS_UART2>;
		clock-names = "uartclk", "apb_pclk";
		assigned-clocks = <&clk_peri PERI_UART2>;
		assigned-clock-rates = <48000000>;
		status = "disabled";
	};

arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi

	uart22_data: uart22_data {
		telechips,pins = "gpc-26", "gpc-27";
		telechips,pin-function = <7>;
		telechips,input_buffer_enable;
	};

arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts

&uart2 {
	status = "okay";
pinctrl-names = "default";
	              pinctrl-0 = <&uart22_data>;
};
```

### 1.3.9.3 Kernel Configuration

### 1.3.9.3.1 Driver Source File

The driver file is located in the following path:

```bash
<Kernel>/drivers/tty/serial/amba-pl011.c
```

### 1.3.9.3.2 Configuration

To set UART driver activity in kernel, execute the following kernel configuration tool.

```bash
$ bitbake virtual/kernel -c menuconfig

Platform selection ---> 
<*> Telechips TCC-based 
```

After system booting, you can find the device nodes through the command below.

```bash
root@jammy:~# cat /proc/tty/driver/ttyAMA     
serinfo:1.0 driver revision:
0: uart:PL011 rev1 mmio:0x16600000 irq:128 tx:1633 rx:130 RTS|DTR
1: uart:PL011 rev1 mmio:0x16610000 irq:129 tx:0 rx:0
2: uart:PL011 rev1 mmio:0x16620000 irq:130 tx:0 rx:0

root@jammy:~# ls /dev/ttyAMA*
/dev/ttyAMA0  /dev/ttyAMA1  /dev/ttyAMA2

root@jammy:~# dmesg | grep UART
[    0.021472] Serial: AMBA PL011 UART driver
[    0.049454] [INFO][PL011][UART00] baud_rate 115200, uart_clk 48000000
[    1.105072] [INFO][PL011][UART00] baud_rate 115200, uart_clk 48000000
[    2.848946] [INFO][PL011][UART00] baud_rate 115200, uart_clk 48000000 
```

### 1.3.9.4 Debug UART

The following is the UART setting information for debugging.

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/aa0a0d0b-45a2-47ad-b840-dc849330b949" width="350" height="250" >
	<img src="https://github.com/Topst-Dev/Documentation/assets/144076415/1967825b-8a66-46fe-bddd-61c334266473" width="600" height="450" >
</p>

### 1.3.9.4.1 Configuration for CA72

The following is the default configuration for U-Boot.

```bash
$ cat u-boot-tcc/drivers/serial/Kconfig

config BAUDRATE
    int "Default baudrate"
    default 115200

$ cat u-boot-tcc/arch/arm/dts/tcc805x.dtsi

	uart0: serial@16600000 {
		compatible = "arm,pl011", "arm,primecell";
		reg = <0x16600000 0x1000 0x16689000 0x8>;
		interrupts = <GIC_SPI 88 IRQ_TYPE_LEVEL_HIGH>;
		clocks = <&clk_peri PERI_UART0>, <&clk_io IOBUS_UART0>;
		clock = <48000000>;
		status = "disabled";
	};

$ cat u-boot-tcc/arch/arm/dts/tcc8050_53-pinctrl.dtsi

	uart18_data: uart18_data {
		telechips,pins = "gpc-10", "gpc-11";
		telechips,pin-function = <8>;
		telechips,input-enable;
		u-boot,dm-pre-reloc;
	};

$ cat u-boot-tcc/arch/arm/dts/tcc8050-ivi-TOPST_sv0.1.dts

&chosen {
	stdout-path = &uart0;
};

&uart0 {
	pinctrl-names = "default";
	              pinctrl-0 = <&uart18_data>;
	              status = "okay";
	u-boot,dm-pre-reloc;
};
```

The following is the Device Driver information for the Device Tree of U-Boot.

```bash
$ cat u-boot-tcc/drivers/serial/serial_pl01x.c

static const struct udevice_id pl01x_serial_id[] ={
    {.compatible = "arm,pl011", .data = TYPE_PL011},
    {.compatible = "arm,pl010", .data = TYPE_PL010},
    {}
};
```

The following is the U-Boot boot log for the main core (A72).

```bash
U-Boot 2020.01-00002-gc0ea191fda-dirty (Dec 15 2022 - 12:05:44 +0000)

CPU:   ARM Cortex-A72 Processor, Rev: r0p3
Model: Telechips TCC8050 IVI TOPST SV0.1 
Loading Environment from MMC... *** Warning - bad CRC, using default environment

In:    serial@16600000
Out:   serial@16600000
Err:   serial@16600000
blkread: mmc 0 is current device
AB: Booting slot: a
Hit any key to stop autoboot:  0 
=>
=> printenv 
baudrate=115200
bootargs=console=ttyAMA0,115200n8 cgroup_disable=memory
```

### 1.3.9.4.2 Configuration for CA53

The following is the Device Tree information for U-Boot of the sub-core (CA53).

```bash
$ cat u-boot-tcc-sub/arch/arm/dts/tcc805x.dtsi

	uart4: serial@16640000 {
		compatible = "arm,pl011", "arm,primecell";
		reg = <0x16640000 0x1000 0x16689000 0x8>;
		interrupts = <GIC_SPI 69 IRQ_TYPE_LEVEL_HIGH>;
		clocks = <&clk_peri PERI_UART4>, <&clk_io IOBUS_UART4>;
		clock = <48000000>;
		status = "disabled";
	};

$ cat u-boot-tcc-sub/arch/arm/dts/tcc8050_53-pinctrl.dtsi

	uart9_data: uart9_data {
		telechips,pins = "gpa-24", "gpa-25";
		telechips,pin-function = <7>;
		telechips,input-enable;
		u-boot,dm-pre-reloc;
};

$ cat u-boot-tcc-sub/arch/arm/dts/tcc8050-subcore-ivi-TOPST_sv0.1.dts

&chosen {
	stdout-path = &uart4;
};

&uart4 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart9_data>;
	status = "okay";
	u-boot,dm-pre-reloc;
};
```

The following is the U-Boot boot log for the sub-core (A53).

```bash
U-Boot 2020.01-00002-gc0ea191fda-dirty (Dec 15 2022 - 12:05:44 +0000)

CPU:   ARM Cortex-A53 Processor, Rev: r0p4
Model: Telechips TCC8050 IVI TOPST SV0.1 Subcore
   tcc_sc_mmc: 0
Loading Environment from MMC... *** Warning - bad CRC, using default environment

In:    serial@16640000
Out:   serial@16640000
Err:   serial@16640000
blkread: mmc 0 is current device
AB: Booting slot: a
Hit any key to stop autoboot:  0 
=>
=> printenv 
baudrate=115200
bootargs=console=ttyAMA4,115200n8 cgroup_disable=memory
```

### 1.3.10 PWM

This chapter describes how to set up Pulse Width Modulation (PWM) device by providing the information on PWM controller architecture. This chapter also provides a user guide for configuration and examples for PWM test.

### 1.3.10.1 Device Tree

### 1.3.10.1.1 Device Tree File

Telechips device tree files are located in <Kernel>/arch/arm64/boot/dts/tcc/.

```bash
<Kernel>/arch/arm64/boot/dts/tcc/tcc805x.dtsi
<Kernel>/arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi
<Kernel>/arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts
```

### 1.3.10.1.2 Properties

Properties for PWM driver in device tree consist of required properties and optional properties.

<table align="center">
  <tr>
    <th>Property</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Compatible</td>
    <td>Name for matching with the drivers depending on chipsets</td>
  </tr>
  <tr>
    <td>reg</td>
    <td>Physical base address and size of the controller's register area.</td>
  </tr>
  <tr>
    <td>tcc,pwm-number</td>
    <td>Should be 4. Total number of ports supported by the chip.
    Refer to Full Specification for ports information.</td>
  </tr>
  <tr>
    <td>#pwm-cells</td>
    <td>Should be &lt;2&gt;
    the pwm-cells property is used to define the number of cells that are used to represent a PWM signal.</td>
  </tr>
  <tr>
    <td>clocks</td>
    <td>phandle to input clock.</td>
  </tr>
  <tr>
    <td>clock-frequency</td>
    <td>Desired PWM frequency in Hz.</td>
  </tr>
  <tr>
    <td>pinctrl-names</td>
    <td>Should be "default".</td>
  </tr>
  <tr>
    <td>pinctrl-0</td>
    <td>phandle to pinctrl function.</td>
  </tr>
  <tr>
    <td>gfb-port</td>
    <td>Port mapping number for PDM0, PDM1, PDM2, PDM3</td>
  </tr>
  <tr>
    <td>status</td>
    <td>Activation status: <br>"okay”: Activates the device.<br>"disabled”: Deactivates the device.</td>
  </tr>
</table>

### 1.3.10.1.3 Example

The driver file is located in the following path:

```bash
<Kernel>/arch/arm64/boot/dts/tcc/tcc805x.dtsi

	pwm: pwm@16030000 { /* pwm */
		compatible = "telechips,pwm";
		reg = <0x0 0x16030000 0x0 0x90>,
		      <0x0 0x1605104C 0x0 0x4>,
		      <0x0 0x16051094 0x0 0x4>;
		tcc,pwm-number = <4>;
		#pwm-cells = <2>;
		clocks = <&clk_peri PERI_PDM &clk_io IOBUS_PWM>;
		status = "okay";
		clock-frequency = <400000000>;
	};

<Kernel>/arch/arm64/boot/dts/tcc/tcc8050_53-pinctrl.dtsi

	//RPI GPIO_C28/ PWM PIN12
	pwm64_out: pwm64_out {
		telechips,pins = "gpc-28";
		telechips,pin-function = <10>;
		telechips,drive-strength = <3>;
		telechips,no-pull = <1>;
		telechips,input_buffer_disable;
		telechips,input-disable;
	};

	pwm64_idle: pwm64_idle {
		telechips,pins = "gpc-28";
		telechips,pin-function = <0>;
		telechips,output-low = <0>;
	};

<Kernel>/arch/arm64/boot/dts/tcc/tcc8050-linux-ivi-tost_sv0.1.dts

&pwm {
		status = "okay";
		gfb-port = <64 68 69 255>;
		pinctrl-names = "default";
		pinctrl-0 = <&pwm64_out &pwm68_out &pwm69_out>;
};
```

### 1.3.10.2 Kernel Configuration

### 1.3.10.2.1 Driver Source File

The driver file is located in the following path:

```bash
<Kernel>/drivers/pwm/pwm-tcc.c
```

### 1.3.10.2.2 Configuration

To set UART driver activity in kernel, execute the following kernel configuration tool:

```bash
$ bitbake virtual/kernel -c menuconfig

Device Drivers ---> 
GPIO Support ---> 
[*] Pulse-Width Modulation (PWM) Support
    <*> Telechips PWM support
```

After system booting, you can find the device nodes through the command below.

```bash
root@jammy:~# ls /sys/class/pwm/pwmchip0/
device  export  npwm  power  subsystem  uevent  unexport  waiting_for_supplier
```