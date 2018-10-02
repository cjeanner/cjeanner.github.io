---
layout: post
title:  "Tripleo-lab, Podman and TripleO, a love story"
date:   2018-10-02 12:00:00 +0200
categories: openstack tripleo podman containers
---

Still working on the integration of Podman in TripleO. Yeah, it's a long and
tricky road to success. But there are some really nice outcomes.

- The [openstack-selinux patch has been merged](https://github.com/redhat-openstack/openstack-selinux/pull/18),
  and a new package should be promoted shortly.
- Tripleo-lab now [allows to chose podman as a container client for the
  undercloud](https://github.com/cjeanner/tripleo-lab/commit/59957b12eddbf420df5fd3d384910efc694c76ee).

Tripleo-lab also allows to build or install custom packages. The built is based
on the [official OpenStack Gerrit](https://review.openstack.org/), and uses the
[official tripleo-ci tools](https://github.com/openstack-infra/tripleo-ci).

How can we use all of that? It's really simple. Let's say we want to deploy an
undercloud with podman, using custom `tripleo-heat-templates` and
`openstack-selinux` packages. In Tripleo-lab, create a "local_env" directory,
and add some files in it:

Describe what instance you want to build
```YAML
---
# local_env/1under.yaml
vms:
  - name: undercloud
    cpu: 6
    memory: 20000
    disksize: 100
    interfaces:
      - mac: "24:42:53:21:52:15"
      - mac: "24:42:53:21:52:16"
    autostart: yes
```

Ensure you're using the latest packages from Master
```YAML
---
# local_env/master.yaml
tripleo_version: master
```

Set podman as container CLI
```YAML
---
# local_env/podman.yaml
undercloud_config:
  - section: DEFAULT
    option: container_cli
    value: podman
```

Fetch and install custom packages based on changes
```YAML
---
# local_env/patches.yaml
patches:
  - name: 'tripleo-heat-templates'
    refs: '35/600535/16'
custom_rpms:
  - https://trunk.rdoproject.org/centos7-master/consistent/openstack-selinux-0.8.15-0.20181001144230.42045c1.el7.noarch.rpm
```

Note the "local_env" directory is ignored by default from the git repository.

Once you have those files with the wanted content, just launch ansible-playbook:
```Bash
ansible-playbook builder.yaml -e @local_env/1under.yaml \
	-e @local_env/master.yaml \
	-e @local_env/podman.yaml \
	-e @local_env/patches.yaml \
	-t lab
```

For now, there are still a "small" issue with podman, as apparently some
containers want to load kernel modules, and this action require elevated
privileges as well as the absence of selinux separation. I'm currently working
on the removal of those nasty calls, at least for the known modules coming from
[kolla](https://review.openstack.org/#/q/status:open+project:openstack/kolla+branch:master+topic:bug/1794550).

Another [row of patches](https://review.openstack.org/#/q/status:open+project:openstack/tripleo-heat-templates+branch:master+topic:bug/1794550)
are being prepared in order to load them from within tripleo-heat-templates
instead, as a "host_prep_tasks". I just need to make those modules persistent
across reboots - for now it's not the case with the current set of patches.


But in the end, it should all work as expected :). The kolla thing isn't 100%
necessary, as the "modprobe" command is smart enough to NOT try to reload an
already loaded module, so if we load it from the host before the container
starts, we're safe, but still. Not having modprobe calls from withing official
containers is a good thing.

Happy hacking ;).
