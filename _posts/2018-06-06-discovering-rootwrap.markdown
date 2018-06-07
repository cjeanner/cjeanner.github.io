---
layout: post
title:  "Discovering rootwrap"
date:   2018-06-06 08:52:12 +0200
categories: openstack tripleo security tripleoclient
---

While digging in the pythong-tripleoclient code in order to find out why and where root accesses were
actually needed, and starting to replace them with a simple "sudo" call from ```subprocess.call``` 
or ```subprocess.check_call```, I was pointed to the [oslo.privsep](https://github.com/openstack/oslo.privsep)
project.

It's a pretty nice project to be honnest. It allows to use another OpenStack project named
[oslo.rootwrap](https://github.com/openstack/oslo.rootwrap), which is "basically" a wrapper for sudo calls.

The really nice part in all that: your sudoers stays clean, and you actually DON'T need to manage it in some
weird way (either with packages, or pusing sudoers config via ansible and the like), no. You have one and
only one entry in your sudoer, and all the filtering and right management and security is done with a nice
configuration file.

You can get some really helpful information in those
[two](https://www.madebymikal.com/how-to-make-a-privileged-call-with-oslo-privsep/)
[blogposts](https://www.madebymikal.com/adding-oslo-privsep-to-a-new-project-a-worked-example/)

More to come of course - security and privileges are a really important matter in TripleO - we aim at squashing rights
and getting ride of the dreadfull ```NOPASSWD:ALL``` currently scatering the sudoers.

Using rootwrap and its friend privsep will allow to get a really convenient and common way to get instant privileges for
needed tasks.
