---
title: Password store
layout: post
icon: fa-key
---

Password managment is the cornerstone of services security. If you use web, you would create one random password per service, and activate [Time-based One-Time Password](https://en.wikipedia.org/wiki/Time-based_One-Time_Password) for all important one, like mail.

I used firefox lockwise to manage my web passwords, and FreeOTP on Android.  
Firefox Lockwise can be configured to asks for a master password at every accesses.  
Easy synchronization between multiple hosts, including my phone, sticked me to it.

Problem is that I don't really own my data, that I can't drop Firefox if I would like to, and I can't manage non web passwords.

[Pass](https://www.passwordstore.org/) defines itself like "**the standard unix password manager**".  
It uses [gnupg](https://gnupg.org/) to encrypt passwords as ``.gpg`` files, and git to distribute.  
A clever extension lets you use it with [rofi](https://github.com/davatorium/rofi) : [rofi-pass](https://github.com/carnager/rofi-pass)
But *pass* defaulty miss some comfortable features:
- Automatic commit/push/pull, if you forget to commit a change, you can't access it on other devices
- TOTP generator, but [pass-otp](https://github.com/tadfisher/pass-otp) exists

[Gopass](https://www.gopass.pw/) handles those by default, and even more.
You can manage multiple password stores and share them with your teams, with access control lists based on gpg.
A password leak and quality checker is embeeded, desktop notifications, etc..

### Setup and configure GnuPG

First, you need to create a gpg keypair. Ensure that you use gnupg2, some linux distros has gnupg *v1.x* in a different package.

On Voidlinux:
```bash
$ gpg2 --full-generate-key
# (1) RSA et RSA
# size: 3073 bits
# set expiration
# set your real name, email and comment
```
Generate a revoke certificate and store it in a safe place, check your *keyid* with ``gpg2 --list-keys``
```bash
$ gpg2 --gen-revoke --output revoke.asc "$key_id"
```
To backup your key, export you secret key and the trust db
```
$ gpg2 --export-secret-keys --armor "$key_id" > secret.asc
$ gpg2 --export-ownertrust > trustdb-backup.txt
```

Gnupg defaulty use a ncurses pinentry, you would change it for a graphical one, I chose ``pinentry-gtk`` and configured gpg by creating the file ``~/.gnupg/gpg-agent.conf``
```bash
pinentry-program /bin/pinentry-gtk-2
```

To test gpupg, you can try to sign a file with ``gpg2 --sign test.file``.

I use [keychain](https://www.funtoo.org/Keychain) to autostart gpg agent at first shell login, with fish
```bash
$ cat ~/.config/fish/conf.d/keychain.fish
# https://stackoverflow.com/questions/39494631/gpg-failed-to-sign-the-data-fatal-failed-to-write-commit-object-git-2-10-0
set -x GPG_TTY (tty)

# https://github.com/fish-shell/fish-shell/issues/4583
if status --is-interactive
    keychain --eval --agents ssh --quiet -Q id_rsa | source
    keychain --eval --agents gpg --quiet --gpg2 -Q | source
end
```

I don't target the key to load, when I will use it, it will be added after typing my passphrase.
Gpg agent is not like SSH agent, it forgets the passphrase token after *600 seconds* by default.

### Configure Git with Gnupg

Now, you need a centralised git repository, and the ability to sign your commits.  
I created mine on my nas server with [gitolite](https://gitolite.com/gitolite/index.html), but you can use a Github private repository.
To make signing work, I needed to edit my git-config to match my gpg binary and my gpg key.

```ini
$ cat ~/.gitconfig
[user]
    name = eoli3n
    email = jonathan.kirszling@runbox.com
    signingkey = eoli3n
[pull]
    rebase = false
[gpg]
    program = gpg2
```
Let's try to sign a commit:
```
$ mkdir git-gpg-test
$ cd git-gpg-test/
$ echo "testing" > README.md
$ git init
$ git add README.md 
$ git commit -S -m "testing to sign this commit"
[master (commit racine) c8fb990] testing to sign this commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
 ```

Ok, great, now let's create the password store.

### Password Store

```bash
$ gopass setup
   __     _    _ _      _ _   ___   ___
 /'_ '\ /'_'\ ( '_'\  /'_' )/',__)/',__)
( (_) |( (_) )| (_) )( (_| |\__, \\__, \
'\__  |'\___/'| ,__/''\__,_)(____/(____/
( )_) |       | |
 \___/'       (_)

🌟 Welcome to gopass!
🌟 Initializing a new password store ...
🌟 Configuring your password store ...
🎮 Please select a private key for encrypting secrets:
[0] gpg - 0xFEEDBEEF - Jonathan Kirszling <jonathan.kirszling@runbox.com>
Please enter the number of a key (0-12, [q]uit) (q to abort) [0]: 0
❓ Do you want to add a git remote? [y/N/q]: y
Configuring the git remote ...
Please enter the git remote for your shared store []: git@nas.domain.fr:passwords.git
✅ Configured
```

### Add your first password

Use ``insert`` subcommand to create password entries
```bash
$ gopass insert work/test
Enter password for work/test: 
Retype password for work/test: 
```
Querying it will prompt for you gpg passphrase.
```bash
$ gopass work/test
Secret: work/test

Passw0rd
```

### Add a TOTP

To add a TOTP key, you need to use ``--multiline`` argument
```bash
$ gopass insert -m work/vpn
totp: XXXXXXXXXXXXXXXXXXXXXX

$ gopass totp work/vpn
069268 lasts 8s 	|----------------------========|
```

### Synchronization with Android

On Android, you need [OpenKeychain](https://www.openkeychain.org/), to add your previously exported gpg secret key.  
To be able to use *TOTP* and your password store, use [Password Store](https://github.com/android-password-store/Android-Password-Store) with your ssh key to reach you git *passwords* repository. No need to use *FreeOTP* anymore.
*Password Store* does not pull/push commit automatically, don't forget to sync from the app !

### Configure a new node

Import your gpg key and your trust db
```bash
$ gpg2 —-import secret.asc
$ rm ~/.gnupg/trustdb.gpg
$ gpg2 --import-ownertrust < trustdb-backup.txt
```

Then gopass wrap your *passwords* reporistory git clone

```bash
gopass clone git@nas.domain.fr:passwords.git
```
You project with be stored in ``.local/share/gopass/stores/root/``.

### Go a bit further

On my desktop hosts, I now need to find an alternative to *rofi-pass* for [Wofi](https://cloudninja.pw/docs/wofi.html).
*gopass* should be fully compatible with *pass* bu default, expect for totp management which differs a bit.

*gopass* has a great plugin to manage passwords from browser: [gopassbridge](https://github.com/gopasspw/gopassbridge).
It would be a great firefox lockwise replacement.
