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
- [A lot of features](https://grapheneos.org/features) added on top of AOSP.
- [32 user profiles](https://blog.privacyguides.org/2022/04/21/grapheneos-or-calyxos/#profiles) instead of just one isolated work profile
- [Better hardening](https://blog.privacyguides.org/2022/04/21/grapheneos-or-calyxos/#additional-hardening)

### Sandboxed Google Play ?

On GrapheneOS, applications are run in a [Sandboxed Google Play instead of privileged microG](https://blog.privacyguides.org/2022/04/21/grapheneos-or-calyxos/#sandboxed-google-play-vs-privileged-microg).

> But first, what are Google Play Services ?

[Google Play Services](https://developers.google.com/android?hl=en) is a closed-source application which provide a specific layer to provide easy access to Google features. Some examples:

- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging?hl=en): push notifications
- [Google play protect](https://developers.google.com/android/play-protect?hl=en): anti malware protection
- Single sign-on
- Location

> Wasn't microG a good idea ?

[Microg](https://microg.org/) is an opensource reimplementation of Google Play Services. The aim is to provide an alternative way to run apps. Not all features are reimplemented, it has a [lot of limitations](https://github.com/microg/GmsCore/wiki/Implementation-Status).

> What is the benefit of unprivileged Play Services over microG ?

MicroG runs in a privileged way, so in front of unprivileged Google Play Services, it has less interest.
Google Play Services is not included on GrapheneOS, but you can install it from the GrapheneOS Apps repository.  

> How to know if I will need it ?

You should ask yourself two questions:  
Do I need to log in with my google account ?
Do I use an application which use Google push notification service ?  

You can use [community maintained list](https://plexus.techlore.tech/) to check if an app have a good ungoogled phone support.  

An alternative to Google push notification is [UnifiedPush](https://unifiedpush.org/), but the app needs to explicitly support it.

### Applications repositories

Lets take a look on options when it comes to install an application.

#### GrapheneOS Apps repository

GrapheneOS comes with its [own application repository](https://github.com/GrapheneOS/Apps) which aim to provide only a limited list of high-quality apps. Default apps of the OS are managed this way.

#### F-droid

I discover that [F-droid is not recommended](https://www.privacyguides.org/en/android/#f-droid) to install android apps. There are three main problems :  

- [The repository signs the packages itself](https://privsec.dev/posts/android/f-droid-security-issues/#1-the-trusted-party-problem), instead of the dev, so you need to fully trust F-droid.
- Because F-droid builds tools itself, and because the process automation can be tricky, it leads to [slow and irregular updates](https://privsec.dev/posts/android/f-droid-security-issues/#2-slow-and-irregular-updates).
- The default f-droid client use a [out of date SDK](https://privsec.dev/posts/android/f-droid-security-issues/#3-low-target-api-level-sdk-for-client--apps).

The packages signing problem can only be solved with [reproducible build](https://reproducible-builds.org/). The principle is to build the package in each sides, hash the result and compare it. If the result is the same, then you're sure that the developer and the repository packages are safe. 

> Trust does not exclude control

Reproducible builds needs that the developers ensure that the way he codes his application always results in the same output when building. Not all developers are using it, because it's sometime hard to implement. Another problem is that closed source cannont reproducibly built, because the repository side doesn't have access to the source code.  
[F-droid supports reproducible builds](https://f-droid.org/docs/Reproducible_Builds/).

If you need to install a F-droid package anyway, [Neo store](https://github.com/NeoApplications/Neo-Store/) is a F-Droid client "with modern UI and an arsenal of extra features", which uses latest SDK.  

#### Some Alternatives

As a long term solution, I keep an eye on [Accrescent](https://accrescent.app/). Instead of putting your trust only on the repository, it let developers build and sign their own packages and upload it to the repository, then you need to trust only the developer.  
Accrescent doesn't aim to be a F-droid replacement, only an alternative, but it solves a lot of F-droid problems.  

Another alternative is [Obtainium](https://github.com/ImranR98/Obtainium) which allows you to get releases directly from sources, Github Releases for exemple.

#### Google Play apps

There is two way to manage Google Play applications.  

Google Play Store provides unattended upgrades, but needs a google account.
[Aurora Store](https://gitlab.com/AuroraOSS/AuroraStore) applications needs to be updated manually, but apps can be installed anonymously.  

Deal with that.

> On GrapheneOS, you would prefer a Google Play app version in front of the F-droid one, because build have quality controls !

#### Banking apps

Some [banking app could not work](https://grapheneos.org/usage#banking-apps) on GrapheneOS. A [community maintained list](https://privsec.dev/posts/android/banking-applications-compatibility-with-grapheneos/) try to list supported apps, feel free to [submit a new report](https://privsec.dev/posts/android/banking-applications-compatibility-with-grapheneos/#submit-a-new-app-report) to update the list.  

If your banking app worked on CalyxOS, it should work on GrapheneOS too.

### Hardware support

GrapheneOS [drops support](https://grapheneos.org/faq#device-lifetime) when Google stops hardware support. As Google stops updating firmware, it's impossible to get full security update. The device then get extended support until the next Android version, but you now need to update your hardware.  
The support of the Pixel 4a 5g stops at November 2023.

[Next Android release](https://developer.android.com/about/versions/14/get) planned to support Pixel 4a 5g.
The preview is available since [February 2023](https://en.wikipedia.org/wiki/Android_version_history), and [the timeline plan the stable version)(https://developer.android.com/about/versions/14/overview) for August 2023.

We can then expect to wait one year before the next stable release which leads to a supposed support date until ~ **August 2024**.

CalyxOS [supports my hardware](https://calyxos.org/docs/guide/device-support/) until **January 2025**, but can't ensure that it will be secure to use after the OEM end date.  
GrapheneOS let me know that I need to renew my hardware as early as possible to keep a secured device.  

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

On GrapheneOS, apps execution is sandboxed. You can also deny storage and contact permission manually for an app.  
An alternative is to use [storage scopes](https://grapheneos.org/usage#storage-access).
Storage scope can be enabled for an app, to ensure that it will only access files created by itself.

### Others features

Backup tool is [SeedVault](https://discuss.grapheneos.org/d/15-how-do-i-backup-my-phone), as in CalyxOS.  

### TOTEST

- Auditors
- Storage scope

### TODO

- upgrade process
  - backup app datas : TOLIST
  - flash GrapheneOS then : https://grapheneos.org/install/

### TOBACKUP

- [x] App list
- [x] Photos : syncthing
- [x] OSMAnd+ : Paramètres / Import/Export
- [x] Contact : carddav but export just in case
- [x] Notes : in a file
- [x] Catima : export
- [x] Mes depenses : export
- [x] Pilote budget : export mail
- [x] Progress bar
- [x] easer
- [/] seedvault backup
  - [x] main profile
  - [ ] work profile ?
- [ ] Whatsapp history export
- [ ] sms exports

###

That Android journey added a new point of view, with Linux distributions and BSD ones, about software deployment. The concern remains the same, like in day to day life: who do **you** choose to trust ? Who is the safest party, the repository (Google, f-droid) or the developer ? Reproducible build is a way to double check that both are not lying, but is hard to generalize.  

IMO, a downside to the GrapheneOS paradigm is that it doesn't push app developers to stop using google services. However, GrapheneOS is for sure the best solution to maximise compatibility and security.  

Thanks to drav-corp and matchboxbananasynergy from ``irc.libera.chat#grapheneos`` who helped me to write this and understand the specifics of each packaging solution.
