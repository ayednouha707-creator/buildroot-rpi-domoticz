# Custom embedded Linux for Raspberry Pi with Domoticz (Buildroot)

Lab project from my embedded Linux course at ISET Bizerte (November 2025): build a minimal Linux system for a Raspberry Pi from source with **Buildroot**, and include **Domoticz**, an open-source home-automation server, in the image.

The final image was built, flashed to an SD card and booted on the school's Raspberry Pi during the lab. That work was done on the class computers, so this repository only keeps the configuration and my notes, not the build output.

## What the image contains

- ARM Cortex-A53 target (Raspberry Pi 3), NEON / VFPv4
- Linux kernel and Raspberry Pi firmware / bootloader from the official `raspberrypi3_defconfig`
- BusyBox init and shell, root login, getty on the console
- Lua 5.3 and the Domoticz package
- ext4 root filesystem (200 MB) plus a tarball of the rootfs

`configs/rpi_domoticz_defconfig` is the configuration fragment I used for the Domoticz build (external ARM cross-toolchain, glibc, C++ support, Lua, Domoticz, ext4 rootfs).

## Build steps

Done in an Ubuntu virtual machine (VMware).

```bash
tar -xvf buildroot-2023.02.8.tar.gz
cd buildroot-2023.02.8

make distclean
make BR2_DEFCONFIG=configs/raspberrypi3_defconfig defconfig   # base board support
make menuconfig                                                # add Lua + Domoticz, filesystem options
make                                                           # toolchain, kernel, packages, rootfs

# result
ls output/images/        # sdcard.img, zImage, *.dtb, rootfs.ext4, boot.vfat
sudo dd if=output/images/sdcard.img of=/dev/sdX bs=4M status=progress   # flash the SD card
```

## Problems I ran into and how I fixed them

| Problem | Cause | Fix |
|---|---|---|
| No Raspberry Pi menus / `raspberrypi3_defconfig` not found | Buildroot 2025.08 reorganised the Raspberry Pi board configs | Switched to Buildroot **2023.02.8** |
| `make raspberrypi3_defconfig` did not change `.config` | old configuration left in the tree | `make distclean`, then load it explicitly with `BR2_DEFCONFIG=… defconfig`, check with `head .config` |
| "No space left on device" during the build | VM disk too small for a full toolchain + kernel build | enlarged the VMware disk, then `growpart /dev/sda 2` and `resize2fs /dev/sda2` |
| VM freezing during compilation | not enough RAM when a browser was open | gave the VM more RAM and kept only the terminal open while building |
| Authentication error when cloning an external board layer | the layer was not needed | used Buildroot's built-in Raspberry Pi support instead |

## What I learned

How a Linux system is put together from source (toolchain, kernel, device tree, bootloader/firmware, root filesystem), how Buildroot's Kconfig / defconfig system works, and how to debug a long cross-compilation build.

## Author

Nouha Ayed — ISET Bizerte
