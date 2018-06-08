---
layout: post
title:  "TripleO Lab - next gen"
date:   2018-06-08 09:30:00 +0200
categories: openstack devstuff
---

My [tripleo-lab](https://github.com/cjeanner/tripleo-lab)
projec just saw some major improvements, as well as my Ansible
skills.

Its main purpose was to allow to deploy a full tripleo-based openstack, with
its undercloud and a bunch of VM playing the role of baremetal nodes to
provision. It worked well, and I was pretty happy with it.

But, after running it again and again in order to ensure I actually HAVE my
running tripleo, I saw some improvements to make. Like better tagging,
better grouping with blocks, and some new features.

In the current process, without any
[custom environment](https://github.com/cjeanner/tripleo-lab/tree/master/environments),
you will still get the "full set", with the preparation of about 9 VMs. Of
course, not all computers can run that kind of workload, not even my current
one.

In addition, you will find the
[tripleo-ci](https://github.com/openstack-infra/tripleo-ci) tools in the stack
home directory, as well as some utilities such as tmux and vim. In this
condition, you're ready to test your changes, run an overcloud deploy and so
on.

But, as said, you might not be able to run the complete stuff. In order to
allow more people to run this playbook, I've added a smaller environment,
starting "only" three controllers and a compute (and, of course, the
undercloud). With only 5 VMs, this setup should match almost all of the needs.

I also documented in the README how to actually create your own environment.
Of course, compared to Quickstart, it's really light. But that was the intended
goal of this tripleo-lab: be quick, light, easy. And help me learning Ansible.
And [jinja2](http://jinja.pocoo.org/).

From now on, if you want to run a small env, you just have to issue this
command:
{% highlight bash %}
ansible-playbook builder.yaml -e @environments/3ctl-1compute.yaml --tags lab
{% endhighlight %}

And if you want to upgrade to the full set, just re-run the same command
whithout the ```-e``` option.

If you don't want the tripleo-ci thingy, you can just disable it. There are
two ways for that: either exclude a tag:
{% highlight bash %}
ansible-playbook builder.yaml --tags lab --skip-tags deploy-ci
{% endhighlight %}
Or pass a custom environment file with the following content:
{% highlight yaml %}
--- no-ci.yaml
ci_tools: no
{% endhighlight %}
And in the shell:
{% highlight bash %}
ansible-playbook builder.yaml --tags lab -e @no-ci.yaml
{% endhighlight %}

More features will probably come as I'm using this tool. Now I'm just waiting
for a 
[real computer](https://www.supermicro.nl/Aplus/motherboard/EPYC7000/H11DSi.cfm)
with [some power](https://www.amd.com/en/products/cpu/amd-epyc-7401p).
