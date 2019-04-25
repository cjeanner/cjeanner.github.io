---
layout: post
title:  "In-flight Validations"
date:   2019-04-25 12:00:00 +0200
categories: openstack tripleo validations
---

We've seen in [the previous post]({% post_url 2019-04-24-validation-framework %})
how the Validation Framework will help getting the whole TripleO deploy more
stable. I've shown how running the validations before and after a deploy is
easy - but that's not all we can do.

Lately, I've also worked on the so-called "in-flight validations" - a way to
run validations (being from the Framework or not) *during* the run.

This provides multiple advantages:
- early failure
- ensuring things are in place before going forward
- provides clear outputs in case of something's missing or crashed

This quick example shows how we can use the already existing health checks
directly inside the deploy - doing so ensures we have a working service.

Is Horizon working?
-------------------
Take the Horizon service. It's an easy one, with only one template, one
container, and a simple deploy path.

Opening ```deployment/horizon/horizon-container-puppet.yaml```, you need to
add a new entry in the ```output```:
```YAML
      deploy_steps_tasks:
        - name: ensure horizon is running
          when: step|int == 4
          shell: |
            podman exec -u root horizon /usr/share/openstack-tripleo-common/healthcheck/horizon
```
You can add it wherever you want, for instance right before the
```# BEGIN DOCKER SETTINGS``` comment.

Some explanations:

The deploy_steps_tasks is a "new" (not THAT new though) task list running on
the host directly. Using the ```when``` condition, you can ensure it's launched
at the right step - for instance, since Horizon container is deployed at step 3,
we want to ensure it's running OK at step 4.

We can, of course, inject some other kind of validations - for instance, we
can call the roles provided by the ```tripleo-validations``` package, the very
same providing all the existing validations for the Validation Framework.

Also, instead of hard-coding the "podman" call, we should use the
```ContainerCli``` used in the tripleo-heat-templates. Of course, keeping clean
code is as important as being able to test the deploy ;).

Make it crash!
--------------
The above example should succeed on every deploy. If you want to see how adding
in-flight validation make it crash early, you can edit the command and set it
to:
```
podman exec -u root horizon /usr/share/openstack-tripleo-common/healthcheck/glance-api
```
Doing so will make the whole deploy crash at step 4, with the following message:
```
TASK [ensure horizon is running] *************************************************************************************************************************************************************************************************************$
fatal: [undercloud]: FAILED! => {
  "changed": true,
  "cmd": "podman exec -u root horizon /usr/share/openstack-tripleo-common/healthcheck/glance-api",
  "delta": "0:00:00.324740",
  "end": "2019-04-25 09:56:40.100641",
  "msg": "non-zero return cod$",
  "rc": 1,
  "start": "2019-04-25 09:56:39.775901",
  "stderr": "curl: (7) Failed connect to 127.0.0.1:9292; Connection refused\nError: exit status 1",
  "stderr_lines": [
    "curl: (7) Failed connect to 127.0.0.1:9292; Connection refused",
    "Error$ exit status 1"
  ],
  "stdout": "\n000 127.0.0.1:9292 0.001 seconds",
  "stdout_lines": ["", "000 127.0.0.1:9292 0.001 seconds"]
}
```

Which is perfect: since Horizon isn't working, we don't need to wait until the
end of the deploy in order to detect it. And we even get a nice error message :).

Using "real" validations from the Framework
-------------------------------------------
In order to call a role from the Framework, you'll need to use the
```include_role``` ansible module, and provide mandatory variables if any.

You have to include it in the ```deploy_steps_tasks``` entry, and... Well.
That's pretty all in fact :).

Final words
-----------
Deploying is a long process. Sometimes it fails, and it might be hard to find
out the root cause of the failure. Messages aren't always helpful, and we might
have to search among a lot of different log files, with a lot of "acceptable
failures" being ignored.

Using in-flight validations, being either simple health check calls or deeper
checks/validations can help the operator as well as the developers to find
and understand the issue. It also can prevent a huge time loss, especially for
services that aren't used during the deploy itself - we will see them as crashed
only at the end of the 5 steps + post-deploy tasks. Meaning "a fair amount of
time".

Make life easier, make validations!
