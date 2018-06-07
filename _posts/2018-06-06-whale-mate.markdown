---
layout: post
title:  "Tmate in a Whale"
date:   2018-06-06 10:52:12 +0200
categories: tmate security docker
---

I use [tmate](https://tmate.io/). Like, a lot. It's really a
convenient way to get help from a colleague without having
to use some clunky etherpad. Want some hints on some code part
you can't understand, or get help on an issue in your code?
Just fire up tmate, pass the ssh link, and Voilà.

One of the issues with this tool is the risk caused to your system.

Indeed, if you pass the "read-write" link to your mate, you have to
fully trust him/her, as it will give a full access to your computer.

You run a sudo in tmate? Well, bad news, this can allow the connected peer
to gain root.

You have some sensitive content on your laptop? Well, bad news, your peers
can access them.

With that knowledge, I created a small docker composition allowing me to
virutally chroot tmate peers in a path, hence avoiding all the described
issues.

The project, [whale-mate](https://github.com/cjeanner/docker-tmate-client),
runs pretty well, and I already used it multiple time.

Just check the readme for the usage, and feel free to report issues, feature
requests and so on.

Happy hacking!
