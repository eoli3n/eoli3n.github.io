---
title: Introduction
layout: post
icon: fa-quote-right
---

Ansible allows me to automate my desktop setup over multiple hosts with specific configuration (dualscreen, packages...) by using jinja2 templating.  
Centralizing my configurations on github is confortable, i can pull my configuration from anywhere, and deploy it to all my managed nodes easily.

I wanted my setup to be modulable at multiple levels. I use ansible roles to be able to separate servers from desktops configuration.
Since i started to use Nixos, i separated my package management project from my dotfiles and added my install script to automate process from the beggining.  
Then i can use Nixos embeedeed automation process to configure it, or ansible for archlinux, then apply my dotfiles with ansible too in a second time.


