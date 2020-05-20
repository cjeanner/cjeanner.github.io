---
layout: post
title: "Tripleo-lab and the fantabulous evolution of things"
date: 2020-05-15 12:00:00 +0200
categories: openstack tripleo
---
Long time without any update right? Well. Here's something worth the wait:
tripleo-lab is now using the
[tripleo-operator-ansible](https://opendev.org/openstack/tripleo-operator-ansible),
a native collection of roles providing a nice interface for ansible.

These roles replace all the calls to the openstack CLI, allowing to get a
unified way to deploy, configure, import nodes, introspect and so on.

This change within tripleo-lab is a huge thing, since it now allows to
configure almost ANY aspect of your deploy, using the operator parameters. Of
course, some aren't directly available, since they are generated within the
lab, but you should get the proper interfaces to extend the generated things.

In order to continue using tripleo-lab, you need to get the operator collection
installed. A helper is here for you:
```Bash
$ ansible-playbook config-host.yaml -e update_operator=true
```

This will call a dedicated role within the lab that will clone the latest
version of the operators, and run the correct command to get the collection
installed in the right location (usually
```~/.ansible/collections/ansible_collections```). Once you're launched this
command, you can run your usual ansible-playbook command, with your custom env
and so on.

Also, please ensure you run it on a regular base, or set the
```update_operator``` to ```true``` in your env file - they actually do change
quite often lately, since I'm pushing new features in order to get them working
fine within tripleo-lab :). Please note this might lead in some deploy issue,
since github might have some hickups.

Since the operators provides a lot of new parameters, and some where duplicated
within the lab, a deep scrub was done in the lab parameters. All the dropped
things are properly deprecated, and a new role has been created in order to
fail the run early, showing what replaces the deprecated variable (or, well,
some were just dropped since they were useless).

This last feature ensures your environment is sane, and doesn't have any
ambiguity regarding what you're deploying.

There are also some new things regarding the proxy support: since the
```no_proxy``` variable is a bit messy (and I'm still polite here), the proxy
is only set for the package manager. This ensures you won't get any weird
issues while fetching container images, or other network resources. Fun fact to
know, there is absolutely NO RFC describing how an application should handle
any proxy related variables. To make things even funnier, some applications
seem to support the CIDR notation for no_proxy, while neither curl nor wget
appear to support it (their manpage talks about domains, no IP)... Not to
mention the length limit of the variable value. This, of course, leads to some
really fun confusion, especially when podman containers, by default, import the
proxy configuration.

Finally... CentOS-8 being out and stable enough, tripleo-lab supports it. Your
builder can be on CentOS-8, as well as the VMs. Note that you'll need to pass
a specific environment file for CentOS-8 based VMs:
```environments/vm-centos8.yaml```. This will ensure you get the right setup,
with the right size and so on.

What a bunch of new things right? :)
