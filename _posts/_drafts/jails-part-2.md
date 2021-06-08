---
title: Manage FreeBSD Jails with Ansible - part 2
layout: post
icon: fa-code
published: false
---
* TOC
{:toc}

In the [first part]({{ site.baseurl }}{% link _posts/*.md %}), we created the Ansible project to manage Jails with shared IP. In this post, we will adapt our playbook to create [vnet](https://www.unix.com/man-page/freebsd/9/vimage/) jails.

``vnet`` gives to jails their own network stacks. Each Jail will have a specific network interface with [epair](https://www.freebsd.org/cgi/man.cgi?query=epair&sektion=4&manpath=freebsd-release-ports) connected to a [bridge](https://www.freebsd.org/cgi/man.cgi?query=bridge&sektion=4&manpath=freebsd-release-ports). Let's quote something I read in a forum which resume well how it works:

> Analogous to a physical network, a bridge interface works like a software switch, an epair works like a virtual network cable and a jail acts as a virtual computer.

The bridge creation will be automated with [jib](https://github.com/freebsd/freebsd-src/blob/373ffc62c158e52cde86a5b934ab4a51307f9f2e/share/examples/jails/jib) script.
Let's adapt our playbook to create that kind of jail networking.

There is only few steps, but before all, reset our first configurations.

### Cleaning environment

Remove the task which creates aliases, and remove them from your system.
```bash
$ ansible host-test -m shell -a "service jail stop bind && service jail stop nginx"
$ ansible host-test -m lineinfile -a 'path=/etc/rc.conf regexp=".*alias.*" state=absent'
$ ansible host-test -m file -a 'path=/etc/jail.conf state=absent'
$ ansible host-test -m raw -a "service netif restart"
```

### Install jib script
When the jail will start, ``jib`` will create a bridge if non existent, create epairs and automatically attach them to the bridge. Stopping the jail will destroy interfaces but not the bridge.
Copy the script in your path with execution perm.
```yaml
- name: install jib script
  copy:
    src: /usr/share/examples/jails/jib
    dest: /usr/local/bin/
    remote_src: yes
    mode: 0755
```

### Configure network stack in jails
Let's change our task to declare jails as follow.
```yaml
- name: declare jails
  vars:
    alias_ip: "{{ inet | ipmath(ansible_loop.index) }}"
  blockinfile:
    path: /etc/jail.conf
    marker: "# {mark} ANSIBLE MANAGED: {{ item }}"
    block: |
      {{ item }} {
          host.hostname = "{{ item }}.domain.local";
          path = "/usr/local/jails/{{ item }}";
          exec.consolelog = "/var/log/jail_{{ item }}.log";
          vnet;
          vnet.interface = "e0b_{{ item }}";
          exec.prestart += "jib addm {{ item }} {{ ansible_default_ipv4.interface }}";
          exec.poststop += "jib destroy {{ item }}";
      }
  loop: "{{ jails | sort | flatten(levels=1) }}"
  loop_control:
    extended: yes
```

### Configure epair interface in jails
Add that new task to configure network in jails ``rc.conf``.
```yaml
- name: configure jails interfaces
  vars:
    jail_ip: "{{ inet | ipmath(ansible_loop.index) }}"
  copy:
    dest: "/usr/local/jails/{{ item }}/etc/rc.conf"
    content: |
      ifconfig_e0b_{{ item }}="inet {{ jail_ip }} netmask 255.255.255.0"
      defaultrouter="{{ gateway }}"
  loop: "{{ jails | sort | flatten(levels=1) }}"
  loop_control:
    extended: yes
```

### Run the playbook and test connectivity
```bash
$ ansible-playbook playbook.yml
[...]

TASK [jails : set default jails config] ****************************************************************************
changed: [host-test]

TASK [jails : declare jails] ***************************************************************************************
changed: [host-test] => (item=bind)
changed: [host-test] => (item=nginx)

TASK [jails : configure jails interfaces] **************************************************************************
changed: [host-test] => (item=bind)
changed: [host-test] => (item=nginx)

TASK [jails : start jails at startup] ******************************************************************************
ok: [host-test]

TASK [jails : start jails] *****************************************************************************************
changed: [host-test] => (item=bind)
changed: [host-test] => (item=nginx)

PLAY RECAP *********************************************************************************************************
host-test                  : ok=10   changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
``jib`` created a bridge and attached jails interfaces in it.
```bash
$ ansible host-test -m shell -a "ifconfig vtnet0bridge | grep member"
host-test | CHANGED | rc=0 >>
	member: e0a_nginx flags=143<LEARNING,DISCOVER,AUTOEDGE,AUTOPTP>
	member: e0a_bind flags=143<LEARNING,DISCOVER,AUTOEDGE,AUTOPTP>
	member: vtnet0 flags=143<LEARNING,DISCOVER,AUTOEDGE,AUTOPTP>
```
Let's check if jails are running.
```bash
$ ansible host-test -m shell -a "jls"
host-test | CHANGED | rc=0 >>
   JID  IP Address      Hostname                      Path
     3                  bind.domain.local             /usr/local/jails/bind
     4                  nginx.domain.local            /usr/local/jails/nginx

```
As IP is configured in the Jail, host doesn't know which one it is.
As my controller host is on the same subnet, I can reach my jails directly from it.
```bash
$ ping -c 1 192.168.0.101 | grep packets
1 packets transmitted, 1 received, 0% packet loss, time 0ms

$ ping -c 1 192.168.0.102 | grep packets
1 packets transmitted, 1 received, 0% packet loss, time 0ms
```
Jails are now exposed on my private subnet.
