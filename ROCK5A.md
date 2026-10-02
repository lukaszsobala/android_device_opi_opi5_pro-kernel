# Radxa ROCK 5A (branch android-17.0-rock5a)

## Files on this branch

| File | What it is |
|---|---|
| `u-boot-rockchip.bin` | Mainline U-Boot v2026.07, `rock5a-rk3588s_defconfig`. Written to sector 64 by `mkimg_gpt.sh`. |
| `rk3588s-rock-5a.dtb` | Built from lukaszsobala/android_kernel_rk_opi, branch `android-17.0-6.18-rock5a`. Replace it with the one from your kernel build. |
| `Image` | **Still the Orange Pi prebuilt (6.18.1, Jan 2026).** Replace it with your build of `android-17.0-6.18-rock5a`. The old Image boots, but has no built-in Ethernet driver, Wi-Fi or Bluetooth for the A8 module. |
| `boot.cmd` / `boot.scr` | Boots from whichever device U-Boot found `boot.scr` on (SD or eMMC). No per-device edits needed. |
| `config.txt` | `fdtfile=rk3588s-rock-5a.dtb`. Add `boot_device=`/`boot_devnum=` only to force a device. U-Boot numbering on ROCK 5A: `mmc 0` = eMMC, `mmc 1` = SD. |

After editing `boot.cmd`, regenerate the script:

    mkimage -A arm64 -O linux -T script -C none -d boot.cmd boot.scr

## How u-boot-rockchip.bin was built

- U-Boot v2026.07 (https://github.com/u-boot/u-boot), `rock5a-rk3588s_defconfig`
- BL31: TF-A v2.15.0, `make PLAT=rk3588 bl31`
- TPL (DDR init): rockchip-linux/rkbin `bin/rk35/rk3588_ddr_lp4_2112MHz_lp5_2400MHz_v1.24.bin`
- No OP-TEE (not needed: the build uses non-secure KeyMint/Gatekeeper)

Build commands:

    export CROSS_COMPILE=aarch64-linux-gnu-
    export BL31=<tfa>/build/rk3588/release/bl31/bl31.elf
    export ROCKCHIP_TPL=<rkbin>/bin/rk35/rk3588_ddr_lp4_2112MHz_lp5_2400MHz_v1.24.bin
    make rock5a-rk3588s_defconfig && make -j$(nproc)

sha256: `a59387bda119a5270453f818c7243f686cdedb8b5b09811c64e040930e041634`

U-Boot tries `mmc1` (SD) before `mmc0` (eMMC), so a bootable SD card takes priority over eMMC.
If the board's SPI NOR flash holds an older bootloader, the boot ROM runs that one first.
Erase the SPI flash, or write a matching U-Boot to it, so the U-Boot from SD/eMMC is the one that runs.
