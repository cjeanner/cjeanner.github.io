---
layout: post
title:  "First ansible in heat"
date:   2018-06-06 08:52:12 +0200
categories: openstack tripleo heat config-download
---

One of my first "big" commit involved the learning of config-download
principles, as well as some bits of Ansible.

As of Queens (and even parts of Pike), TripleO deploy process wants to remove
Puppet, prefering Ansible for the configuration deployment and management.

This means, a lot of things are to be moved and changed. Currently, TripleO
calls Heat and Mistral, in order to launch processes directly from them.

Using config-download, Heat (and Mistral) will still be used, but more like some
kind of "configuration repository". They will generate ansible playbooks and
configurations, and the Heat agents will then download them, and execute them on
the different nodes.

My first patch in this "new" world was a simple migration from a Bash script
executed by Heat to a more modern approach, using Ansible instead of Bash,
and hence using the config-download process.

The [patch](https://review.openstack.org/570627) has been merged since, but the
whole process helped me understand how all of that works together, and how to test
the changes.

Basically, the change hits only the
[tripleo-heat-templates(https://github.com/openstack/tripleo-heat-templates) (tht/t-h-t)
repository.

In order to test the changes, I had only to get a working tripleo lab - I used
[quickstart](https://github.com/openstack/tripleo-quickstart) back then, in order
to get a working undercloud with 3 controllers and a compute.

The patch will only affect the controllers, as it's related to the public TLS certificate.

All I had to do was:
- build a new tripleo-heat-template package
- install the new package on the undercloud
- run a deploy from scratch of the overcloud
- run an update of the overcloud

*Note: All the commands are executed on the undercloud, as stack user.*


In order to build the package, I had to take the [tripleo-ci](https://github.com/openstack/tripleo-ci)
repository:
```Bash
git clone https://github.com/openstack/tripleo-ci
./tripleo-ci/scripts/tripleo.sh --delorean-setup
```
This will install all the requested softwares and libs in order to build proper RPMs.

All you have to do then is to clone your repository on the undercloud in the right location:
```Bash
git clone https://git.openstack.org/openstack/tripleo-heat-templates ~/tripleo/tripleo-heat-templates
```
Do your modifications, and commit your changes.

The package build is easy, but will take some times as it runs all the configured tests.
And you will just need to install the RPMs once it's done:
```
./tripleo-ci/scripts/tripleo.sh --delorean-build tripleo-heat-templates
sudo yum install $(find tripleo/delorean/tripleo/delorean/data/repos/ -name "*.rpm" -and -not -name "*src.rpm")
```

Finally, run the deploy as usual - using quickstart you can just use the provider ```overcloud-deploy.sh```
script.

You can also avoid package compilation: either set the ```--template``` path to the cloned version, or
symlink (```ln -s``` the default location (```/usr/share/openstack-tripleo-heat-templates```).

Either way, you must run the tests before submitting your review - unless you're happy seeing Zuul failing ;).

Happy hacking!
