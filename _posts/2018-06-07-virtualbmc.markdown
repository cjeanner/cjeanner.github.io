---
layout: post
title:  "VirtualBMC - an IPMI translator for Virsh"
date:   2018-06-07 09:30:00 +0200
categories: openstack tripleo
---

Being able to build a lab is interesting. Being able to mount it using only
virtual hardware is nice. Being able to manage your libvirt domains as if it
was actual hardware, with IPMI commands, this is a really, really nice thing.

Problem is: libvirt doesn't understand IPMI commands. It just doesn't speak
that language. Like not at all.

Solution is: VirtualBMC. This nice tool is provided in the Delorean
repositories, and translates IPMI commands into plain virsh commands. Really
nice, and works well.

The only issue is: the current python2-virtualbmc package doesn't offer a
proper systemd unit file. Meaning: you will need to manually restart the
service upon node reboot.

But this might change, there are some work toward a better integration,
better stability and so on.

In the meanwhile, I've created an ansible task and template in my
[tripleo-lab](https://github.com/cjeanner/tripleo-lab) project.

The main script,
[vbmc-run](https://github.com/cjeanner/tripleo-lab/blob/master/roles/builder/templates/vbmc-run.j2),
is easy to understand, even if you never made any jinja templating.

Basically, it will loop over the "vms" ansible var, and write down plain
vbmc commands. The script supports start, stop and reload arguments.

The
[unit file](https://github.com/cjeanner/tripleo-lab/blob/master/roles/builder/templates/vbmc.service.j2)
on the other hand allows to run the vbmc as non-root user. That non-root
user must have the rights to manage libvirt.

The whole glue that installs the script, set the service, and reload it when
needed (i.e. if the script or unit changes) is in a
[dedicated tasks file](https://github.com/cjeanner/tripleo-lab/blob/master/roles/builder/tasks/virtualbmc.yaml)

Regarding the non-root user and libvirt management: on CentOS, we have a
thing named [PolKit](https://en.wikipedia.org/wiki/Polkit) (fka PolicyKit).

We can easily allow a non-root user to make use of "virsh" commands, and
without hacking libvirtd daemon. Just feed PolKit with a file containing
the followin:

```
polkit.addRule(function(action, subject) {
        if (action.id == "org.libvirt.unix.manage" &&
            subject.user == "YOUR USER") {
                return polkit.Result.YES;
                polkit.log("action=" + action);
                polkit.log("subject=" + subject);
        }
});

```

No need to restart any service, just push that in
```/etc/polkit-1/rules.d/``` directory and you're good

Obviously, my tripleo-lab
[already does that](https://github.com/cjeanner/tripleo-lab/blob/master/roles/builder/tasks/libvirt.yaml#L30-L42)
as I'm using a non-root user for all the management.

