---
layout: post
title: "How to test patches within containers with tripleo-lab"
date: 2019-11-15 08:00:00 +0100
categories: openstack tripleo containers
---

While doing patch testing, it might happen we have to patch containers on the
fly. There's already a way to apply one patch to a container, but it has some
limitations:
- no way to apply multiple patches to the same container
- patch must be on gerrit

So, in a case I'm
[currently debugging](https://bugs.launchpad.net/tripleo/+bug/1850843), we had
to apply 3 patches from 3 different projects to some containers. The standard
way doesn't work, even if those patches are already merged/on gerrit (no
promotion so far, so no built containers with those changes, so... meh)

Here's the fast way I've used in order to ensure I can test those patches in
a working env.

First, I've prepared the repositories in my laptop. Ensuring I get the right
patches available for the different repositories, in a proper branch, allows
me to call the ```synchronize``` feature of tripleo-lab: it will push content
from my laptop to the undercloud VM, and build packages based on those
repositories, and finally install them. It's not fast (building takes time),
but at least it's automated.

In this case, those are [mistral](https://review.opendev.org/693189),
[mistral-lib](https://review.opendev.org/692967) and
[oslo.utils](https://review.opendev.org/692965).

Then, I've modified tripleo-heat-templates in order to bind-mount new
locations. Those locations are, in fact, the ones affected by the new packages.
This ensures containers are running with the versions I want, without the need
to rebuild them all.
That patch is pretty easy in the end:
```
commit d2798aa02cfa509ccbc0ea335d1cf3a3754d4b92 (HEAD -> hacking)
Author: Cédric Jeanneret <cjeanner@redhat.com>
Date:   Fri Nov 15 09:31:07 2019 +0100

    mount mistral/oslo patches
    
    Change-Id: Ib73d3baf6b29b305e09daa775af251dbee689eaa

diff --git a/deployment/mistral/mistral-api-container-puppet.yaml b/deployment/mistral/mistral-api-container-puppet.yaml
index dee5fadc8..0d0851bb0 100644
--- a/deployment/mistral/mistral-api-container-puppet.yaml
+++ b/deployment/mistral/mistral-api-container-puppet.yaml
@@ -196,6 +196,9 @@ outputs:
                   - /var/lib/kolla/config_files/mistral_api.json:/var/lib/kolla/config_files/config.json:ro
                   - /var/lib/config-data/puppet-generated/mistral/:/var/lib/kolla/config_files/src:ro
                   - /var/log/containers/mistral:/var/log/mistral:z
+                  - /usr/lib/python2.7/site-packages/oslo_utils:/usr/lib/python2.7/site-packages/oslo_utils:ro
+                  - /usr/lib/python2.7/site-packages/mistral_lib:/usr/lib/python2.7/site-packages/mistral_lib:ro
+                  - /usr/lib/python2.7/site-packages/mistral:/usr/lib/python2.7/site-packages/mistral:ro
             environment:
               KOLLA_CONFIG_STRATEGY: COPY_ALWAYS
         step_5:
diff --git a/deployment/mistral/mistral-engine-container-puppet.yaml b/deployment/mistral/mistral-engine-container-puppet.yaml
index 21123299c..e885eedae 100644
--- a/deployment/mistral/mistral-engine-container-puppet.yaml
+++ b/deployment/mistral/mistral-engine-container-puppet.yaml
@@ -129,6 +129,9 @@ outputs:
                   - /var/lib/mistral:/var/lib/mistral:ro
                   - /usr/share/ansible/:/usr/share/ansible/:ro
                   - /usr/share/openstack-tripleo-validations:/usr/share/openstack-tripleo-validations:ro
+                  - /usr/lib/python2.7/site-packages/oslo_utils:/usr/lib/python2.7/site-packages/oslo_utils:ro
+                  - /usr/lib/python2.7/site-packages/mistral_lib:/usr/lib/python2.7/site-packages/mistral_lib:ro
+                  - /usr/lib/python2.7/site-packages/mistral:/usr/lib/python2.7/site-packages/mistral:ro
             environment:
               KOLLA_CONFIG_STRATEGY: COPY_ALWAYS
       host_prep_tasks:
diff --git a/deployment/mistral/mistral-event-engine-container-puppet.yaml b/deployment/mistral/mistral-event-engine-container-puppet.yaml
index 21dfcd82e..757b7251e 100644
--- a/deployment/mistral/mistral-event-engine-container-puppet.yaml
+++ b/deployment/mistral/mistral-event-engine-container-puppet.yaml
@@ -104,6 +104,9 @@ outputs:
                   - /var/lib/mistral:/var/lib/mistral:ro
                   - /usr/share/ansible/:/usr/share/ansible/:ro
                   - /usr/share/openstack-tripleo-validations:/usr/share/openstack-tripleo-validations:ro
+                  - /usr/lib/python2.7/site-packages/oslo_utils:/usr/lib/python2.7/site-packages/oslo_utils:ro
+                  - /usr/lib/python2.7/site-packages/mistral_lib:/usr/lib/python2.7/site-packages/mistral_lib:ro
+                  - /usr/lib/python2.7/site-packages/mistral:/usr/lib/python2.7/site-packages/mistral:ro
             environment:
               KOLLA_CONFIG_STRATEGY: COPY_ALWAYS
       host_prep_tasks:
```
I've created a quick ```local_env/patches.yaml``` file in order to pass the
correct environment options to tripleo-lab:
```YAML
synchronize:
  - name: mistral
    base: /home/cjeanner/work/gerrit
    dest: /home/stack/tripleo/
  - name: mistral-lib
    base: /home/cjeanner/work/gerrit
    dest: /home/stack/tripleo/
  - name: oslo.utils
    base: /home/cjeanner/work/gerrit
    dest: /home/stack/tripleo/
  - name: tripleo-heat-templates
    base: /home/cjeanner/work/gerrit
    dest: /home/stack/tripleo/
  - name: tripleo-ansible
    base: /home/cjeanner/work/gerrit
    dest: /home/stack/tripleo/
```

Deploy your lab with the following:
```Bash
ansible-playbook builder.yaml -e @local_env/tmate.yaml \
  -e @local_env/master.yaml \
  -e @local_env/1ctl-2compute.yaml \
  -e @local_env/patches.yaml --skip-tags validations -t lab
```
Of course, that's my own env - update things according to your own usage.

You should end up with a deployed undercloud, with mistral containers having
new bind-mounts pointing to the patched versions of the code.

Now, you can test :).
