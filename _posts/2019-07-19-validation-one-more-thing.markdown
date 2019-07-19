---
layout: post
title:  "Validations: One More Thing©"
date:   2019-07-18 16:00:00 +0200
categories: openstack tripleo validations
---
So we talked about [new things in Validations]({% post_url 2019-07-18-validation-without-mistral %})
right? Well. There's one more thing that will make life easier for everyone,
and allow to get a nice job in the CI :).

We are able to override validation parameters. In a really convenient way.

For instance, let's say you want to just test a validation, on a small
undercloud node - say 2 CPU cores, and 16G of RAM.

The default values for the involved validations are 8 cores and 24G of RAM.
This means, of course, that your run will fail. But, if you want it to pass for
some reasons, you just need to push the right parameters, either directly in
the CLI, or in a JSON or YAML file. So easy!

Using the CLI, you will call:
{% highlight bash %}
source ~/stackrc
openstack tripleo validator run \
  --validation-name undercloud-cpu,undercloud-ram \
  --extra-vars '{"min_undercloud_ram_gb": 12, "min_undercloud_cpu_count": 2}'
{% endhighlight %}

This will produce the wanted output:
{% highlight bash %}
[SUCCESS] - undercloud-cpu.yaml
    Using /tmp/undercloud-cpu.yamlSvHpNAansible.cfg as config file
    Success! The validation passed for all hosts:
    * undercloud

[SUCCESS] - undercloud-ram.yaml
    Using /tmp/undercloud-ram.yamlENqqADansible.cfg as config file
    Task: Debug
    Host: undercloud
    Message: The RAM on the undercloud node is 15715 MB, the minimal recommended value is 12288 MB.
    Success! The validation passed for all hosts:
    * undercloud
{% endhighlight %}

Did I already say it was easy?

And, if you'd rather get a plain file because you have a ton of variable to
override, or just because you don't like JSON, just push a YAML file, for
instance:
{% highlight yaml %}
---
min_undercloud_cpu_count: 4
min_undercloud_ram_gb: 14
{% endhighlight %}

And run the CLI with:
{% highlight bash %}
source ~/stackrc
openstack tripleo validator run \
  --validation-name undercloud-cpu,undercloud-ram \
  --extra-vars-file ~/custom-validations.yaml
{% endhighlight %}

Note: if you want to use Mistral (```--use-mistral```), you will need to use
a JSON file anyway. And, for now, launching a validation group
(```--group```) also needs Mistral, so you will also need a JSON file.

Here's a small asciinema with the custom file in use:
[![asciicast](https://asciinema.org/a/258123.png)](https://asciinema.org/a/258123?autoplay=1)

And... That's it!
