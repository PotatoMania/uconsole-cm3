# Create a clean AArch64 chroot for installation

Objectives:

- on a x86_64 ArchLinux host
- install a AArch64 ArchLinux environment
    - that is to create a clean chroot, for a new installation, not for ABS
- from scratch

## Steps

Install necessary programs:

```bash
pacman -Sy --noconfirm qemu-user-static qemu-user-static-binfmt arch-install-scripts
```

Write a temporary config for pacman. This will __NOT__ become the config in the chroot.

```ini
# pacman-aarch64-stage0.conf
# bootstrapping configuration

[options]
Architecture = aarch64
ParallelDownloads = 5
SigLevel = Never

[core]
Server = http://mirror.archlinuxarm.org/\$arch/core

[extra]
Server = http://mirror.archlinuxarm.org/\$arch/core

[alarm]
Server = http://mirror.archlinuxarm.org/\$arch/core

[aur]
Server = http://mirror.archlinuxarm.org/\$arch/core
```

Then tell pacstrap to use it for the new installation:

```bash
pacman -KM -C [YOUR_TEMP_PACMAN_CONFIG] [NEW_ROOT] base [YOUR_PACKAGES]
```
