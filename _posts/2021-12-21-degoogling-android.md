---
title: Degoogling
layout: post
icon: fa-android
icon-style: solid
---
* TOC
{:toc}

As many users, Google trapped me many years ago, with my first Android smartphone. Google services are like cotton candy, sweet, colorful, comfortable, but abuse it and you ruin you health. Before even thinking about privacy, I want to be able to choose what services I run on my smartphone. Android version upgrades and the overlay of the manufacturer android rom is heavy and it makes any mid-range smartphone unusable in less than 2 years.
Another problem is that you can't use Android without a google account.

So I started degoogling, step by step, which was a one year trip...

### Apps

I started by trying to find FOSS alternatives for each apps I use.
[F-droid](https://f-droid.org/) is a application store for FOSS on Android and you can find some alternative on [Degoogle](https://degoogle.jmoore.dev/).

Here's a cool app list :

- [Book Reader](https://f-droid.org/en/packages/com.github.axet.bookreader/) : Simple book reader
- [Catima](https://f-droid.org/en/packages/me.hackerchick.catima/) : Loyalty Card Wallet
- [Etar Calendar](https://f-droid.org/en/packages/ws.xsoh.etar/) : Agenda app which supports CalDav
- [FairEmail](https://f-droid.org/en/packages/eu.faircode.email/) : Fully featured email client. Neat, intuitive user interface. Privacy friendly
- [Infinity](https://f-droid.org/en/packages/ml.docilealligator.infinityforreddit/) : A beautiful, feature-rich Reddit client.
- [LibreOffice Viewer](https://f-droid.org/en/packages/org.documentfoundation.libreoffice/) :
- [Markor](https://f-droid.org/en/packages/net.gsantner.markor/) : Lightweight text editor, Markdown Notes & ToDo.
- [MuPDF](https://f-droid.org/en/packages/com.artifex.mupdf.viewer.app/) : Minimalist PDF viewer
- [NewPipe](https://f-droid.org/en/packages/org.schabi.newpipe/) : Lightweight YouTube frontend
- [OpenBoard](https://f-droid.org/fr/packages/org.dslul.openboard.inputmethod.latin/) : Look and feel of Gboard without the tracking
- [OpenKeychain](https://f-droid.org/en/packages/org.sufficientlysecure.keychain/) : GPG key manager
- [OsmAnd](https://f-droid.org/en/packages/net.osmand.plus/) : Map Viewing & Navigation for Offline and Online OSM Maps
- [Password Store](https://f-droid.org/en/packages/dev.msfjarvis.aps/) : Password Manager
- [Shelter](https://f-droid.org/en/packages/net.typeblog.shelter/) : Provides an isolated space that you can install or clone apps into
- [Simple Notes](https://f-droid.org/en/packages/com.simplemobiletools.notes.pro/) : Notes app with a clean widget
- [Termux](https://f-droid.org/en/packages/com.termux/) : Terminal emulator with packages
- [Voice](https://f-droid.org/en/packages/de.ph1b.audiobook/) : Simple audiobook reader

### Data

Google Drive allows you to access data on the cloud from any device. I used [rclone](https://rclone.org/) to be able to sync my data to a local storage. But Google still owns it too and its security is dependant to my Google account.

[Syncthing](https://syncthing.net/) is a open, trustworthy and decentralized file synchronization. You can install it on any OS, it's easy to configure and fast.

As you can sync any directory, I use it to sync my pictures to my nas and backup it.

![syncthing]({{site.baseurl}}/assets/images/degoogling/syncthing.png)

### Emails, contacts, and agenda

Fashion is to end-to-end encryption services, like [Protonmail](https://protonmail.com/), or [Tutanota](https://tutanota.com).
Both of them are known backdoored, you can share encrypted mails only with users of the same service (if you don't use gpg manually) and on Android, you need to use specific apps to access your synced contacts and agendas.
Some workaround are one the road, but I prefer more open solutions, because I don't need that level of security.

For mail hosting, I chose [runbox.com](runbox.com). The [last version](https://github.com/runbox/runbox7) of the web app si open-source. It is hosted in Norway, which has a respectful privacy legislation, servers are powered with green energy, and it provides caldav sync for contacts and agendas.

I use the f-droid app [Davx5](https://f-droid.org/fr/packages/at.bitfire.davdroid/) as Cal/Card Dav sync client.

### Maps and navigation

FOSS alternative for Google Maps is OsmAnd. I tried to switch for my daily use, and I quickly realized that google maps was not only a simple navigation service. I use it to find opening hours of shops, phone numbers, shop based on a "meta" search... OsmAnd is based on [openstreetmap](https://www.openstreetmap.org/) which is a great service, if you know the full (and well typed) address. Another problem is that route calculation is done on the device, so it doesn't know for the traffic, accidents, etc. To be honest, I think Google Maps as I use it, cannot be replaced for now by a FOSS app, so I would continue to use it.

### OS

The main alternative OS is [LineageOS](https://lineageos.org/). It is shipped defaultly without Google services.
Google services has been reimplemented open-source by the [MicroG](https://github.com/microg) project.

Many apps and features depends on Google services : mainly push notifications with GCM.
Sadly, I still need those services, to be able to use any app from the regular store.
You can check compatibility of apps without Google services with [Plexus](https://plexus.techlore.tech/).

LineageOS is great, but you need to find the right hardware to run it, and the hardware support is community-driven.
The flash process could be tricky, with a significant risk to brick your device.

Here comes the funny part : The only choice that allows you to unlock the bootloader and install a custom ROM without too much effort/risk are Google phones because those are designed for developpers !

Choosing a Google smartphone like a Pixel or a Nexus opens your choices to [GrapheneOS](https://grapheneos.org/) or [CalyxOS](https://calyxos.org/). Both of them as based on [Android Open Source Project](https://source.android.com/), which is Android without the Google commercial overlay.

I will not compare those 2 here, but I chose CalyxOS for those features :
- Project driven by [Calyx Institute](https://calyxinstitute.org/)
- MicroG included (and can be disabled)
- Focused on privacy
- Security with [Datura](https://calyxos.org/docs/tech/datura-details/) per app firewall
- Auto backup with [Seedvault](https://calyxinstitute.org/projects/seedvault-encrypted-backup-for-android)

To go further : [privacyguides.org/android](https://privacyguides.org/android/#aosp-derivatives)

### Hardware

I just switch to a 5G subscription, so the choice is pretty limited.
Despite the fact that the Google Pixel 6 is out since 4 month, I went for a Google Pixel 4a 5g, because it's cheaper, the screen is smaller and it stills have a minijack :) I also wanted to be able to directly install CalyxOs without beeing stuck on the Google Pixel ROM. When I write those lines, CalyxOS doesn't support Google Pixel 6 still.

### CalyxOS installation

As said, a benefit for using a developer phone is that the flash procedure is pretty simple, and stressless.
You just need to download the flasher binary and the OS archive.
See [Install on a Pixel 4a (5G)](https://calyxos.org/install/devices/bramble/linux/).

On void linux, I needed to install some packages.

```bash
$ xbps-install -S android-tools android-udev-rules
```

At first, ``device-flasher`` didn't detect the smartphone.
I got help on IRC ``libera.chat #calyxos``, the community has been nice and helpful.
They adviced me to [boot it in fastboot mode](https://calyxos.org/install/fastboot/) and to run ``device-flasher`` with sudo. It worked like a charm, 5 min and the phone rebooter under CalyxOS.

### Security configurations

#### Aurora store

After the first boot, a configuration menu let you enable ``microG`` and it doesn't ask you to attach a google account. To be able to install Play Store apps without account, it uses [Aurora Store](https://aurora-store.fr.uptodown.com/android). It supports anonymous apk downloads and installations, and can silently auto upgrade for apps in background.

#### Shelter isolation

Those untrustable applications doesn't need for the most part to be able to access my data, or network.
[Shelter](https://f-droid.org/fr/packages/net.typeblog.shelter/) use the Android work profile feature to allow you to isolate apps from your data, disabling the ability to use or leak you contacts, for exemple.

To configure it, install ``shelter``, and use it to activate your work profile. Then you can clone ``Aurora`` to the work profile, and use it from here to install untrusted apps.

![shelter]({{site.baseurl}}/assets/images/degoogling/shelter_clone_aurora.png){: width="300" } ![work]({{site.baseurl}}/assets/images/degoogling/work_profile.png){: width="300" }

#### Datura firewall

To isolate an app from network with ``datura``, start the app ``firewall`` from the main profile.

![firewall]({{site.baseurl}}/assets/images/degoogling/datura_firewall.png){: width="300" }

#### SeedVault backups

Start ``Backup`` app and configure it to backup apps and its configurations on the local storage.
Encrypted backups will be stored in ``~/.SeedVaultAndroidBackup``.
Then I use ``Syncthing`` to spread it on my nas which snapshots it on ZFS.

![backup]({{site.baseurl}}/assets/images/degoogling/backup.png){: width="300" } ![backup_syncthing]({{site.baseurl}}/assets/images/degoogling/backup_syncthing.png){: width="300" }

### A good move

For now, I didn't face any issue, let's see in daily use.
I still have many things to test :
- CalyxOS provides two free VPNs by default, [CalyxVPN](https://calyxinstitute.org/projects/digital-services/vpn) and [Rise Up VPN](https://calyxos.org/docs/guide/apps/riseup-vpn/).
- A default offline map app : [Organic Maps](https://f-droid.org/fr/packages/app.organicmaps/)
- How [location privacy](https://calyxos.org/docs/guide/security/location/) works

Over The Air upgrade to Android 12 is coming soon.

For sure, it is a one way trip, may CalyxOS have a long life !
