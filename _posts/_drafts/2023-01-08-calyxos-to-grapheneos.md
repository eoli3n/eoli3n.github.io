---
title: 
layout: post
icon: fa-android
icon-style: solid
---
* TOC
{:toc}

One year ago, I started to [improve my smartphone software mastery]({{ site.baseurl }}{% link _posts/2021-12-21-degoogling-android.md %}).
I wanted a well hardware supported OS, with more security features than Android stock, and not any usage limitation, while degoogling the most of my usages. At this time, CalyxOS seemed to me a reasonable solution, easier to use and with less limitations than GrapheneOS.

The only really problematic problem I encoutered with CalyxOS is the OTA upgrades management. When a new OTA upgrade is downloaded, you can't trigger a phone reboot without triggering the upgrade.

Privacy guide [removed CalyxOS](https://github.com/privacyguides/privacyguides.org/pull/1518), from its [OS recommendation section](https://www.privacyguides.org/en/tools/#operating-systems) and explains it in a [blog post](https://blog.privacyguides.org/2022/04/21/grapheneos-or-calyxos/).

The section now recommends only [DivestOS](https://divestos.org/) or [GrapheneOS](https://grapheneos.org/).

### DivestOS

DivestOS is a fork of LineageOS, that I excluded because installers are community maintained. The aim is not only Google Pixel phones, then verified boot is not available for all hardware.

For a Pixel phone, the recommendation remains *GrapheneOS*.

> We still recommend GrapheneOS depending on your device's compatibility. For other devices, DivestOS is a good alternative.

### GrapheneOS

Main reasons to choose GrapheneOS over CalyxOS are:

- [Higher release frequency](https://blog.privacyguides.org/2022/04/21/grapheneos-or-calyxos/#update-frequency) and stock security feature integration
- [Sandboxed Google Play](https://blog.privacyguides.org/2022/04/21/grapheneos-or-calyxos/#sandboxed-google-play-vs-privileged-microg) vs Privileged microG
- 32 [user profiles]() instead of just one isolated work profile
- [Better hardening](https://blog.privacyguides.org/2022/04/21/grapheneos-or-calyxos/#additional-hardening)

### Applications

On GrapheneOS, applications are run in a [Sandboxed Google Play instead of privileged microG](https://blog.privacyguides.org/2022/04/21/grapheneos-or-calyxos/#sandboxed-google-play-vs-privileged-microg).
It increases security and compatibility, because Google Play Services are the real one, not a reimplementation and the sandboxing let you control their permissions.

Lets take a look on options when it comes to install an application.

#### GrapheneOS Apps repository

GrapheneOS comes with its [own application repository](https://github.com/GrapheneOS/Apps) which aim to provide only a limited list of high-quality apps. Default apps of the OS are managed this way.

#### F-droid

I discover that [F-droid is not recommended](https://www.privacyguides.org/en/android/#f-droid) to install android apps. There are three main problems :  

- [The repository signs the packages itself](https://privsec.dev/posts/android/f-droid-security-issues/#1-the-trusted-party-problem), instead of the dev, so you need to fully trust F-droid.
- Because F-droid builds tools itself, and because the process automation can be tricky, it leads to [slow and irregular updates](https://privsec.dev/posts/android/f-droid-security-issues/#2-slow-and-irregular-updates).
- The default f-droid client use a [out of date SDK](https://privsec.dev/posts/android/f-droid-security-issues/#3-low-target-api-level-sdk-for-client--apps).

The packages signing problem can only be solved with reproducible build, but not all developers are using it.  

If you need to install a F-droid package anyway, [Neo store](https://github.com/NeoApplications/Neo-Store/) is a F-Droid client "with modern UI and an arsenal of extra features", which uses latest SDK.  

As a long term solution, keep an eye on [Accrescent](https://accrescent.app/) which try to solve a lot of F-droid problems.  

On GrapheneOS, you would prefer a Google Play app version in front of the F-droid one !

#### Google Play apps

There is two way to manage Google Play applications.  

Google Play Store provides unattended upgrades, but needs a google account.
[Aurora Store](https://gitlab.com/AuroraOSS/AuroraStore) applications needs to be updated manually, but apps can be installed anonymously.  

Deal with that.

### Hardware support

CalyxOS [supports my hardware](https://calyxos.org/docs/guide/device-support/) until **January 2025**.

GrapheneOS [drops support](https://grapheneos.org/faq#device-lifetime) when Google stops hardware support. The device then get extended support until the next Android version.
The support of the Pixel 4a 5g stops at November 2023.

[Next Android release](https://developer.android.com/about/versions/14/get) planned to support Pixel 4a 5g.
The preview is available since [February 2023](https://en.wikipedia.org/wiki/Android_version_history), and [the timeline plan the stable version)(https://developer.android.com/about/versions/14/overview) for August 2023.

We can then expect to wait one year before the next stable release which leads to a supposed support date until ~ **August 2024**.

GrapheneOS will force me to renew my hardware at least 6 months earlier.

### Applications isolation

#### Network and censors

There is a [per app firewall](https://grapheneos.org/faq#firewall), when an app use network for the first time, you get asked if you want to deny it.
Same thing for the sensors.

#### Storage and contact

On CalyxOS, I used Shelter to create a work profile. GrapheneOS improved Android User profiles, which is a more secured way to isolate applications. Problem with those is that [User profiles need to be manually switched](https://hub.libranet.de/wiki/and-priv-sec/wiki/user-profiles) where work profile share the same home, you can mix app shortcut from main and work profile on your home screen.

Some alternatives to Shelter are :

- https://gitlab.com/secure-system/Insular
- https://island.oasisfeng.com/

You can find a comparison table of the three solutions on [the Insular Faq](https://secure-system.gitlab.io/Insular/faq.html#Insularcompare-with-island-and-shelter). As shelter is the only one which can "Block Contacts Searching", I would stick to it.

> But do I even need profile isolation on GrapheneOS ?

On GrapheneOS, you can deny storage and contact permission manually for an app.  
You can also use [storage scopes](https://grapheneos.org/usage#storage-access).

Storage scope can be enabled for an app, to ensure that it will only access files created by itself.

### Others features

Backup tool is [SeedVault](https://discuss.grapheneos.org/d/15-how-do-i-backup-my-phone), as in CalyxOS.
...
...
...


### OS comparison


### TOTEST

- Auditors
- Storage scope

### TODO upgrade process

Backup app datas : 
Flash Stock android first : https://flash.android.com/welcome?continue=%2Fback-to-public
Flash GrapheneOS then : https://grapheneos.org/install/
