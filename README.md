# ArchLinux on NanoPi R2S

## Install to a micro SD card

1. Zero the beginning of the SD card:

    ```sh
    dd if=/dev/zero of=/dev/sdx bs=1M count=16
    ````

1. Start fdisk to partition the micro SD card:

    ```sh
    fdisk /dev/sdx
    ```

    1. type o.
    1. type p.
    1. type n, then p for primary, 1 for first partition on the drive, 32768 for the first sector.
    1. type w.

1. Create the ext4 filesystem:

    ```sh
    mkfs.ext4 /dev/sdx1
    ```

1. Mount the filesystem:

    ```sh
    mkdir root
    mount /dev/sdx1 root
    ```

1. Download and extract the root filesystem (as root, not via sudo):

    ```sh
    wget http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
    bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C root
    ```

1. extract r2s kernel images.

    ```sh
    bsdtar -xpf kernel-r2s.tar.xz -C root
    ```

1. Unmount the partition:

    ```sh
    umount root
    ```

1. Write bootloader.

    Prebuilt binaries are in bootloader directory.

    ```sh
    dd if=idbloader.bin of=/dev/sdx seek=64 conv=notrunc
    dd if=uboot.img of=/dev/sdx seek=16384 conv=notrunc
    dd if=trust.bin of=/dev/sdx seek=24576 conv=notrunc
    ```

    These files are built from armbian sources.

1. Use the serial console or SSH to the IP address given to the board by your router.

    - Login as the default user alarm with the password alarm.
    - The fault root password is root.

    Make sshd_config to `PermitRootLogin yes` for root login from ssh.

1. Remove useless kernel.

    Don't `pacman -Syu` before replace linux-aarch64.

    ```ssh
    pacman -R linux-aarch64
    ```

    Replace kernel to packaged version.
    Prebuilt packages from https://github.com/sakapoko/linux-r2s/releases 

    ```sh
    pacman -U linux-r2s-5.4.75-1-aarch64.pkg.tar.xz --overwrite '/usr/lib/modules/*','/boot/*'
    pacman -U linux-r2s-headers-5.4.75-1-aarch64.pkg.tar.xz
    ```

1. Initialize the pacman keyring and populate the Arch Linux ARM package signing keys:

    ```sh
    pacman-key --init
    pacman-key --populate archlinuxarm
    ```

1. system up to date

    ```sh
    pacman -Syu
    ```

1. Configuration OS settings.

    localegen, localtime and so on.

## Reference

- kernel from https://github.com/friendlyarm/sd-fuse_rk3328
- bootloader from https://github.com/armbian/build
- install step from https://archlinuxarm.org/platforms/armv8/rockchip/rock64