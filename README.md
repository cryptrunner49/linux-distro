# linux-distro

Building a linux distro using the linux kernel v6.13 and busybox in debian.

## Cloning the repo
```
git clone https://github.com/cryptrunner49/linux-distro
cd linux-distro
git submodule update --init --recursive --depth 1
```

## Installing system requirements
```
sudo apt update
sudo apt install gcc git g++ make cmake pkg-config \
curl bc binutils bison dwarves flex gnupg2 gzip \
libelf-dev libncurses5-dev libssl-dev openssl pahole \
perl-base rsync tar xz-utils bzip2 vim \
libncurses-dev cpio syslinux dosfstools
```

## Building the Linux kernel
```
cp .linux-config linux/.config
cd linux
make
mkdir ../distro
cp arch/x86/boot/bzImage ../distro/
```

## Building BusyBox
```
cd ..
cp .busybox-config busybox/.config
cp .kconfig.d busybox/.kconfig.d
cp .kernelrelease busybox/.kernelrelease
cd busybox
make
mkdir ../distro/initramfs
make CONFIG_PREFIX=../distro/initramfs install
cd ..
cp init distro/initramfs/init
```
## Copying the script to start the shell
```
cd distro/initramfs
chmod +x init
```

## Creating an archive of all files with a format accepted by the kernel
```
find . | cpio -o -H newc > ../init.cpio
```

## Creating the boot file (using the fat filesystem)
```
cd ..
dd if=/dev/zero of=boot bs=1M count=50
mkfs -t fat boot
syslinux boot
```

## Copying the kernel and initramfs into boot
```
mkdir system
sudo mount boot system
sudo cp bzImage init.cpio system/
sudo umount system
```

## Using qemu to run the system
```
qemu-system-x86_64 boot
```

## Run this command inside the qemu vm to boot the system
```
/bzImage -initrd=init.cpio
```
