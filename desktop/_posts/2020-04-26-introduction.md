---
title: Introduction
layout: post
icon: fa-quote-right
---

I manage ~800 Ubuntu nodes at work, so i searched a deployment method which allow me to fully automate packages and configurations deployments.   

That's how i met Ansible.

Managing hosts with ansible has many benefits:

- No agent : it uses SSH.
- Playbooks are written in *YAML* which is human readable. As each tasks has a name, plays are auto documented.
- Idempotency : Ansible check if what you want is already done. Remove an inexistant file will not break your play.

Managing my desktops with Ansible allows me to automate my setup over multiple hosts with specific configuration (dualscreen, packages...) by using jinja2 templating.  
Centralizing my configurations on github is confortable, i can pull my configuration from anywhere, and deploy it to all my managed nodes easily.

I wanted my setup to be modulable at multiple levels. I use roles to be able to separate servers from desktops configuration.  
Since i started to use Nixos, i separated my package management project from my dotfiles and added my install scripts to automate process from the beggining.  
Then i can use Nixos embeedeed automation process to configure it, then apply my dotfiles with ansible too in a second time.
