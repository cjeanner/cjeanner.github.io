---
layout: post
title:  "TripleO and SELinux: timing"
date:   2019-03-22 12:00:00 +0200
categories: openstack tripleo selinux
---

We had a small issue in the TripleO CI: a VM image was enforcing SELinux, while
we usually set it to permissive.

As a matter of fact, we actually DO set it to permissive during the deploy,
but that configuration is applied with
[puppet](https://github.com/openstack/puppet-tripleo/commit/8e533aaf447022c62865130f2ffc88690f06aef1).

This means it kicks in at step 1, while we already have plenty of things done
on the host before that, and, in fact, it failed in the CI because we want
to configure an
[httpd vhost on port 8787](https://github.com/openstack/tripleo-heat-templates/blob/master/deployment/image-serve/image-serve-baremetal-ansible.yaml#L47-L50),
and apparently this port is already taken/flagged on Fedora 28 for jboss debug,
preventing httpd to listen.

After some checks and researches, it appears the Fedora 28 image is actually
enforcing SELinux while CentOS doesn't, and F28 has this tiny difference with
CentOS regarding that specific port. A good thing we could spot it.

This lead to some ping-pong on Launchpad, and a
[couple](https://bugs.launchpad.net/tripleo/+bug/1821025) of
[new issues](https://bugs.launchpad.net/tripleo/+bug/1821178).

In this specific case, we encounter two issues:

  - one regarding the port being unauthorized for httpd
  - one regarding the timing within the deploy steps

The first issue is really easy to correct, and a
[patch](https://review.openstack.org/#/c/645059/) was issued and quickly
merged.

The latter one is a bit more tricky, at least for me.

As there's a will to move away from Puppet in favour of Ansible, I took that
opportunity to manage SELinux directly with Ansible, at the earliest possible
stage.

The [following patch](https://review.openstack.org/645238) ensures we have a
proper SELinux state as the third or fourth task on the hosts - and as it's
a common file, used everywhere, we can also ensure ALL nodes are affected.

The small issue I had on that one was the way to actually inject the ansible
code in that file - a jinja2 template, that generates a proper playbook at
deploy time. This means a specific syntax is to be used, and it took me a
couple of hours to figure it out, while testing live the changes on my lab.

In the end, I succeeded, and could push a first version of the patch.


In order to ensure nothing else comes in the way, I've also
[dropped the Puppet part](https://review.openstack.org/645477). This ensures
we won't have any conflict, and runs are indempotent, which is a must.

Side notes:

I'm never happy to disable SELinux, but in the CI, it makes sense. More or
less. As long as people continue to test their own changes locally, with a
SELinux enforced system, and/or checks the audit.log, we're on the safe side.
