---
layout: post
title:  "Accessing private stuff from your build server"
date:   2018-08-07 13:00:00 +0200
categories: openstack tripleo
---

While working on the validations/tests for the new release, I had the need to
access specific packages and repositories behind a VPN.

Some corporate VPN don't allow to have multiple connection for the same user
(confession: I didn't test it ;)), meaning your home build server can't access
the private resources while your workstation is connected.

There are two ways: either build some gateway on your own, put your corporate
VPN on it, and put your work machines behind. That can be a pain, especially
since VPN eats a lot of resources, and in some cases, you will need to access
the gateway and update the password everytime you reboot it.

The other one is kind of more funny: just use squid.

The setup is really easy:

* setup squid on your workstation
* allow only your LAN to access this service
* allow port 3128/tcp
* configure your buildserver VMs to use that proxy

In TripleO environment, this means mainly: configure the undercloud to use your
squid as a proxy for Yum and Docker.

In order to do that, you need to push a simple line in /etc/yum.conf:

```
proxy = http://<PROXY_HOST>:3128
```

`PROXY_HOST` being your workstation IP address. In addition, you will need to push a
configuration file for docker. The easiest way is to push something in systemd
directly.

If you create a `/etc/systemd/system/docker.service.d/http-proxy.conf` file with
the following content, you should be fine:

```
[Service]
Environment="HTTP_PROXY=http://<PROXY_HOST>:3128/"
Environment="NO_PROXY=localhost,127.0.0.0/8"
```

In case you do that after docker has been deployed, you will need to reload
the systemd daemon and restart docker:

```
systemctl daemon-reload
systemctl restart docker
```

Once this is done, you will be able to access the private things behind the
corporate VPN without the need of running the VPN on you build machine.

This also is more secure, especially if you ensure the squid port is locked
by default, and open it only when you actually NEED it.

On my own, I just run the following command in order to open it on the workstation:

```
firewall-cmd --zone public --add-port 3128/tcp
```

Note: there isn't the "--permanent" option, meaning next reboot or firewall
reload will drop the rule. In addition, ensuring your squid.conf only allows
your LAN to connect is a good thing.

In addition, my squid.conf also ignore repodata files in order to avoid bad
caching:

```
acl repodata url_regex .*repodata.*
cache deny repodata
```

As a final note: squid is a powerful tool, not only for caching. In fact, if
you must access TLS encrypted URI, squid won't cache anything (by default it
doesn't do TLS-MitM), but at least it will allow to access remote, protected
contents in an easy way.

The fact it doesn't cache TLS URI will prevent most of the caching, especially
the containers layers - this is a good thing in order to prevent disk usage
issues on your computer ;).
