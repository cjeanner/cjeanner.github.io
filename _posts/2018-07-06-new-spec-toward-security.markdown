---
layout: post
title:  "New spec for TripleO: integrate oslo.privsep in tripleoclient"
date:   2018-07-06 10:30:00 +0200
categories: openstack devstuff tripleo
---

Security is an important thing. Especially nowadays, when you can read, hear
tons of companies and entities getting hacked in funny ways.

Currently, the TripleO project has some issues with too widely opened sudo
rules. They allow standard users to basically own the whole node without any
password.

Of course, you'll still need an access (remote or local) to the node in order
to profit from the sudo. But it might be triggered without a shell, if some
service using one of those power users have a flaw.

This can't stay as is, and should be corrected, of course. Security is also
important in Open Source projects, especially an attack surface as big as
OpenStack.

In order to correct that bad situation, two specifications are currently under
review, and are to be implemented in Stein release (due for next year).

The [first one](https://review.openstack.org/580033) aims python scripts.
Currently, at least in python-tripleoclient, there are
[wild calls to sudo](https://github.com/openstack/python-tripleoclient/blob/a26dfc0c9f0c10ae5af1c453ed57ce4752e6c9bd/tripleoclient/v1/undercloud_config.py#L571)
in order to call itself. This is one example among others, and it's pretty ugly.

This kind of calls also creates some issue with the rights of generated files,
and leads to other [sudo calls](https://review.openstack.org/580354) we don't
want to see.

The solution isn't easy though. In order to prevent any ```NOPASSWD:ALL``` in
the sudoers, we'll need to refactor the code, and introduce
[oslo.privsep](https://github.com/openstack/oslo.privsep), a python library
aiming a secure and clear privilege escalation, with proper filters and the
like.

Basically, in the tripleoclient case, we will need to replace all the
```os.chown``` calls by plain ```rootwrap``` subprocess calls. The
```os.chown``` isn't the only one, there are most probably other things that
will need to be corrected and correctly isolated. And as
[some services are launched as another user](https://github.com/openstack/python-tripleoclient/blob/9d89001cea6cd9423c03a495510f4d1c594eac41/tripleoclient/v1/tripleo_deploy.py#L380)
during the deploy process, we will also need to take those kind of calls out,
and push them in a separated script, called with the ```rootwrap``` instead.

The advantages of using ```oslo.privsep``` are multiple:
- generic library embedded in oslo, meaning good code convergence
- better filtering than plain sudo
- good isolation
- clean separation of the application code and the privilege escalation
- young project, meaning we can build best practices based on real experience

The [second proposal](https://review.openstack.org/572760) aims plain ```sudo```
calls from shell script and ansible. It will not require any code refactoring,
but more a log review in order to catch all the reall sudo needs, and list them
in the ```sudoers.d/user``` file.

There again, the goal is to prevent any unwanted privilege escalation via
passwordless ```sudo``` calls.

In the end, when both those proposals are accepted and implemented, we will end
up with a more secure TripleO deployment, more secure nodes, and less issues
with people refusing to deploy OpenStack due to those bad sudo practices.

So, if you're interested in Security and OpenStack, you definitively will review
those two proposals and take part in this revolution.
