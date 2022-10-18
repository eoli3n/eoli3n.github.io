---
title: ZFS to archiso
layout: post
icon: fa-database
---

After a test of [Nixos implementation of ZFS](https://nixos.wiki/wiki/NixOS_on_ZFS), and the fact that Ubuntu added support for [install on ZFS root](https://wiki.ubuntu.com/FocalFossa/ReleaseNotes#ZFS_0.8.3) support, I was curious about how to use it on Arch Linux.

### Harder than on NixOS

ZFS subvolumes are called ``datasets`` which are stored in ``zpools``.
NixOS [doesn't use datasets as it should](https://nixos.wiki/wiki/NixOS_on_ZFS#Known_issues), it uses classic fstab mounts.
ZFS is designed to be used with its own mount mecanic, that's pretty surprising at first look, I have never seen any FS which doesn't use fstab/crypttab entries.

My [NixOS install scripts](https://github.com/eoli3n/nix-config/tree/master/scripts/install) are not usable as is on Arch Linux.
Let's rewrite it.

### No arms, no chocolate

After searching for the [ZFS article](https://wiki.archlinux.org/index.php/ZFS) on the Arch Linux wiki, ...

>Due to potential legal incompatibilities between CDDL license of ZFS code and GPL of the Linux kernel ([2],CDDL-GPL,ZFS in Linux) - ZFS development is not supported by the kernel.
>
>As a result:
>
>    ZFSonLinux project must keep up with Linux kernel versions. After making stable ZFSonLinux release - Arch ZFS maintainers release them.
>    This situation sometimes locks down the normal rolling update process by unsatisfied dependencies because the new kernel version, proposed by update, is unsupported by ZFSonLinux.

What a nice start, ZFS needs its kernel module, but you need to install it manually from ``Archzfs`` [unofficial user repository](https://wiki.archlinux.org/index.php/unofficial_user_repositories#archzfs).
But there is a huge problem: as Arch Linux iso is released at first of each month embeeding the current kernel, at the first kernel upgrade on community list, zfs modules of Archzfs are recompiled for the latest kernel.
Then the kernel version that embeed your latest archiso image dismatch your zfs module version.

The workaround is to [build your own archiso](https://wiki.archlinux.org/index.php/Install_Arch_Linux_on_ZFS#Embedding_archzfs_into_archiso) that includes that module.
What a deception, I just found a way to use [netboot Arch Linux]({{ site.baseurl }}{% link _posts/2020-04-25-recovery.md %}) as installer or recovery system.

### There's always a way, if you're a committer

Archzfs has a ``zfs-dkms`` package which compile zfs kernel module.
In order to build the module, DKMS needs the ``linux-headers`` package for the running kernel.
Fortunately, [Arch Linux Archive](https://wiki.archlinux.org/index.php/Arch_Linux_Archive#How_to_restore_all_packages_to_a_specific_date) lets you set the mirrorlist to a specific date.

I [wrote a script](https://github.com/eoli3n/archiso-zfs) which automates the whole building process and opened an issue to the maintainer of Archzfs.
It seems that there is an archive mirror for its repository too, it would be easier to uses it, avoiding compilation step.

For now, just use this after booting a standard archiso system to initialize zfs module.

```
curl -s https://raw.githubusercontent.com/eoli3n/archiso-zfs/master/init | bash
```

Wait and see.

### Less than 10 seconds


*06 May 2020*

After some work was done with [ArchZFS maintainer](https://github.com/archzfs/archzfs/issues/337), i changed the script to get precompiled package from ``ArchZFS archives`` repository.
The ZFS module is now easily accessible for Archiso. \o/

![zfs]({{site.baseurl}}/assets/images/archlinux/zfs.png)

[https://github.com/eoli3n/archiso-zfs](https://github.com/eoli3n/archiso-zfs)
