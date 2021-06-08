---
title: Manage FreeBSD Jails with Ansible - part 1
layout: post
icon: fa-code
published: false
---

{:toc}

When it comes to manage an OS configuration, I always fully automate the process, it is a oneshot work, you write it, test it, forget it, until you need to modify the process. I kind of use Ansible to make the work concrete as I use my blog to clarify my reflection.

[Jails](https://docs.freebsd.org/en/books/handbook/jails/) are like containers for FreeBSD, it lets you isolate your services from each others. Each Jail has its own IP, there are different ways to manage networking, let's explore automation for each.

### Init Ansible

##### Create project and configure
Create your Ansible project root.
```bash
# Init tree
$ mkdir jails && cd jails
$ git init
$ mkdir roles group_vars

# Create inventory, if your host is configured with DHCP, check its IP
$ echo << EOF > hosts
host-test ansible_host=192.168.0.44
EOF

# Configure ansible
$ echo << EOF > ansible.cfg
[defaults]
stdout_callback = yaml
inventory = hosts
remote_user = root
interpreter_python = auto_silent
EOF

# Init the playbook
$ echo << EOF > playbook.yml
---
- hosts: host-test
  roles:
    - { role: jails, tags: jails }
EOF

$ mkdir -p roles/jails/tasks
```

Install [sysrc](https://docs.ansible.com/ansible/latest/collections/community/general/sysrc_module.html) ansible module.
```bash
$ ansible-galaxy collection install community.general
```

##### Make your project dynamic
Add a configuration file as ``group_vars/all.yml``.
```yaml
# Define the static IP of your jails host
inet: 192.168.0.100
netmask: 255.255.255.0
# Define its default gateway
gateway: 192.168.0.254
```

##### Prepare the jails host
Bootstrap [FreeBSD for Ansible](https://docs.ansible.com/ansible/latest/user_guide/intro_bsd.html#bootstrapping-bsd).
```bash
$ ansible host-test -m raw -a "pkg install -y python37"
```

Configure your default network interface
```bash
$ ansible host-test -m sysrc -a 'name="ifconfig_{{ ansible_default_ipv4.device }}" value="inet {{ inet }} netmask {{ netmask }}"'
$ ansible host-test -m sysrc -a 'name="defaultrouter" value="{{ gateway }}"'
```
Restart ``netif`` service to read you network configuration for ``rc.conf``.
```bash
$ ansible host-test -m raw -a "service netif restart"
```

Adapt your inventory with the new static IP and you're ready.
```bash
echo << EOF > hosts
host-test ansible_host=192.168.0.100
EOF
```

### Shared IP Jail

The simpler way to create a Jail is to add IP alias to you network interface and then bind that IP to your Jail.
For the exemple, let's create 2 jails, ``bind`` and ``nginx``.
First we need to create two IP aliases, default IP is incremented with ``ipmath`` which needs ``netaddr`` python package installed on the controller.

Let's declare our jails in a list in ``group_vars/all.yml``

```yaml
jails:
  - bind
  - nginx
```

##### Configure IP aliases
In ``roles/jails/tasks/main.yml``.
We use ``index.loop`` to increment ``inet`` IP with ``ipmath`` which needs ``netaddr`` python package installed on the controller. ``index.loop1`` starts index at ``1`` instead of ``0``.
To be sure that the list will always be processed in the same order, it needs to be explicitly sorted.

```yaml
---
- name: create IP aliases for jails
  vars:
    alias_ip: "{{ inet | ipmath({{ loop.index1 }}) }}"
  community.general.sysrc:
    name: 'ifconfig_{{ ansible_default_ipv4.device }}_alias{{ loop.index }}'
    value: 'inet {{ alias_ip }} netmask {{ netmask }}'
  loop: "{{ jails | sort | flatten(levels=1) }}"
  notify: restart netif
```

You need a handler to trigger the ``netif`` service restart on configuration update.
In ``roles/jails/handlers/main.yml``.
```yaml
---
- name: restart netif
  service:
    name: netif
    state: restarted
```

Run your roles, and test.
```bash
$ ansible-playbook playbook.yml

$ ansible host-test -m shell -a "ifconfig vtnet0 | grep inet"
	inet 192.168.0.100 netmask 0xffffff00 broadcast 192.168.0.255
	inet 192.168.0.102 netmask 0xffffff00 broadcast 192.168.0.255
	inet 192.168.0.101 netmask 0xffffff00 broadcast 192.168.0.255
```

##### Provision jails environments

```yaml
- name: create top zfs jails dataset
  community.general.zfs:
    name: zroot/jails
    state: present
    extra_zfs_properties:
      mountpoint: /usr/local/jails

- name: create zfs per jail dataset
  community.general.zfs:
    name: "zroot/jails/{{ item }}"
    state: present
  loop: "{{ jails | sort | flatten(levels=1) }}"
```

Install FreeBSD on the jails with a custom ``bsdinstall`` script.
I faced an issue here, ``bsdinstall`` uses its ``jail`` argument to target the specific environment, and its ``script`` argument to automate the process. Using both make it ignore the second one.

I improved the ``jail`` script to be able to use a ``SCRIPT`` env var with the script path, to automate in jail provisionning and [created a PR](https://github.com/freebsd/freebsd-src/pull/473).

Now let's create templates for automated provisionning.
I install ``{{ item }}`` just for the demonstration. Maybe that task should be not factorized and declarative for each jail.

```yaml
- name: template bsdinstall script
  copy:
    dest: "/usr/local/jails/{{ item }}.template"
    content: |
      DISTRIBUTIONS="base.txz"
      export nonInteractive="YES"

      #!/bin/sh
      pkg install {{ item }} -y
  loop: "{{ jails | sort | flatten(levels=1) }}"
```

And then triggers the provisionning with the ``shell`` module and ``args: creates:`` to make it idempotent.
```yaml
- name: bsdinstall jails
  shell: bsdinstall jail /usr/local/jails/"{{ item }}"
  environment:
    SCRIPT: "/usr/local/jails/{{ item }}.template"
  args:
    creates: "/usr/local/jails/{{ item }}/bin"
  loop: "{{ jails | sort | flatten(levels=1) }}"
```

##### Declare your jails
Last thing we need is to declare jails in ``/etc/jail.conf``.

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
          exec.consolelog = "/var/log/jail_{{ jail }}.log";
          ip4.add = {{ alias_ip }};
      }
  loop: "{{ jails | sort | flatten(levels=1) }}"
```
Let's tell ``rc.conf`` to run jails at startup.
```yaml
- name: start jails at startup
  community.general.sysrc:
    name: "jail_enable"
    value: "YES"
```

##### Run and test

Run the playbook to provision jails
```bash
$ ansible-playbook playbook.yml
[ paste result ]
```

[ TEST ]

In the next part, we will see how to provision ``vnet`` jails.

### VNET Jail

``vnet`` gives to jails their own network stacks. Each Jail will have a specific network interface with [epair](https://www.freebsd.org/cgi/man.cgi?query=epair&sektion=4&manpath=freebsd-release-ports) connected to a [bridge](https://www.freebsd.org/cgi/man.cgi?query=bridge&sektion=4&manpath=freebsd-release-ports). Let's quote something I read in a forum which resume well how it works:

> Analogous to a physical network, a bridge(4) interface works like a software switch, an epair(4) works like a virtual network cable and a jail(8) acts as a virtual computer.
> The network configurations then work almost identically to physical devices.

The bridge create will be automated with FreeBSD ``jib`` script.
Let's adapt our playbook to create that kind of jail networking.



### Bastille
