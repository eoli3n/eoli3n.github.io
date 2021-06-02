---
title: Enter the Void
layout: post
icon: fa-exchange-alt
---

After few years on Arch Linux, I was curious to try a new OS. My needs are increasingly less years after years, I focus on features, and try to keep my system as light and rock solid as possible.
I heard about [Void Linux](https://voidlinux.org/), a rolling release distribution without Systemd.

### No more systemd, use the system D

The Void Linux [documentation](https://docs.voidlinux.org/) goes to essentials, the wiki is deprecated, but you get everything you need to make your system run perfectly.
A system without systemd is a paradigm that I forgot, it makes me rediscover how many things it manages, sadly.

Without systemd, no systemd-journald, no systemd-resolved, no systemd-networkd, no systemd-boot, no systemd-logind, no systemd-timer...
Documentation gives lighter alternatives, known for some, for all of those components. Two important ones are:

### Init and services
It defaultly uses [runit](http://smarden.org/runit/).
I don't miss systemd services, runit ones are simple shell scripts, back to basics.
Per user services are possible. To enable a service, you only need to symlink service directory ``/etc/sv/$service`` to ``/var/service/$service``.
Each directory contains a ``run`` shell script.

```bash
$ sudo sv start vpn
ok: run: vpn: (pid 2686) 0s, normally down

$ sudo sv status vpn
run: vpn: (pid 2686) 5s, normally down

$ sudo sv stop vpn
ok: down: vpn: 1s

$ ls /var/service
acpid        agetty-tty3  agetty-tty6  dbus         libvirtd       seatd         udevd      vpn
agetty-tty1  agetty-tty4  chronyd      dhcpcd-eth0  nanoklogd      socklog-unix  virtlockd
agetty-tty2  agetty-tty5  crond        iwd          runsvdir-user  tlp           virtlogd

$ cat /var/service/libvirtd/run 
#!/bin/sh
sv check dbus >/dev/null || exit 1
[ -f ./conf ] && . ./conf
exec libvirtd $OPTS 2>&1
```

### Logs
[Socklog](http://smarden.org/socklog/) is made by the same dev as runit and manage logs as plain text files, not binary files which need a specific tool to be queried...

```bash  
$ ls /var/log/socklog/
cron  daemon  debug  errors  everything  kernel  lpr  mail  messages  remote-udp  secure  tty12  user  xbps
  
$ grep dnsmasq /var/log/socklog/everything/current
2021-05-20T20:19:16.77340 daemon.info: May 20 22:19:16 dnsmasq[2141]: reading /etc/resolv.conf
2021-05-20T20:19:16.77344 daemon.info: May 20 22:19:16 dnsmasq[2141]: using nameserver 1.1.1.1#53
2021-05-20T20:19:16.77345 daemon.info: May 20 22:19:16 dnsmasq[2141]: using nameserver 9.9.9.9#53
2021-05-20T20:19:16.77346 daemon.info: May 20 22:19:16 dnsmasq[2141]: using nameserver 192.168.0.254#53
```

### What about the filesystem ?

Void Linux community developed a powerful tool, to let you manage ZFS root system rollbacks far better than [zectl on Arch Linux]({{ site.baseurl }}{% link _posts/2020-05-09-system-rollbacks.md %}) does.
[zfsbootmenu](https://zfsbootmenu.org/) uses dracut to generate a kernel/initramfs couple with zfs module packed into an ``efi`` file. It lets you point directly to it with ``efibootmgr`` which removes the need to use a bootloader.
Booting this gives you a nice menu to manage your ZFS pool, you can create clone from a dataset snapshot, chroot a dataset, etc...
To boot the OS, ``kexec`` is used to load the kernel from the chosen zfs dataset.

### An OS is nothing without its packages

The first step to know if switching could be a good idea was to check if my applications was already packaged.
On Void Linux, there is no AUR-like, all packages are validated, this is great as my AUR list on Arch Linux was growing more and more.
99% of my needs was already packaged, and I used the missing one to learn how to package as [xbps packages](https://github.com/void-linux/void-packages). I sweated a bit trying to package ``x2go`` client and server packages, but the community helped me a lot on IRC and Github, to finish my first [Void Linux Pull Requests](https://github.com/void-linux/void-packages/pulls/eoli3n).

For packages with licensing problem, I use flatpak packages.

### Finish me...

On Arch Linux, defaultly, you cannot upgrade your kernel and load a new kernel module without rebooting, the ``modules`` directory is managed by the package, so the directory is cleaned when upgrading. The workaround exists as an AUR package, [kernel modules hook](https://aur.archlinux.org/packages/kernel-modules-hook/). Void Linux is again smoother.

```bash
$ sudo rmmod kvm_intel

$ sudo xbps-install -Su

Name              Action    Version           New version            Download size
linux5.11         update    5.11.21_1         5.11.22_1              112MB 
[...]

$ sudo modprobe kvm_intel && echo ":)"
:)
```

Partial updates are possible, on Arch Linux ``pacman -Sy $package`` is forbidden if you don't want to break your system.

I ported my [arch-config](https://github.com/eoli4n/arch-config) repository to [void-config](https://github.com/eoli3n/void-config) with automated install scripts and an ansible playbook to configure the full system.
Flatpak and xbps ansible modules exist, no need to deal with AUR run as user and a sudo tweak to be able to automate package installation with ansible anymore.

### Any negative point ?

The only possible improvment is the lack of ``xbps`` hooks to be able to trigger a post flatpak upgrade or a pre ZFS snapshot. [An issue](https://github.com/void-linux/xbps/issues/304) already exists about that point.
