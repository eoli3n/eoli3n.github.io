---
title: Manage FreeBSD Jails with Ansible - part 3
layout: post
icon: fa-code
published: false
---
* TOC
{:toc}

In the [second part]({{ site.baseurl }}{% link _posts/2021-06-09-jails-part-2.md %}), we adapted our Ansible project to manage Jails with vnet.
The two first post was usefull to understand basic Jail creation, let's now wrap it with [Bastille](https://github.com/BastilleBSD/bastille).
Bastille is the BSD Docker-like toolset for managing containers.
That solution has many advantages, it can wrap many [commands](https://github.com/BastilleBSD/bastille#basic-usage) to manage Jails without having to rewrite it with Ansible.
It also has a [template](https://github.com/BastilleBSD/bastille#bastille-template) feature to automate Jail provisionning.
The ``template`` automate Jails creation with a ``Bastillefile``. Note the docker reference, even the syntax looks similar.

> So why wrapping Bastille with Ansible ?

Using Bastille let you provision a Jail easily, but it does not loop Jails creation nor automate the configuration of your services.

Following Bastille documentation, we will configure the server as if it was in DMZ, using [pf](https://www.openbsd.org/faq/pf/) as firewall to expose containers ports.

Remove all roles in the Ansible project. As ansible code is pretty clean to read, I will not comment a lot, and this post will be composed of a lot of code.

{% raw %}
### Network role: bastille0 bridge

We need to match [network requirements](https://github.com/BastilleBSD/bastille#network-requirements).
Create a *network* role.

``roles/network/tasks/main.yml``
```yaml
---
- name: Add lo1 interface
  community.general.sysrc:
    name: cloned_interfaces
    state: value_present
    value: "lo1"

- name: Name lo1 interface bastille0
  community.general.sysrc:
    name: ifconfig_lo1_name
    value: "bastille0"
  notify: netif cloneup
```

``roles/network/handlers/main.yml``
```yaml
---
- name: netif cloneup
  shell: service netif cloneup
```

### Firewall role: pf firewall

Create a *firewall* role.
``roles/firefall/tasks/main.yml``
```yaml
---
- name: enable pf
  community.general.sysrc:
    name: pf_enable
    value: "YES"
  notify: start pf

- name: enable pflog
  community.general.sysrc:
    name: pflog_enable
    value: "YES"
  notify: start pflog

- name: template pf.conf
  template:
    src: pf.conf.j2
    dest: /etc/pf.conf
  notify: reload pf
```

``roles/firefall/templates/pf.conf.j2``
```
ext_if="{{ ansible_default_ipv4.interface }}"

### Default block policy is to return a reset packet
set block-policy return
### Reassemble fragmented packets
scrub in on $ext_if all fragment reassemble
### Ignore loopback interface
set skip on lo

### Allow empty table to exist
table <jails> persist
### Nat in jails table
nat on $ext_if from <jails> to any -> ($ext_if:0)

### Static rdr
# rdr pass inet proto tcp from any to any port {80, 443} -> 10.17.89.45

### Enable dynamic rdr (see below)
rdr-anchor "rdr/*"

### Block on incoming traffic
block in all
### Allow outgoing, skip others rules if match, and track connections
pass out quick keep state
### Block all incoming traffic from the $ext_if subnet which is not from $ext_if interface
### And block incoming traffic from $ext_if IP on $ext_if interface
antispoof for $ext_if inet
### Allow SSH
pass in inet proto tcp from any to any port ssh flags S/SA keep state
```

We use ``async`` on ``pf`` restart to [keep ansible connection](https://docs.ansible.com/ansible/latest/user_guide/playbooks_async.html#avoid-connection-timeouts-poll-0).
``roles/firewall/handlers/main.yml``
```yaml
---
- name: start pf
  service:
    name: pf
    state: started
  async: 45
  poll: 5

- name: start pflog
  service:
    name: pflog
    state: started

- name: reload pf
  shell: pfctl -nf /etc/pf.conf && pfctl -f /etc/pf.conf
```

### Jails role
##### Install Bastille

Create a role ``jails``.
``roles/jails/tasks/main.yml``
```yaml
---
- name: install bastille
  pkgng:
    name: bastille

- name: enable bastille
  community.general.sysrc:
    name: bastille_enable
    value: "YES"

- name: add bastille devfs rule
  blockinfile:
    path: /etc/devfs.rules
    marker: "<!-- {mark} ANSIBLE MANAGED vnet -->"
    create: yes
    block: |
      [bastille_vnet=13]
      add path 'bpf*' unhide

- name: enable zfs for bastille
  community.general.sysrc:
    name: "{{ item.name }}"
    value: "{{ item.value }}"
    path: /usr/local/etc/bastille/bastille.conf
  loop:
    - { name: "bastille_zfs_enable", value: "YES" }
    - { name: "bastille_zfs_zpool", value: "zroot" }
```

Bootstrap the latest realease and configure it to use latest pkgs.
Add a var to ``group_vars/all.yml``
```yaml
release: 13.0-RELEASE
```
```yaml
- name: bootstrap 13.0 release
  shell: bastille bootstrap '{{ release }}' || true

- name: configure bootstrap to use latest pkgs
  replace:
    path: "/usr/local/bastille/releases/{{ release }}/etc/pkg/FreeBSD.conf"
    regexp: '^(.*)quarterly(.*)$'
    replace: '\1latest\2'

- name: update bootstrap
  shell: "bastille update {{ release }}"
```

### Web role
##### Prepare the nginx template

Create a role *nginx*.

```yaml
---
- name: Create services template dir
  file:
    path: /usr/local/bastille/templates/services/nginx
    state: directory
    recurse: yes

- name: Copy template config files
  copy:
    src: '{{ item }}'
    dest: /usr/local/bastille/templates/services/nginx/
  loop:
    - Bastillefile
    - OVERLAY

- name: Create nginx config path
  file:
    path: /usr/local/bastille/templates/services/nginx/usr/local/etc/nginx/
    state: directory
    recurse: yes

- name: Copy nginx config file
  copy:
    src: nginx.conf
    dest: /usr/local/bastille/templates/services/nginx/usr/local/etc/nginx/

- name: Create data/www dataset
  community.general.zfs:
    name: zroot/www
    state: present
    extra_zfs_properties:
      mountpoint: /data/www

- name: Copy index.html
  copy:
    src: index.html
    dest: /data/www/
```

``roles/nginx/files/Bastillefile``
```
PKG nginx
SYSRC nginx_enable=YES
CMD nginx -t
SERVICE nginx restart
CMD mkdir -p /data/www
FSTAB /data/www data/www nullfs ro 0 0
RDR tcp 80 80
```

``roles/nginx/files/OVERLAY``
```
usr
```

``roles/nginx/files/nginx.conf``
```
http {
    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /data/www;
            index  index.html index.htm;
        }
    }
}
```

``roles/nginx/files/index.html``
```html
<html>
THIS IS A TEST.
</html>
```

### Create a nginx jail

``roles/nginx/tasks/main.yml
```yaml
- name: create nginx jail
  shell: "bastille create nginx {{ release }} 10.0.0.1"
  args:
    creates: /usr/local/bastille/jails/nginx

- name: start nginx jail
  shell: bastille start nginx || true
```

### Template the nginx jail

```yaml
- name: template nginx jail with nginx template
  shell: bastille template nginx services/nginx

- name: restart nginx jail to read OVERLAY
  # https://github.com/BastilleBSD/bastille/issues/398
  shell: bastille restart nginx
```

### Test our jails

From the server.
```bash
$ curl http://10.0.0.1
<html>
THIS IS A TEST.
</html>
```

From a client in the main subnet.
```bash
$ curl http://192.168.0.100
<html>
THIS IS A TEST.
</html>
```

{% endraw %}
