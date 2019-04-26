---
layout: post
title:  "Validation Framework: validate your TripleO env!"
date:   2019-04-24 12:00:00 +0200
categories: openstack tripleo validations
---
I'm currently working with a great team on a new thing: the Validation Framework.

This new feature will take over the whole validations we might want to run
before, during and after a deploy, update or upgrade task.

Background
----------
Currently, the validations are available either through the UI, or through Mistral
calls. There are two issues with this approach: the UI is deprecated and will
be removed soon, and Mistral isn't always available.

Providing a way to run validations on their own is a must, since running them
allows to:
  * ensure we have the right resources available
  * ensure services are running as expected
  * ensure services are answering as expected
  * provide a good overview of the cluster state

Mistral vs Ansible
------------------
As just said, the current way doesn't allow to run the validations in a simple
way if you don't have Mistral.

For instance, you're currently unable to validate your undercloud node before
deploying anything. So you will already lose time in order to get the whole
TripleO tools installed, and then you'll need to tweak things in order to
actually be able to run the validations.

You're also unable to run any validations if you're deploying a Standalone
instance, since it doesn't have the Mistral thingy installed.

Using the new framework allows to get a nice way to run things whenever you
want.

For example, if you're wanting to validate the node you want to use as an
undercloud before doing anything, you will just need to install git, ansible,
and run 4 commands:
```Bash
yum install -y git ansible
git clone https://opendev.org/openstack/tripleo-validations
cd tripleo-validations
IP=$(ip r get 8.8.8.8 | awk '/src/{print $7}')
printf "[undercloud]\n$IP\n" > hosts
for i in $(grep -l '^\s\+-\s\+prep' -r validations);
  do echo $i;
  ansible-playbook -i hosts $i;
done
```

[![asciicast](https://asciinema.org/a/242562.png)](https://asciinema.org/a/242562?autoplay=1)

Running Validations through the new CLI
---------------------------------------
The Framework also includes a new CLI option, ```validator```. For now it
only supports Mistral, but we aim to enable plain ansible run in case of either
broken or absent Mistral (or if the operator wants plain ansible).

This new CLI allows to list and run validations, either by name or group:
```Bash
source ~/stackrc
openstack tripleo validator list
openstack tripleo validator run --validation-name validation1[,validation2,...]
openstack tripleo validator run --group validation-group
```

[![asciicast](https://asciinema.org/a/242381.png)](https://asciinema.org/a/242381?autoplay=1)

Running validations using plain Ansible (bis)
---------------------------------------------
For now, if you want to run the validations through plain Ansible, you have to
tweak things a bit.

First, create a "run-validations.sh" script:
```Bash
#!/bin/bash
# IF running on Undercloud
source /home/stack/stackrc
# IF running on standalone, replace by
# export OS_CLOUD=standalone

VALIDATIONS_BASEDIR="/usr/share/openstack-tripleo-validations"

# Use custom validation-specific formatter
export ANSIBLE_STDOUT_CALLBACK=validation_output
# Disable retry files to avoid messages like this:
# [Errno 13] Permission denied:
# u'/usr/share/openstack-tripleo-validations/validations/*.retry'
export ANSIBLE_RETRY_FILES_ENABLED=false
export ANSIBLE_KEEP_REMOTE_FILES=1

export ANSIBLE_CALLBACK_PLUGINS="${VALIDATIONS_BASEDIR}/callback_plugins"
export ANSIBLE_ROLES_PATH="${VALIDATIONS_BASEDIR}/roles"
export ANSIBLE_LOOKUP_PLUGINS="${VALIDATIONS_BASEDIR}/lookup_plugins"
export ANSIBLE_LIBRARY="${VALIDATIONS_BASEDIR}/library"

# IF running on Undercloud
ANSIBLE_INVENTORY_BIN=$(which tripleo-ansible-inventory)
export ANSIBLE_INVENTORY=${ANSIBLE_INVENTORY_BIN}
# IF running on standalone, create a "hosts" file with mandatory [undercloud]
# entry, and pass it in the ANSIBLE_INVENTORY

VALIDATION="${1:-undercloud-validate.yaml}"

ansible-playbook ${VALIDATION}

```

Then, create your playbook, adding the roles you want to run on the node, for
instance:
```Bash
---
- hosts: undercloud
  vars:
    container_cli: podman
  roles:
    - dns
    - undercloud-cpu
    - undercloud-disk-space
    - undercloud-heat-purge-deleted
    - undercloud-process-count
    - undercloud-ram
    - undercloud-selinux-mode
    - undercloud-service-status
```

And Voilà. You have your validation playbook ready to fire!

[![asciicast](https://asciinema.org/a/242520.png)](https://asciinema.org/a/242520?autoplay=1)
