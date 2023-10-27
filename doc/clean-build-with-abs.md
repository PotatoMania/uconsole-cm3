# Clean build with Arch Build System

Objectives:

- on a x86_64 ArchLinux host
- in a aarch64 chroot
- build with ABS
    - follow [Building_in_a_clean_chroot](https://wiki.archlinux.org/title/DeveloperWiki:Building_in_a_clean_chroot)

In practice, I cannot get it run efficiently enough.

## Setup rootfs

First write a pacman config for initialization(this WILL become the config in rootfs):

```ini
# pacman-aarch64.conf
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

# [aur]
# Server = http://mirror.archlinuxarm.org/\$arch/core
```

Then create the chroot.

```bash
# use -s to skip setarch
mkarchroot -s -C pacman-aarch64.conf /CHROOT/a64/root base-devel
```

Note: It __MUST__ be `some_parent_folder/root`. `root` is hardcoded in some other tools. `some_parent_folder` is the workspace for other tools.

## Mask setarch

`makechrootpkg` use `setarch` to spawn building command. But this will break when it's desired to work in a aarch64 chroot. `setarch` does not support aarch64 on x86_64.

Instead, the binary can be wrapped to execute the original command directly. Place the script below to `$HOME/.local/bin` and add it at the very begining of `PATH`.

```bash
#!/bin/sh
# setarch wrapper

if ! /usr/bin/setarch "$@"
then
        echo "setarch failed, but we decide to continue"
        shift # remove arch param(eg. aarch64)
        echo "Using command: $@"
        "$@"
fi
```

## Config qemu-user-static

Note that using `sudo` is not feasible now in chroot. The config for qemu-user-static need modification.

Copy the default config:

```bash
cp /usr/lib/binfmt.d/qemu-aarch64-static.conf /etc/binfmt.d/
```

Original content:

```
:qemu-aarch64:M::\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xb7\x00:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/usr/bin/qemu-aarch64-static:FP
```

After modification:

```
:qemu-aarch64:M::\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xb7\x00:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/usr/bin/qemu-aarch64-static:FPC
```

The flags changed from `FP` to `FPC`.

More details on [Wikipedia](https://en.wikipedia.org/wiki/Binfmt_misc).

A reboot or reload of the kernel module is required to make the modification take effect.

## Build the package

Change pwd to where PKGBUILD sits. Run the following command:

```bash
makechrootpkg -c -r /CHROOT/a64 -- -s -C -f
```

`/CHROOT/a64/root` is where the rootfs generated. See [Setup rootfs](#setup-rootfs).

## Tweak: makepkg.conf

Enable Distcc and parallel build for better performance.
