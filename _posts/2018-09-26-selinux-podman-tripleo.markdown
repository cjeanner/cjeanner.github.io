---
layout: post
title:  "Working on Podman integration in TripleO: SELinux in da place"
date:   2018-09-26 10:00:00 +0200
categories: openstack tripleo podman
---

Working on TripleO deploy framework is probably the most interesting thing you
might want to do. It allows you to discover a bunch of new things almost every
week if not day.

In my case, although I knew the "SELinux" name and its purpose, I never really
worked with it. I knew RHEL has it, and enforces the policies, and that it's
the same case for CentOS. But beyond that, I was clueless.

That changed dramatically once I got to work on Podman integration in TripleO.

Some basics: with the current release, we deploy the undercloud and overcloud
using containers, with the Docker engine. It does work. But without the SELinux
separation we can get using containers.

It was deactivated from the very beginning, meaning docker containers aren't as
isolated as we might think.

This tiny "hack" has been applied to the Docker daemon, and allows to avoid
any SELinux issues when we bind-mount volumes in one or multiple containers.

"Unfortunately", this hack doesn't work with Podman, as that nasty boy doesn't
have a daemon, and no real way to get a global configuration.

This means we had two choices: either modify all the calls to the container
engine in order to add the right option (```--security-opt label=disable```),
or make it work with an enforcing SELinux.

I chose the latter. Of course, it took some time (about 4 weeks), because I had
to:
- understand how SELinux works
- understand how SELinux works with containers
- understand what was failing with the deploy
- step-by-step correct the issues

If the two steps were easy (a couple of days), the next were really, really
painful, as I had to launch a deploy each time and check in parallel what was
going on in the `audit.log` file.

Also, an interesting difference between Podman and Docker: volumes.
If a directory/file doesn't exist on the host filesystem, Docker will create it.
On the contrary, Podman will just fail. Unfrotunately for me, this docker
capability was widely used, without knowing it was used...

In the end, a few patches were issued, and are being reviewed as I'm writing
this blog post:
- [Create the missing directories](https://review.openstack.org/600532)
- [Small correction for previous patch](https://review.openstack.org/605039)
- [Run bootstrap containers in privileged mode](https://review.openstack.org/600533)
- [Set proper setype on directories](https://review.openstack.org/600534)
- [Allow to deactivate SELinux separation on some containers](https://review.openstack.org/600535)
- [Allow container_t to access some specific other file type](https://github.com/redhat-openstack/openstack-selinux/pull/18)

With all those patches, we're able to deploy a complete, working undercloud,
with added security, as we get proper SELinux separation for a vast majority
of our containers. Some of them can't currently run with that separation, but
we're still working on them, hoping to get a fine solution.

Of course, other patches were also involved, and we had to report issues to the
Podman team - they are really responsive and concerned, meaning we could get a
really fast answer and correction for every issue we got.

A really nice thing is, we should be able to re-enable separation with Docker
as well, as the SELinux types are the same. Meaning: I've improved the overall
security of the product. And that's cool ;).
