---
title: Manage FreeBSD Jails with Ansible - part 2
layout: post
icon: fa-code
published: false
---

{:toc}

In the [first part]({{ site.baseurl }}{% link _posts/*.md %}), we created the Ansible project to manage Jails with shared IP. In this post, we will adapt our playbok to create [vnet](https://www.unix.com/man-page/freebsd/9/vimage/) jails.

``vnet`` gives to jails their own network stacks. Each Jail will have a specific network interface with [epair](https://www.freebsd.org/cgi/man.cgi?query=epair&sektion=4&manpath=freebsd-release-ports) connected to a [bridge](https://www.freebsd.org/cgi/man.cgi?query=bridge&sektion=4&manpath=freebsd-release-ports). Let's quote something I read in a forum which resume well how it works:

> Analogous to a physical network, a bridge(4) interface works like a software switch, an epair(4) works like a virtual network cable and a jail(8) acts as a virtual computer.

The bridge creation will be automated with [jib](https://github.com/freebsd/freebsd-src/blob/373ffc62c158e52cde86a5b934ab4a51307f9f2e/share/examples/jails/jib) script.
Let's adapt our playbook to create that kind of jail networking.

There is only few steps, but before all, reset our first configurations.

### Cleaning environment

Remove the task which creates aliases, and remove them from your system.
```bash
$ ansible host-test -m lineinfile -a 'path=/etc/rc.conf regexp="{{ ansible_default_ipv4.device }}_alias.*" state=absent'
$ ansible host-test -m raw -a "service netif restart"
```

### Declare vnet and jib in jail.conf
Let's change our task to declare jails as follow.
```yaml
- name: declare jails
  vars:
    alias_ip: "{{ inet | ipmath({{ loop.index1 }}) }}"
  copy:
    dest: /etc/jail.conf
    content: |
      # Global settings
      exec.start = "/bin/sh /etc/rc";
      exec.stop = "/bin/sh /etc/rc.shutdown";
      exec.clean;
      mount.devfs;

      # Jails definition
      {% for jail in jails %}
      {{ jail }} {
          host.hostname = "{{ jail }}.domain.local";
          path = "/usr/local/jails/{{ jail }}";
          vnet;
          vnet.interface = "e0b_{{ jail }}";
          exec.prestart += "jib addm {{ jail }} {{ ansible_default_ipv4.device }}";
          exec.poststop += "jib destroy {{ jail }}";
          exec.consolelog = "/var/log/jail_{{ jail }}.log";
      }
  loop: "{{ jails | sort | flatten(levels=1) }}"
```
When the jail will start, ``jib`` will create a bridge if non existent, create epairs and automatically attach them to the bridge. Stopping the jail will destroy interfaces but not the bridge.

### Configure jails rc.conf
Add that new task to configure network in jails ``rc.conf``.
```yaml
- name: configure jails interfaces
  vars:
    jail_ip: "{{ inet | ipmath({{ loop.index1 }}) }}"
  copy:
    dest: "/usr/local/jails/{{ item }}/etc/rc.conf"
    content: |
      ifconfig_e0b_{{ jail }}="inet {{ jail_ip }} netmask 255.255.255.0"
      defaultrouter="{{ gateway }}"
  loop: "{{ jails | sort | flatten(levels=1) }}"
```
