---
layout: post
title:  "Running validations without Mistral, and more"
date:   2019-07-18 16:00:00 +0200
categories: openstack tripleo validations
---

We've already talked about Validations, [how to run them]({% post_url 2019-04-24-validation-framework %}) and how the new CLI
will be awesome.

Now is time for some updates!

But first, some history :). The validations were mostly launched using Mistral,
an OpenStack service. Since there are talks about removing this service, we had
to think a bit about "how can we ensure we'll still be able to run them".

The current state of the Framework is: we're almost freed of Mistral!

Indeed, there's "only" one thing that needs Mistral right now: listing the
validations. The quotes are due to the fact "listing" is also used when we want
to run a validation group.

This means the following command will work, by default, without mistral:
{% highlight bash %}
source ~/stackrc
openstack tripleo validator run --validation-name undercloud-selinux-mode
{% endhighlight %}
[![asciicast](https://asciinema.org/a/257957.png)](https://asciinema.org/a/257957?autoplay=1)

And, as you can see in the asciicast, without any runing Mistral!

We're actively working on a way to get rid of Mistral for the "listing" part,
so stay tuned for more Mistral-less features :).

But. That's not the only thing we can brag about!

Having validations is fine, having a nice way to run them is good. But how can
you ensure the validations you're running are, actually, checking the things?

We have to provide a level of trust, with proof that the validations are
working as intended, so that Devs, Operators and Support can rely on them with
confidence.

This is now possible, thanks to [Molecule](https://molecule.readthedocs.io/en/stable/)
and the heavy work of my colleagues in order to integrate those tests, both in
tripleo-validations repository, but also in the CI.

Running Molecule is easy, although it might be tricky to create the test suit.
[![asciicast](https://asciinema.org/a/257994.png)](https://asciinema.org/a/257994?autoplay=1)

The next steps are obvious: finish the Mistral-less changes, and work on
unit-tests for the existing validations.

Another battle exists though: performances. We have to find a way to get a
faster run. The current validations are "simple", involving just a couple of
tasks, but are, for some reasons, really slow, at least when launched via the
CLI.

There are multiple reasons for that, one being the fact gathering. If we avoid
gathering facts for nothing, we will gain time, especially when we have
validations running against a 100+ compute infrastructure.

Another lead: the way we actually run ansible through the new "validator" CLI.

So the work is far from being over, but we're seeing massive improvments.

Stay tuned!
That's it for today!
