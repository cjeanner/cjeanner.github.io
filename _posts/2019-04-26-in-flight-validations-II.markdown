---
layout: post
title:  "In-flight Validations II"
date:   2019-04-26 12:00:00 +0200
categories: openstack tripleo validations
---

Here's a quick demo for the in-flight validations, with some (edited) cast!

As [previously stated]({% post_url 2019-04-25-in-flight-validations %}), being
able to call validations during the deploy/update itself provides a quick way
to get early failures, avoiding head scratching and time loss.

This quick demo shows how it can be done easily, with a real validations. It
uses the (hopefully) [soon-to-be merged new "image-serve" validation](https://review.opendev.org/655708)
and calls it just after the service is configured.

Doing so allows to ensure the configuration is actually working fine. In this
demo, the httpd service is stopped before calling the validation, in order to
show the early failure occuring even before we actually need that service.

Preparation
-----------
You need to build a ```tripleo-validations``` package with the new validation.
You can do so using the [tripleo-lab]({% post_url 2018-06-08-tripleo-lab %}).

Once you have built and installed the package, you need to edit
tripleo-heat-templates content, in our case:
```
sudo vim /usr/share/openstack-tripleo-heat-templates/deployment/image-serve/image-serve-baremetal-ansible.yaml
```
Go to the ```host_prep_tasks``` section, and, at the end of the *Install,
Configure and Run Apache to serve container images* block, insert this:
```
          - name: DEMO - stop httpd
            service:
              name: httpd
              state: stopped
          - include_role:
              role: image-serve
```
Of course, the *DEMO - stop httpd* should not be added on the prod, since it
will make the validation fail ;). This entry is only for the demo effect.

Save the edited file, and... Well. That's it. You have just added a simple
validation that will ensure the container image registry is working as expected!

And, after so many words, here's the
[promised cast](https://asciinema.org/a/242942)!
[![asciicast](https://asciinema.org/a/242942.png)](https://asciinema.org/a/242942)

Do you validate this feature/content? ;)
