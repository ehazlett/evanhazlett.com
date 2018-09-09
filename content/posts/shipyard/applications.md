---
Categories:
- shipyard
Description: "Announcing Shipyard Applications"
Tags:
- shipyard
date: 2013-07-09
title: "Shipyard: Applications"
---
The next major feature for [Shipyard](https://github.com/ehazlett/shipyard) is "applications".  These are a collection of containers that are accessible by a domain name.  Shipyard uses [Hipache](https://github.com/dotcloud/hipache) for the frontend load balancer.

Using Shipyard, you can launch a set of containers that expose a backend port.  You can then create an "application" which configures Hipache and routes traffic to all containers in that app on the container backend port.  Configuration is automatic and you can have multiple containers across multiple hosts and Hipache will balance and direct traffic to them.

### Deeper Dive
Under the covers, Shipyard contains the configuration for the application.  Upon attaching a container to the application, Shipyard will queue a task to update the frontend mapping in Redis which Hipache uses.  The configuration will also be updated upon container restart since the port(s) can change.

Upcoming plans for Shipyard include stats collection.  These will collect the number of requests, response times, etc. from Hipache for applications.  With this, we can do things like autoscaling.  Lots of stuff planned -- join in :)

### More

Read more on applications [here](https://github.com/ehazlett/shipyard/wiki/Applications) and source is [https://github.com/ehazlett/shipyard](https://github.com/ehazlett/shipyard)

For a screencast showing applications, visit [https://www.youtube.com/watch?v=pLX3QF17Sj0](https://www.youtube.com/watch?v=pLX3QF17Sj0).
