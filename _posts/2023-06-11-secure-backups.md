---
title: Secure Backups 
layout: post
icon: fa-hdd
icon-style: solid
---

I continue the post serie about Borg and Borgmatic, I demonstrated how to spread backups with borgmatic in [that previous post]({{ site.baseurl }}{% link _posts/2021-05-21-improve-backups.md %}), let's now secure those backup.      

### Why aren't backups not secure ?

The repository is configured to let clients access to the repo in read/write, which is the default mode.
Borg provides a ``--append-only`` mode, which let clients only add data. Then if you host is compromised, the hacker can not remove all your backups before crypt-locking your data.

### Configure the repository

To configure your repository in append-only mode, edit the line in ``authorized_keys`` file.

```diff
- command="borg serve --restrict-to-path /backups/host1",restrict ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCth9LMtb5y1qQHENUWLEhYF8gi6NnMwM69oj8Q80/HSM3PrNT3Wf81Q6AKw4f2miqfisxS5p7zCvjf7Yle3CQY0E5NlF/ZulP4aRShjH09N3STAWryUy5wlExmcLp+L07Tq9VvqHF0aSObdb7voLnOKemvt/xDXwR0UTl/gCdueKWLDZ+HiZc7cnAKhtI/KlYKy6nIJCDOyHVRbBbEyuTm78JHxueG2BR3KiZO46XQbuVsEFx8v7AxvCUEi/a+2r3WmsYP1ux3rZ4Gs1JeK2YCck31o/dcK9ZToVSrxD6EP/HH3h/ci0sWgt8goROhqaIjCrmLQKjPMUKgSoirQRO9 root@host1
+ command="borg serve --append-only --restrict-to-path /backups/host1",restrict ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCth9LMtb5y1qQHENUWLEhYF8gi6NnMwM69oj8Q80/HSM3PrNT3Wf81Q6AKw4f2miqfisxS5p7zCvjf7Yle3CQY0E5NlF/ZulP4aRShjH09N3STAWryUy5wlExmcLp+L07Tq9VvqHF0aSObdb7voLnOKemvt/xDXwR0UTl/gCdueKWLDZ+HiZc7cnAKhtI/KlYKy6nIJCDOyHVRbBbEyuTm78JHxueG2BR3KiZO46XQbuVsEFx8v7AxvCUEi/a+2r3WmsYP1ux3rZ4Gs1JeK2YCck31o/dcK9ZToVSrxD6EP/HH3h/ci0sWgt8goROhqaIjCrmLQKjPMUKgSoirQRO9 root@host1
```

### Configure clients

There is a problem then. That configuration prevents backup rotation from the client, ``prune`` is not now possible. To workaround this, you can configure borgmatic on the server, with a cron which runs everyday.

``borgmatic.d/host1-prune.yaml``
```yaml
location:
  source_directories:
    - /
  repositories:
    - /backups/host1

storage:
  encryption_passphrase: "********************************"
  compression: zstd
  archive_name_format: 'host1-{now:%Y-%m-%dT%H:%M:%S}'

  # It won't work without that line
  relocated_repo_access_is_ok: true

retention:
  prefix: 'host1-'
  keep_daily: 7
  keep_weekly: 4
  keep_monthly: 6
```

On the client, edit the ``borgmatic.d/config-nas.yaml`` file to remove the ``retention`` part.

```diff
  <<: !include /root/.config/borgmatic/config-main.yaml
  
  location:
    source_directories:
      - /
  
    repositories:
      - path: ssh://nas/data/zfs/backups/osz
        label: nas
  
  consistency:
    check_last: 3
- 
- retention:
-   prefix: '{hostname}-'
-   keep_daily: 7
-   keep_weekly: 4
-   keep_monthly: 6
-   keep_yearly: 1
```

### A last workaround

When running ``borgmatic`` command without specifying a subcommand, it defaulty runs ``create``, ``check`` based on ``consistency`` configuration and ``prune`` based on ``retention`` one.
Removing the retention part will not prevent borgmatic from triggering ``prune`` subcommand. You need then to run ``borgmatic create && borgmatic check`` explicitly instead of just ``borgmatic``.

I created an issue to be able to limit Borgmatic actions explicitly in the configuration file, this is on the road.
See [issue "unified borgmatic command to trigger actions (prune or not) from what is in the config file"](https://projects.torsion.org/borgmatic-collective/borgmatic/issues/701).
