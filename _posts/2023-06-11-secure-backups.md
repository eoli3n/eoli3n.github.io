---
title: Secure Backups
layout: post
icon: fa-hdd
icon-style: solid
---

I continue the post serie abou Borg, I demonstrate how to spread backups with borgmatic in [that previous post]({{ site.baseurl }}{% link _posts/2021-05-21-improve-backups.md %}) about how to setup BorgBackup and a first install of a home nas, I do have now two borg servers, lets spread our backups !

### Drop the shell backup script

[Borgmatic](https://torsion.org/borgmatic/) is an overlay to BorgBackup which let you configure everything with a yaml file and wrap Borg command to let you interact with your repository easierly.

Here a ``/etc/borgmatic/config.yaml`` file exemple

```yaml
location:
  source_directories:
    - /

  repositories:
    - root@nas:/data/backups/osz

  exclude_caches: true

  # See https://github.com/borgbackup/borg/pull/7635
  patterns:
    - R /
    - '- **/lost+found'
    - '- **/*.iso'
    - '- **/*.mkv'
    - '- **/*.vmdk'
    - '- **/*.pyc'
    - '- root/.cache'
    - '- home/*/.cache'
    - '- home/*/.var/app/*/cache' # flatpak caches
    - '- home/*/.local/share/Steam' # steam installed games
    - '+ etc/**'
    - '+ root/**'
    - '+ home/**'
    - '! re:^(dev|proc|run|sys|tmp)'
    - '- **'

storage:
  encryption_passphrase: "***************************"
  compression: zstd
  archive_name_format: '{hostname}-{now:%Y-%m-%dT%H:%M:%S}'
  ssh_command: ssh -i /root/.ssh/backup_rsa
  relocated_repo_access_is_ok: true

retention:
  prefix: '{hostname}-'
  keep_daily: 7
  keep_weekly: 4
  keep_monthly: 6
  keep_yearly: 2

consistency:
  check_last: 3
```

On the repository server, you need to add a restricted authorized_key as explained in my previous post.
Then you can create the repository and start your first backup.

```bash
$ sudo borgmatic init --encryption=repokey-blake2
$ sudo borgmatic -v2
```

As the process is now standard, you can write an ansible task to add an anacron and automate backups more nicely.
```yaml
- name: automate daily backups
  copy:
    dest: /etc/cron.daily/backup
    mode: 0755
    content: |
      #!/bin/bash
      borgmatic -v1
```

The borgmatic toolbox let you interact with the repository from the client.
```bash
$ sudo borgmatic info
root@nas:/data/backups/osz: Displaying summary info for archives
Repository ID: 664b076398e3d4ef96031d33f99ec0df1bb98a8ca39b181052f5dbc6c335f70e
Location: ssh://root@nas/data/backups/osz
Encrypted: Yes (repokey BLAKE2b)
Cache: /root/.cache/borg/664b076398e3d4ef96031d33f99ec0df1bb98a8ca39b181052f5dbc6c335f70e
Security dir: /root/.config/borg/security/664b076398e3d4ef96031d33f99ec0df1bb98a8ca39b181052f5dbc6c335f70e
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
All archives:                1.90 TB              1.08 TB            149.50 GB
                       Unique chunks         Total chunks
Chunk index:                  961954              8286300

$ sudo borgmatic list
root@nas:/data/backups/osz: Listing archives
osz-2021-05-14T18:19:19-voidlinux-install Fri, 2021-05-14 18:19:20 [dd21865bff728fdf4751cdc0e1f714164436eb5863452298b72952093dfbad4c]
osz-2021-05-15T11:50:05              Sat, 2021-05-15 11:50:06 [a38b92d57f58c97195e42047611679aa24a065a092da93d6ed9a68d7d94a52ad]
osz-2021-05-16T12:21:03              Sun, 2021-05-16 12:21:03 [b2e2f061939bb4818cb7be33e7da2c572ce7b60ebb6f9482ed317e18cf01895f]
osz-2021-05-17T09:23:14              Mon, 2021-05-17 09:23:15 [183c1b2f399999012cfa977a3b5f67ca3b8b0299384adfad486e3858a469659e]
osz-2021-05-18T18:36:34              Tue, 2021-05-18 18:36:35 [b71d73f6d62a328ff09c5728ddb042b6858148fcfbfe078828178abd34a10795]
osz-2021-05-19T08:16:24              Wed, 2021-05-19 08:16:24 [514b13dcdfbca936930adf67066bb7fcb5e668f5301c0d5ec1a98915c9926bb9]
osz-2021-05-20T09:40:59              Thu, 2021-05-20 09:40:59 [de40b1ca2cfe99893cb8023b2c496ba56695f3596199a126ac79ccf36ee566d0]

$ sudo borgmatic mount --archive osz-2021-05-19T08:16:24 --mount-point /mnt

$ sudo ls /mnt
bin   dev  etc	lib	lib64  mnt  proc  run	sys	tmp  var
boot  efi  home  lib32	media  opt  root  sbin	sysroot  usr
```

### Spread your Backups

I use [Syncthing](https://syncthing.net/) to sync my backups over the network between two repository servers.
The important line in the client configuration is ``relocated_repo_access_is_ok: true`` which lets you access your backups from the second server.
Syncing a borg repository is not the recommended way to spread your backups, because if a data corruption occurs on one side, it is stupidly replicated.
You should prefer to add a second repository in the yaml config, borgmatic will trigger two separated backups.

I chose to sync with Syncthing because one of my repository is accessible only from a OTP secured vpn. I can't automate VPN connection on all clients that I backup.

The replication is done in two ways, from ``server 1`` to ``server 2`` and vice-versa.

_02/01/22 edit_

As borg [documentation says](https://borgbackup.readthedocs.io/en/stable/faq.html#can-i-copy-or-synchronize-my-repo-to-another-location), borg repositories are not designed to be synced.
When I switched to redundant backups, I had to debug my repositories for few hours... So, just ,don't.
Using two backup locations in borgmatic config will took twice the time, but security worth it.
```yaml
  repositories:
    - root@nas:/data/backups/osz
    - root@borgbase:/repo
```

### Online Backups

If you backup at home and at a different location, that's pretty solid. I was annoyed by the fact that I backup my personnal data at work as second place, and wanted, for my most important data, to be safe to move in another city, and changing work without to be worried about my backups.

[BorgBase](https://www.borgbase.com/) describes itself as "Simple and Secure Offsite Backups" service.
To use it, simply open an account for free, to test. Then you will be able to upgrade your plan to the 100G small plan. That's enough for me, for my most important data, and only costs 2€/month under 100G, and then 0.01€/Go/month.

![borgbase]({{site.baseurl}}/assets/images/server/borgbase.png)

Repositories support encryption, and the web UI is secured with 2fa TOTP authentication.
I upload my backups at 12Mo/s, so I'm fully satisfied with the service.
You can enable alerts when repositories didn't get any backup since some days.
Let's see with time.
