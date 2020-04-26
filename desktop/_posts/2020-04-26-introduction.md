---
title: Introduction
layout: post
icon: fa-quote-right
---

I manage ~800 Ubuntu nodes at work, so I looked for a deployment method which would allow me to fully automate software deployments and manage configurations after an OS provisionning.  

That's how I met Ansible.

The tool has many benefits:

- No agent : it uses SSH.
- Playbooks are written in *YAML* which is human readable. As each tasks has a name, plays are auto documented.
- Idempotency : Ansible check if what you want is already done. Remove an inexistant file will not break your play.

Managing my desktop recipe with Ansible allows me to deploy my setup over multiple hosts with specific configuration (dualscreen, packages...) by using jinja2 templating.  
Centralizing my configurations on github is confortable, I can pull my configuration from anywhere, and deploy it to all my managed nodes easily.
It also let me share my configurations with other linux users.

I wanted my setup to be modulable at multiple levels. I use roles and inventory to be able to separate servers from desktops configuration.  
Since I started to use Nixos, I separated my package management project from my dotfiles and added to each OS-projects my install scripts to automate provisionning process from the beggining.  
Then I can use Nixos embeedeed automation process to configure it, then apply my dotfiles with ansible too in a second time.

As my configuration is fully managed, i can auto test at each push with a Continuous Integration tool : [Travis-CI](https://travis-ci.org/github/eoli3n)
