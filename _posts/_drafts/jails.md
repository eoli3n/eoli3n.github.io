---
title: Manage FreeBSD Jails with Ansible
layout: post
icon: fa-code
published: false
---

When it comes to manage an OS configuration, I always fully automate the process, it is a oneshot work, you write it, test it, forget it, until you need to modify the process. I kind of use Ansible to make the work concrete as I use my blog to clarify my reflection.

[Jails](https://docs.freebsd.org/en/books/handbook/jails/) are like containers for FreeBSD, it lets you isolate your services from each others. Each Jails has its own IP, there are different ways to manage networking, let's explore automation for each.

### Init Ansible and networking

Once you created your Ansible project, and configure your inventory (with a temporary ``ansible_host`` ip), configure ``group_vars/all.yml``.
```yaml
# Define your static IP
inet: 192.168.0.100
netmask: 255.255.255.0
# Define your bridge default gateway
gateway: 192.168.0.254
```
Install sysrc ansible module
```bash
$ ansible-galaxy collection install community.general
```

Configure your network interface
```yaml
- name: configure default interface and gateway
  community.general.sysrc:
    name: '{{ item.name }}'
    value: '{{ item.value }}'
  loop:
    - { name: "ifconfig_{{ ansible_default_ipv4.device }}", value: "inet {{ inet }} netmask {{ netmask }}" }
    - { name: "defaultrouter", value: "{{ gateway }}" }
```
Run your playbook and then, on your host, restart ``netif`` service
```bash
$ service netif restart
```

Adapt your inventory with the new static IP and you're ready.

### Shared IP Jail

The simpler way to create a Jail is to add IP alias to you network interface and then bind that IP to your Jail.
For the exemple, let's create 2 jails, ``bind`` and ``nginx``.
First we need to create two IP aliases, default IP is incremented with ``ipmath`` which needs ``netaddr`` python package installed on the controller.
```yaml
- name: create IP aliases for jails
  community.general.sysrc:
    name: '{{ item.name }}'
    value: '{{ item.value }}'
  loop:
    - { name: "ifconfig_{{ ansible_default_ipv4.device }}_alias0", value: "inet {{ inet | ipmath(1) }} netmask {{ netmask }}" }
    - { name: "ifconfig_{{ ansible_default_ipv4.device }}_alias1", value: "inet {{ inet | ipmath(2) }} netmask {{ netmask }}" }
  notify: restart netif
```

You need a handler to trigger the ``netif`` service restart.
```yaml
- name: restart netif
  service:
    name: netif
    state: restarted
```

Run your tasks, and test.
```bash
$ ifconfig vtnet0 | grep inet
	inet 192.168.0.100 netmask 0xffffff00 broadcast 192.168.0.255
	inet 192.168.0.102 netmask 0xffffff00 broadcast 192.168.0.255
	inet 192.168.0.101 netmask 0xffffff00 broadcast 192.168.0.255
```

Create two zfs datasets.

```yaml
- name: create zfs jail datasets
  community.general.zfs:
    name: '{{ item }}'
    state: present
  loop:
    - zroot/jails/bind
    - zroot/jails/nginx
```

Install FreeBSD on the jails with a custom ``bsdinstall`` script.
I faced an issue here, ``bsdinstall`` uses its ``jail`` argument to target the specific environment, and its ``script`` argument to automate the process. Using both make it ignore the second one.



### VNET Jail

### Bastille
