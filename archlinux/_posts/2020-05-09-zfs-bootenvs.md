---
title: ZFS Boot Environments
layout: post
icon: fa-history
---

Since I tested [NixOS generations](https://nixos.wiki/wiki/NixOS#Generations), i wanted to be able to rollback as easily on Arch Linux. After a first try with Grub, BTRFS snapshots and snapper, I was disappointed: that's not a default feature and is hard to manage. That's one of reasons which gave me the push to try ZFS.

ZFS [Boot Environments](https://ramsdenj.com/2018/05/29/zedenv-zfs-boot-environment-manager.html) are just dataset clones with some boot management. As ZFS embeed all mount options, a clone is bootable just by editing ``zfs=`` cmdline var. We just need to specify which dataset to use as bootfs.

[zectl](https://ramsdenj.com/2020/03/18/zectl-zfs-boot-environment-manager-for-linux.html) is a Boot Environment manager which has a ``systemd-boot`` plugin. It is really simple: after [some tweak](https://github.com/johnramsden/zectl/blob/master/docs/plugins/systemdboot.md) arround ``boot`` location, and [few configurations](https://github.com/eoli3n/arch-config/blob/master/ansible/roles/zfs/systemd-boot-zectl/tasks/main.yml), you can create your first BE !

### Check your setup

First lets check that ``systemdboot`` plugin is enabled and that ``/efi`` mount and ``/boot`` bind mount are ok.

```
[root@osz ~]# zectl get
PROPERTY                    VALUE
org.zectl:bootloader        systemdboot
org.zectl.systemdboot:efi   /efi
org.zectl.systemdboot:boot  /boot
org.zectl:bootpool_root
org.zectl:bootpool_prefix

[root@osz ~]# ls /efi
dc6d323dfcb146d4b79641866073ba33  EFI  env  loader

[root@osz ~]# ls /efi/env/org.zectl-default/
initramfs-linux-lts-fallback.img  initramfs-linux-lts.img  intel-ucode.img  vmlinuz-linux-lts
```

We bind mount ``/boot`` to ``/efi/env/org.zectl-default/``, to let zectl manage it automatically.  
It also needs a ``org.zectl-default.conf`` entry.

```
[root@osz ~]# cat /efi/loader/entries/
org.zectl-default.conf  recovery.conf

[root@osz ~]# cat /efi/loader/entries/org.zectl-default.conf
title           Arch Linux ZFS Default
linux           /env/org.zectl-default/vmlinuz-linux-lts
initrd          /env/org.zectl-default/intel-ucode.img
initrd          /env/org.zectl-default/initramfs-linux-lts.img
options         zfs=zroot/ROOT/default rw
```

### Create a Boot Environment

Creating a BE is simple:

```
[root@osz ~]# zectl create test

[root@osz ~]# ls /efi/loader/entries/
org.zectl-default.conf	org.zectl-test.conf  recovery.conf
[root@osz ~]# cat /efi/loader/entries/org.zectl-test.conf
title           Arch Linux ZFS Default
linux           /env/org.zectl-test/vmlinuz-linux-lts
initrd          /env/org.zectl-test/intel-ucode.img
initrd          /env/org.zectl-test/initramfs-linux-lts.img
options         zfs=zroot/ROOT/test rw
```

zectl just:
- Created a new bootloader entry which target the new dataset as bootfs
- Backed up current kernel and initramfs in a new directory ``/efi/env/org.zectl-test/``
- Edited ``fstab`` entry in snapshot to know where is located that new ``/boot`` directory

### Break and restore

Let's play a bit

```
[root@osz ~]# rm -Rf /usr
[root@osz ~]# ls
bash: ls : command not found
```
Nice, lets restore the snapshot.

After rebooting and selecting the new ``test`` entry in the bootloader, my system boots well!
zectl think that's still a test boot environment, we need to activate current booted environment:

```
[root@osz ~]# zectl list
Name     Active  Mountpoint  Creation
default  R       -                 2020-05-09 11:13
test     N       /                 2020-05-09 12:33
[root@osz ~]# zectl activate test
[root@osz ~]# zectl list
Name     Active  Mountpoint  Creation
default          -                 2020-05-09 11:13
test     NR      /                 2020-05-09 12:33
[root@osz ~]# zectl destroy default
[root@osz ~]# zectl list
Name  Active  Mountpoint  Creation
test  NR      /                 2020-05-09 12:33
```

Sounds solid, and simple to manage, far than my previous try with BTRFS.
I will probably write a pacman hook to automate the process before each upgrades.
