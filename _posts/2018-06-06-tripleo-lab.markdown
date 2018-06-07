---
layout: post
title:  "Get your TripleO lab"
date:   2018-06-06 10:52:12 +0200
categories: openstack tripleo
---

As a developper, we need to be able to test and validate
our code.

In some project, it's easy - you bootstrap a server somewhere,
you push your code, it builds, you check the output, blah.

But in TripleO and OpenStack world, it's a bit more complicated:
- you need to validate on a reproductible environment
- you need to be able to destroy all your test infra and start from scratch
- you don't want to lose 3 days in order to re-build your env
- usually a validation uses more than one server.

Currently, there's already the [quickstart](https://github.com/openstack/tripleo-quickstart)
project. It works well, but in fact, it goes too far, at least for my use.

I ended up creating a [new ansible receipt](https://github.com/cjeanner/tripleo-lab)
that will just deploy empty VMs:
- 1 undercloud
- 3 controllers
- 2 computes
- 3 ceph-storages

This basic setup allows me to test and validate a lot of cases, such as:
- multi-controller HA
- multi-compute scenarii
- and so on

Of course, I have a kind of a monster in order to deploy that lab - but in the
end, it's worth the expense.

The deploy is based on libvirt with the default network. It will also create a dedicated
user on your build machine, and let it access libvirt management.

In addition, it will push in your ~/.ssh/config some configuration in order to let you
access directly the undercloud node - this node is the only one running when the playbook is
over. The other nodes needing a deploy, it's not worth the resources to start them directly.

The undercloud VM will only get the tools and "stack" user, letting you manage the undercloud
install in whatever way you want (baremetal, containers, with specific package version, and
so on).

There are still a couple of issues for now, especially regarding the SSH jump to the undercloud
for some ansible tasks, but it works pretty well in the end.
