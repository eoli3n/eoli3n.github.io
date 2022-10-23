---
title: Install FreeBSD from Linux
layout: post
icon: fa-freebsd
icon-style: solid
---
Self-hosting rocks... until you need to change place.
I was searching for a cheap VPS to move all my services online.

# VPS or dedicated server ?

A dedicated server is what it is, a physical device, dedicated to your use. You don't share it, that's the benefit, but then you need to ensure that you will not get storage failure for exemple. If it fails, you will wait for the technician to change you HDD. To secure your data, you should then ensure that it is replicated, or get a dedicated server with two disks to soft RAID.

Storage is expensive, cheapest dedicated server offers, usually don't have two disks.

A VPS storage is managed, you get a part of a SAN over SSD, which is much more secure than a single breakable HDD.
I choose to get a [Contabo](https://contabo.com/en/) which offers on the paper a solid solution, a lot more cheaper than others providers:
- 4 vCPU
- 8GB of RAM
- 32TB of outcoming traffic
- 200G of SSD storage
- 7,2€ per month

# Get FreeBSD

I needed for the provider to support FreeBSD. When purchasing the VPS, I didn't found how to get preinstallation with FreeBSD, so I contacted the support to ask how to.

> Please be kindly informed that FreeBSD is not offered as an OS to start with, however, it is present in the list of OSs that you can choose from to reinstall on your existing server. Once the server is provisioned - you will be able to go to your Customer Control Panel and reinstall OS of your server from there. Right now we have FreeBSD 13.1 (64 bit) as an option.

Nice ! So I did and had the bad surprise: The default install from the managed 13.1 image is not root on ZFS.

Let's reask to the Contabo support how to proceed then.

> Please be kindly informed that you are able to purchase Custom Image storage, add iso and install OS from it and then mark this service for a cancellation from your account at my.contabo.com.

Hm, paying 1,20€ to get Freebsd over ZFS ?

# Depenguinator

After asking on irc.libera.chat#freebsd, I was told about a project which could let me prepare a FreeBSD installer from Linux: [depenguinator](https://github.com/allanjude/depenguinator)
It puts the installer on the swap partition, then create a bootloader entry to directly boot on it. Clever !
The original version announcement can be found [here](https://www.daemonology.net/depenguinator/).

Let's reinstall my VPS to a free Linux install first.

![reinstall]({{site.baseurl}}/assets/images/vps/reinstall-debian11.png)

After one minute, let's connect to the vps et clone the project.  
The last commit on the main project is from 2016, so I forked to add my patch before asking for a PR.

```bash
root@vps:~# apt install -y git

root@vps:~# git clone https://github.com/eoli3n/depenguinator
Cloning into 'depenguinator'...
remote: Enumerating objects: 41, done.
remote: Total 41 (delta 0), reused 0 (delta 0), pack-reused 41
Receiving objects: 100% (41/41), 916.43 KiB | 8.18 MiB/s, done.
Resolving deltas: 100% (17/17), done.
```

We need to be sure that the VPS is using GRUB, let's check with [that method](https://unix.stackexchange.com/a/621012)

```bash
echo $((`cat /proc/sys/kernel/bootloader_type`>>4)) 
7
```

The bootloader ID "7" is reffering to GRUB in [that list](https://github.com/torvalds/linux/blob/0adb32858b0bddf4ada5f364a84ed60b196dbcda/Documentation/x86/boot.txt#L376-L393).

There are not instructions in the [README](https://github.com/allanjude/depenguinator/blob/master/README.md) file, a documentation can be found on the [original author website](https://www.daemonology.net/blog/2008-01-29-depenguinator-2.0.html).

Let's install asked dependencies. The script needs bsdtar which is in ``libarchive-tools`` package on Debian 11.

```bash
root@vps:~# apt install -y libarchive-tools libc6-dev zlib1g-dev gcc
```

We now need to configure depenguinator. In the config file, we need to specify how the booted installer system will connect to internet.

```bash
root@vps:~# cd depenguinator

root@vps:~/depenguinator# mv depenguinator.conf.dist depenguinator.conf
```

All informations can be found on the currently running Debian install

```ini
# `hostname` command to get your hostname
hostname="vps"

# `ip route | grep default`
defaultrouter="194.163.128.1"

# `cat /etc/resolv.conf` to get the dns
depenguinator_nameserver="161.97.189.52"

# `ifconfig eth0 | grep -E 'inet |ether'` to get interface ip, netmask and mac
depenguinator_interfaces="external"
depenguinator_mac_external="xx:xx:xx:xx:xx:xx"
depenguinator_ip_external="xxx.xxx.xxx.xxx"
depenguinator_netmask_external="255.255.192.0"
```

Allan Jude version of the script doesn't need an iso path as argument, it directly fetch data on freebsd servers from the release name, so we can now build the installer system.
I also [patched the script](https://github.com/allanjude/depenguinator/commit/3a29a2bba2347e6e7207c3cfc96fe1778637a4e1) to use ``bsdtar`` instead of ``tar``.

```bash
root@vps:~/depenguinator# sh makeimage.sh 13.1-RELEASE ~/.ssh/authorized_keys
```
