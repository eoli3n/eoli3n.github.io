---
title: Nas project
layout: post
icon: fa-quote-right
---

Since many years, I [backup](https://eoli3n.github.io/archlinux/2020/04/30/backup.html) my personnal and professional datas on distant storage at work but the lack of a reliable hosting solution makes me keep important datas on my main desktop. Then backuping my desktop duplicates that datas, but it takes place on my 240G SSD.  

I want to setup my self hosted NAS and it would be useful to be able to host some webservers like [Gogs](https://gogs.io/) git server, to keep a duplicate of work repositories and why not a personnal wiki.
The solution would be modular using containers with a reverse proxy in front.

Concerning storage, needs are not so heavy, 3To to 6To would be enough.

### OS Choice

Reading about storage management usually leads to consider [ZFS](https://eoli3n.github.io/archlinux/2020/05/09/system-rollbacks.html) as the best solution.  
It is rock solid, as far as host OS support the module nativly.  

BSD systems have the best [ZFS support](https://github.com/eoli3n/archiso-zfs), it is included in the kernel.
FreeNAS is great on the paper:  
- provides a [Web UI](https://www.freenas.org/about/screenshots/)
- based on [FreeBSD](https://www.unixsheikh.com/articles/why-you-should-migrate-everything-from-linux-to-bsd.html)
- nativly support [virtualisation](https://www.freenas.org/about/screenshots/) as docker containers run on [RancherOS](https://rancher.com/docs/os/v1.x/en/)
Sadly, it suffers of a pretty bad reputation concerning its stability, and hard OS upgrades, confirmed by my tests on a VM.

FreeBSD would be the next Gold choice, but Docker is [currently broken](https://wiki.freebsd.org/Docker), and I don't trust the Linux Compatibility Layer. There are jails with [iocage](https://github.com/iocage/iocage) or [bastille](https://github.com/BastilleBSD/bastille) which support templates, but everything needs to be done manually while docker provides a huge images catalog on [hub.docker.com](https://hub.docker.com/).  

In Linux world, Archlinux could be a great choice, it is my main OS and I worked on ZFS archlinux install. Rolling upgrades are also less stressful than release upgrades but ZFS isn't supported defaultly and its the primary purpose of the NAS.

The only Linux distro which supports ZFS nativly is Ubuntu since 20.04 LTS ! It has a server version, and I trust Caninical after many years maintaning 800 Ubuntu desktop clients.  
It seems to be the [best of two worlds](https://www.reddit.com/r/zfs/comments/hd58hv/vanilla_zfs_on_ubuntu_for_nas_server_better_than/), a solid storage management with ZFS, and native possibility of using Docker or Podman containers.  

### Hardware

A nice guide gave me lots of directions about hardware choice: [Building a DIY Home Server with FreeNAS](https://www.devroom.io/2020/02/28/building-a-diy-home-server-with-freenas/).

##### Case  
[Fractal Design Node 804 Black Window](https://www.fractal-design.com/products/cases/node/node-804/black/)
A bit expensive, but it has a good design, not so massive and you can setup 8 disks, and a micro ATX motherboard.  

[Node 304](https://www.fractal-design.com/products/cases/node/node-304/Black/) is a good option too, but it is in mini ITX and motherboards are more expensive. Fans are also smaller so it would be a louder and less evolutive config.  

**Motherboard**  
[MSI H310M PRO-M2 PLUS](https://fr.msi.com/Motherboard/H310M-PRO-M2-PLUS.html)  
4xSATA3, 1xM.2, 2xDDR4 2666Mh  
It is the cheaper and less evolutive solution.  

[Asus PRIME B360M-A](https://www.asus.com/fr/Motherboards-Components/Motherboards/PRIME/PRIME-B360M-A/)  
6xSATA3, 2xM.2, 4xDDR4 2666Mh  
A solution which let you use mirroring of the M.2 SSD, 2 extra SATA ports and 2 extra RAM slots.  

**CPU**
[Intel Pentium G5400](https://www.intel.fr/content/www/fr/fr/products/processors/pentium/g5400.html)  
2 cores, 4 threads, 3.70 GHz, 4 Mo, HD Graphics, Coffee Lake, 54 Watts  
Cheaper solution, less evolutive if more containers are needed in the future.

[Intel Core i3 8100](https://www.intel.fr/content/www/fr/fr/products/processors/core/i3-processors/i3-8100.html)  
4 cores, 4 threads, 3.60 GHz, 6 Mo, HD Graphics, Coffee Lake, 65 Watts  
2 extra cores which allow Hardware virtualization if needed.  

**RAM**  
[2x8G DDR4 2666Mh](https://www.materiel.net/produit/201804240051.html) or [1x16G DDR4 2666Mh](https://www.materiel.net/produit/201810080050.html)  

**Storage**
For OS
[120G WD Green SSD M.2](https://shop.westerndigital.com/fr-fr/products/internal-drives/wd-green-sata-ssd#WDS120G2G0A)  
For storage, two choices possible:  
[4x1To WD Blue 5400 RPM](https://shop.westerndigital.com/fr-fr/products/internal-drives/wd-blue-desktop-sata-hdd#WD10EZRZ)  
[4x2To WD Blue 5400 RPM](https://shop.westerndigital.com/fr-fr/products/internal-drives/wd-blue-desktop-sata-hdd#WD20EZRZ)  

**Power supply**  
[Be Quiet! Pure Power 11 CM - 400W - Gold](https://www.bequiet.com/fr/powersupply/1549)  

**Thermal paste**
[Noctua NT-H1](https://noctua.at/fr/products/thermal-grease/nt-h1-3-5g)

It misses SATA cables, total for the cheaper solution is ~540€ and the expensive one is ~750€.
