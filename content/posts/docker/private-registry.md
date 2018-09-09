---
Categories:
- docker
Description: "Docker Private Registry"
Tags:
- docker
date: 2014-01-10
title: Docker Private Registry
---
One of the best features in [Docker](http://docker.io) is the ability to build and share images.  The public Docker index is a great place to find and share images.  However, if you want to have a private registry that is in your control (inside the firewall, etc.) this is where the [Docker Registry](https://github.com/dotcloud/docker-registry) shines.

The Docker Registry is an application that powers the public index.  The Docker team has put a lot of work into making it easy to run a standalone registry.  This gives you a complete registry in your control.

With this deployed you can have a private image store but anyone can push/pull to and from it.  To make it private, I augmented the `stackbrew/registry` image with Nginx that requires authentication.  I also added a simple Flask application that enables user management.

Currently there is an open issue [https://github.com/dotcloud/docker/pull/2687](https://github.com/dotcloud/docker/pull/2687) to allow self-signed certificates.  For now you will need a certificate issued from a standard CA.

Simple Quickstart:

* `docker pull shipyard/private-registry`
* `docker run -name registry -p 443 -v /path/to/cert_and_key:/opt/ssl -e SSL_CERT_PATH=/opt/ssl/cert.crt -e SSL_CERT_KEY_PATH=/opt/ssl/cert.key shipyard/private-registry`
* `docker port registry 443` (this will show you the mapped port)

The default username is `admin` with a password of `docker`.

Now you should be able to login to your private registry (you can set an `/etc/hosts` entry for use with your certificate - replace the port with the port from above):

`docker login https://registry.mydomain.com:49154`

Use `admin` for username and `docker` for password.  You can leave the email blank.

Once logged in you can now build an image:

`docker build -t registry.mydomain.com:49154/redis`

And then push:

`docker push registry.mydomain.com:49154/redis`

To manage users, open a browser to https://registry.mydomain.com:49154/manage.

![Docker Private Registry Management](/media/docker-private-registry-management.png)

If you have not checked out the Docker registry yet, it is a great compliment to Docker and I highly recommend.

Shipyard Private Registry: [https://github.com/shipyard/docker-private-registry](https://github.com/shipyard/docker-private-registry)

Docker Registry: [https://github.com/dotcloud/docker-registry](https://github.com/dotcloud/docker-registry)

