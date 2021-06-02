---
title: Nas project - part 1
layout: post
icon: fa-quote-right
---

Since many years, I [backup](https://eoli3n.github.io/archlinux/2020/04/30/backup.html) my personnal and professional data on distant storage at work but the lack of a reliable hosting solution makes me keep important data on my main desktop computer. Then backuping duplicates that data, but it takes up spaces on my 240G SSD.  

I want to setup my self hosted NAS and it would be useful to be able to host some webservers like [Gogs](https://gogs.io/) git server, to keep a duplicate of work repositories and why not a personnal wiki.
The solution would be modular using containers with a reverse proxy in front.

Concerning storage, needs are not so heavy, 3To to 6To would be enough.  
Reading about storage management usually leads to consider [ZFS](https://eoli3n.github.io/archlinux/2020/05/09/system-rollbacks.html) as the best solution.  
It is rock solid, as long as host OS support the module nativly.  
RAID-Z1 needs a minimum of three disks and i will include a parity disk, resulting pool capacity is N-1.

### OS

BSD systems have the best [ZFS support](https://github.com/eoli3n/archiso-zfs), it is included in the kernel.
FreeNAS is great on the paper:  
- it provides a [Web UI](https://www.freenas.org/about/screenshots/)
- based on [FreeBSD](https://www.unixsheikh.com/articles/why-you-should-migrate-everything-from-linux-to-bsd.html)
- it nativly supports [virtualisation](https://www.freenas.org/about/screenshots/),docker containers are run on a [RancherOS](https://rancher.com/docs/os/v1.x/en/) VM  
Sadly, it suffers of a pretty bad reputation concerning its stability, and OS upgrades, confirmed by my tests on a VM.

FreeBSD would be the next gold choice, but Docker is [currently broken](https://wiki.freebsd.org/Docker), and I don't trust the Linux Compatibility Layer. There are jails with [iocage](https://github.com/iocage/iocage) or [bastille](https://github.com/BastilleBSD/bastille) which support templates, but everything needs to be done manually while docker provides a huge images catalog on [hub.docker.com](https://hub.docker.com/).  

In Linux world, Archlinux could be a great choice, it is my main OS and I worked on archlinux install over ZFS. Rolling upgrades are also less stressful than release upgrades but ZFS isn't supported defaultly and it is the primary purpose of the NAS.

The only Linux distro which supports ZFS nativly is Ubuntu since 20.04 LTS ! It has a server version, and I trust Canonical after many years maintaning 800 Ubuntu desktop clients.  
It seems to be the [best compromise](https://www.reddit.com/r/zfs/comments/hd58hv/vanilla_zfs_on_ubuntu_for_nas_server_better_than/), a solid storage management with ZFS, and native possibility of using Docker or Podman containers.  

### Scheme


![server]({{site.baseurl}}/assets/svg/server.svg)

### Hardware

A nice guide gave me lots of directions about hardware choice: [Building a DIY Home Server with FreeNAS](https://www.devroom.io/2020/02/28/building-a-diy-home-server-with-freenas/).

**Case**  
[Fractal Design Node 804 Black Window](https://www.fractal-design.com/products/cases/node/node-804/black/)
A bit expensive, but it has a good design, not so massive and you can setup 8 disks, and a micro ATX motherboard.  

[Node 304](https://www.fractal-design.com/products/cases/node/node-304/Black/) is a good option too, but it is in mini ITX and motherboards are more expensive. Fans are also smaller so it would be a louder and less evolutive config.  

**Motherboard**  
[MSI H310M PRO-M2 PLUS](https://fr.msi.com/Motherboard/H310M-PRO-M2-PLUS.html)  
4xSATA3, 1xM.2, 2xDDR4 2666Mh, Intel 8gen  
It is the cheaper but less evolutive solution.  

[MSI B365M Pro-VDH](https://fr.msi.com/Motherboard/B365M-PRO-VDH)
6xSATA3, 1xM.2, 4xDDR4 2666Mh, Intel 8gen and 9gen  
Middle solution with 2 extra SATA ports and 2 extra RAM slots.  

[Asus PRIME B360M-A](https://www.asus.com/fr/Motherboards-Components/Motherboards/PRIME/PRIME-B360M-A/)  
6xSATA3, 2xM.2, 4xDDR4 2666Mh, Intel 8gen  
A solution which let you use mirroring of the M.2 SSD.  

**CPU**
[Intel Pentium G5400](https://www.intel.fr/content/www/fr/fr/products/processors/pentium/g5400.html)  
2 cores, 4 threads, 3.70 GHz, 4 Mo, HD Graphics, Coffee Lake, 54 Watts  
Cheaper solution, less evolutive if more containers are needed in the future.

[Intel Core i3 8100](https://www.intel.fr/content/www/fr/fr/products/processors/core/i3-processors/i3-8100.html)  
4 cores, 4 threads, 3.60 GHz, 6 Mo, HD Graphics, Coffee Lake, 65 Watts  
2 extra cores which allow Hardware virtualization if needed.  

[Intel Core i3 9100F](https://ark.intel.com/content/www/fr/fr/ark/products/190886/intel-core-i3-9100f-processor-6m-cache-up-to-4-20-ghz.html)
4 coeurs, 4 threads, 3.60 GHz, 6 Mo, Coffee Lake Refresh, 65 Watts  
Cheaper and better, but only for MSI B365M Pro-VDH motherboard.  

**RAM**  
[2x8G DDR4 2666Mh](https://www.materiel.net/produit/201804240051.html)
[1x16G DDR4 2666Mh](https://www.materiel.net/produit/201810080050.html)  

**Storage**
For OS
[120G WD Green SSD M.2](https://shop.westerndigital.com/fr-fr/products/internal-drives/wd-green-sata-ssd#WDS120G2G0A)  
For storage, two choices possible:  
[4x1To WD Blue 5400 RPM](https://shop.westerndigital.com/fr-fr/products/internal-drives/wd-blue-desktop-sata-hdd#WD10EZRZ)  
[4x2To WD Blue 5400 RPM](https://shop.westerndigital.com/fr-fr/products/internal-drives/wd-blue-desktop-sata-hdd#WD20EZRZ)  

**Power supply**  
[Be Quiet! Pure Power 11 CM - 400W - Gold](https://www.bequiet.com/fr/powersupply/1549)  

It misses SATA cables, total for the cheaper solution is ~530€ and the expensive one is ~740€.
