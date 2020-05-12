---
title: System rollbacks
layout: post
icon: fa-history
---

Backing up an OS slash filesystem as data seems useless today. Since we use highly automated deployment processes, it is safer to re-provision a fresh, healthy, up-to-date system than to restore a file system with potential instabilities, in case of problem. It can be comfortable to historize system states at low level to allow us to rollback quickly. This mechanism should be very simple to use, as an emergency process.

[NixOS generations](https://nixos.wiki/wiki/NixOS#Generations) implemented this at the file level, each file states are stored in an index and the whole system reffers to it. It's easy to make filesets, to rollback to any previous state. I wanted to be able to rollback as easily on Arch Linux. As NixOS paradigm is unique, I needed to use a solution at the block level: rootfs snapshots. I wanted to moved from ``ext4`` over ``LVM`` with thin provisionning, let's give a try to ``BTRFS`` and ``ZFS``.

After a first try with [Grub, BTRFS snapshots and snapper](https://github.com/eoli3n/arch-config/tree/master/ansible/roles/btrfs), I was disappointed: boot a snapshot is not a default feature and is hard to manage and use safely. That's one of reasons which gave me the push to try ``ZFS``, with a FS crash after I killed my virtual machine with the power button...

###  The last word ?

ZFS uses volumes named ``datasets``, each one stores its own configuration.

ZFS [Boot Environments](https://ramsdenj.com/2018/05/29/zedenv-zfs-boot-environment-manager.html) are just dataset clones with some boot management. Datasets embeed their mountpoints, a clone is bootable just by editing ``zfs=`` cmdline var. We just need to specify which dataset to use as bootfs.

[zectl](https://ramsdenj.com/2020/03/18/zectl-zfs-boot-environment-manager-for-linux.html) is a Boot Environment manager which has a ``systemd-boot`` plugin. As systemd-boot can't read ``/boot`` on ZFS, it forces to set a separated one. After [some tweak](https://github.com/johnramsden/zectl/blob/master/docs/plugins/systemdboot.md) arround ``/boot`` location, and [few configurations](https://github.com/eoli3n/arch-config/blob/master/ansible/roles/zfs/systemd-boot-zectl/tasks/main.yml), zectl knows how to manage your separated `/boot`, you can now create your first BE !

### Check your setup

First lets check that ``systemdboot`` plugin is enabled and that ``/efi`` mount and ``/boot`` bind mount are ok.

```bash
$ zectl get
PROPERTY                    VALUE
org.zectl:bootloader        systemdboot
org.zectl.systemdboot:efi   /efi
org.zectl.systemdboot:boot  /boot
org.zectl:bootpool_root
org.zectl:bootpool_prefix

$ ls /efi
dc6d323dfcb146d4b79641866073ba33  EFI  env  loader

$ ls /efi/env/org.zectl-default/
initramfs-linux-lts-fallback.img  initramfs-linux-lts.img  intel-ucode.img  vmlinuz-linux-lts
```

We bind mount ``/boot`` to ``/efi/env/org.zectl-default/``, to let zectl manage it automatically.  
It also needs a ``org.zectl-default.conf`` entry.

```bash
$ cat /efi/loader/entries/
org.zectl-default.conf  recovery.conf

$ cat /efi/loader/entries/org.zectl-default.conf
title           Arch Linux ZFS Default
linux           /env/org.zectl-default/vmlinuz-linux-lts
initrd          /env/org.zectl-default/intel-ucode.img
initrd          /env/org.zectl-default/initramfs-linux-lts.img
options         zfs=zroot/ROOT/default rw
```

### Create a Boot Environment

Creating a BE is simple:

```bash
$ zectl create test

$ ls /efi/loader/entries/
org.zectl-default.conf	org.zectl-test.conf  recovery.conf

$ cat /efi/loader/entries/org.zectl-test.conf
title           Arch Linux ZFS Default
linux           /env/org.zectl-test/vmlinuz-linux-lts
initrd          /env/org.zectl-test/intel-ucode.img
initrd          /env/org.zectl-test/initramfs-linux-lts.img
options         zfs=zroot/ROOT/test rw
```

zectl just:
- Created a clone of currently running dataset
- Created a new bootloader entry which target the new dataset as bootfs
- Backed up current kernel and initramfs in a new directory ``/efi/env/org.zectl-test/``
- Edited ``fstab`` entry in snapshot to know where is located that new ``/boot`` directory

### Break and restore

Let's play a bit

```bash
$ rm -Rf /usr
$ ls
bash: ls : command not found
```
Nice, lets restore the snapshot.

After rebooting and selecting the new ``test`` entry in the bootloader, my system boots well!
zectl think that's still a test boot environment, we need to activate current booted environment:

```bash
$ zectl list
Name     Active  Mountpoint  Creation
default  R       -                 2020-05-09 11:13
test     N       /                 2020-05-09 12:33

$ zectl activate test

$ zectl list
Name     Active  Mountpoint  Creation
default          -                 2020-05-09 11:13
test     NR      /                 2020-05-09 12:33

$ zectl destroy default

$ zectl list
Name  Active  Mountpoint  Creation
test  NR      /                 2020-05-09 12:33
```

From the man page:
> The Active column displays an N on the boot environment currently booted, and a R on the activate boot environment.

Sounds solid, and simple to manage, far than my previous try with BTRFS.  
I started to write a pacman hook to automate the process before each upgrades.  
[https://github.com/eoli3n/zectl-pacman-hook](https://github.com/eoli3n/zectl-pacman-hook)

## My first AUR package
  

*11 May 2020*

``zectl-pacman-hook`` is now distributed through AUR: [https://aur.archlinux.org/packages/zectl-pacman-hook/](https://aur.archlinux.org/packages/zectl-pacman-hook/)  
At each kernel upgrade, a pacman hook triggers a boot environment creation and a rotation. It would help on any problem with the zfs module build.
I opened issues to [add prune feature](https://github.com/johnramsden/zectl/issues/16) to ``zectl``.

```
$ sudo pacman -Syu
:: Synchronizing package databases...
 core is up to date
 extra is up to date
 community is up to date
 archzfs is up to date
 multilib is up to date
:: Starting full system upgrade...
resolving dependencies...
looking for conflicting packages...

Packages (1) linux-lts-5.4.39-1

Total Installed Size:  73.34 MiB
Net Upgrade Size:      -0.01 MiB

:: Proceed with installation? [Y/n] Y
(1/1) checking keys in keyring                     [------------------------] 100%
(1/1) checking package integrity                   [------------------------] 100%
(1/1) loading package files                        [------------------------] 100%
(1/1) checking for file conflicts                  [------------------------] 100%
(1/1) checking available disk space                [------------------------] 100%
:: Running pre-transaction hooks...
(1/3) Create a boot environment
• Destroyed pacmanhook-20200512T154713
• Created pacmanhook-20200512T154826
(2/3) Removing linux initcpios...
(3/3) Remove DKMS modules
:: Processing package changes...
(1/1) upgrading linux-lts                          [------------------------] 100%
:: Running post-transaction hooks...
...
```
