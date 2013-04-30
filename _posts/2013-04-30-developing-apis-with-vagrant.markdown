---
layout: post
title: Developing APIs with Vagrant
category: posts
---

> “We would skate up the Avenue, but there isn't any ice”
— A Couple of Swells

Don't do it. I mean, fine, use [Vagrant][1] if you're developing a single application with fifteen developers — consistency is great. Or if you're looking for an excuse to learn Chef, go nuts (although that sounds to me a lot like looking for an excuse to punch yourself repeatedly in the nose and mouth). But if, like us, you develop multiple apps in parallel (an API and a couple of clients for example), or if one of your developers is actually a shit-hot designer who happens to crush CSS like it was a fishing boat under a falling slab of ice in a Norwegian fjord, but who would rather not touch the command line thanks for asking, then you might want to give it a miss. I've read a lot of reasons why you should use Vagrant for Rails development so here are my reasons why not to:
 - a very minor one to start off with, but using mailcatcher is kind of annoying. You have to add `config.vm.forward_port 1080, 1080` to your Vagrantfile and then start up mailcatcher with `mailcatcher --http-ip 10.0.2.15` (or whatever the local IP of your vagrant box is). Who can remember that?
 - a much bigger issue, especially for that non-command-line developer working remotely. You can't get unicorn to start when you boot vagrant because it relies on a config file in  the shared `/vagrant` folder which won't be mounted yet. So every time you restart your computer you have `vagrant ssh` in and then you have to `sudo service unicorn restart` or something equally ridiculous.
 - One advantage of Vagrant is that you can replicate your production environment, almost. We use [Redis][2] and [ElasticSearch][3], both hosted externally; our vagrant box has them locally. _Almost the same_ is as reassuring as _not the same at all_, so why bother with all the faff.
 - You can't use [Zeus][4]. Well, maybe [you can][5], but I can't, and I really want to use Zeus.
 - I have a bunch of projects that run on my Mac, via [Pow][6] or whatever. When I'm using Vim in my vagrant box I can't `:r` any of files on my Mac. Although that shouldn't matter because my code couldn't be DRYer, obviously.
 - I can't use `pbpaste` or `pbcopy`, and I _need_ to use them because as great as [tmux][7] is, it is only medium amused with copying and pasting.

You love vagrant -- that's fine. But we're looking into [Boxen][8].

[1]:  http://www.vagrantup.com/
[2]:  https://redistogo.com
[3]:  http://www.found.no/
[4]:  https://github.com/burke/zeus
[5]:  https://github.com/burke/zeus/issues/231
[6]:  http://pow.cx/
[7]:  http://tmux.sourceforge.net/
[8]:  http://boxen.github.com/
