1. Create a workspace containing vanilla 'android-2.3.4_r1'  release from Google.

2. Overlay Motorola-provided published repos on top of original Google versions. 

3. Build user space components:

        cd <workspace>
        . build/envsetup.sh
        lunch generic-user
        make TARGET_BOARD_PLATFORM=tegra TARGET_ARCH_VARIANT=armv7-a \
            arch_variant_cflags="-march=armv7-a -mtune=cortex-a9 -mfloat-abi=softfp -mfpu=vfpv3-d16" \
            BOARD_HAVE_BLUETOOTH=true BOARD_HAVE_BLUETOOTH_BCM=true \
            BOARD_GPS_LIBRARIES= WPA_SUPPLICANT_VERSION=VER_0_6_X <target>

   Example for <target>: out/target/product/generic/system/bin/bluetoothd

make TARGET_BOARD_PLATFORM=tegra TARGET_ARCH_VARIANT=armv7-a arch_variant_cflags="-march=armv7-a -mtune=cortex-a9 -mfloat-abi=softfp -mfpu=vfpv3-d16" BOARD_HAVE_BLUETOOTH=true BOARD_HAVE_BLUETOOTH_BCM=true BOARD_GPS_LIBRARIES= WPA_SUPPLICANT_VERSION=VER_0_6_X out/target/product/generic/system/bin/bluetoothd

4. Building kernel and kernel modules. 

        export PLATFORM_DIR=<path to the android root>
        export KERNEL_BUILD_OUT=$PLATFORM_DIR/out/target/product/sunfire/obj/PARTITIONS/kernel_intermediates/build
        mkdir -p $KERNEL_BUILD_OUT
        export ARCH=arm
        export CROSS_COMPILE=$PLATFORM_DIR/prebuilt/linux-x86/toolchain/arm-eabi-4.4.0/bin/arm-eabi-
        export KERNEL_SRC=$PLATFORM_DIR/kernel/tegra
        make -j1 -C $KERNEL_SRC O=$KERNEL_BUILD_OUT KBUILD_DEFCONFIG=__ext_merge_defconfig defconfig modules_prepare
        make -j1 -C $KERNEL_SRC O=$KERNEL_BUILD_OUT DEPMOD=out/host/linux-x86/bin/depmod INSTALL_MOD_PATH=$KERNEL_BUILD_OUT modules
        make -j1 -C $KERNEL_SRC O=$KERNEL_BUILD_OUT DEPMOD=out/host/linux-x86/bin/depmod INSTALL_MOD_PATH=$KERNEL_BUILD_OUT modules_install
        make -j1 -C $KERNEL_SRC O=$KERNEL_BUILD_OUT zImage

vpndriver.ko:
        make -j1 -C $KERNEL_SRC O=$KERNEL_BUILD_OUT DEPMOD=out/host/linux-x86/bin/depmod INSTALL_MOD_PATH=$KERNEL_BUILD_OUT M=$PLATFORM_DIR/vendor/motorola/sunfire/vpndriver modules 

DHD.ko:
        make -j1 -C $KERNEL_SRC O=$KERNEL_BUILD_OUT DEPMOD=out/host/linux-x86/bin/depmod INSTALL_MOD_PATH=$KERNEL_BUILD_OUT M=$PLATFORM_DIR/vendor/bcm/wlan/osrc/open-src/src/dhd/linux/dhd-cdc-sdmmc-oob-gpl-2.6.32.9-00027-g0b50239 modules

bcmsdio.ko:
        make -j1 -C $KERNEL_SRC O=$KERNEL_BUILD_OUT DEPMOD=out/host/linux-x86/bin/depmod INSTALL_MOD_PATH=$KERNEL_BUILD_OUT M=$PLATFORM_DIR/vendor/bcm/wimax/Source/Driver/bcm_sdio_wrapper modules
