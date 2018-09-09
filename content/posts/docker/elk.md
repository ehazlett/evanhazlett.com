---
Categories:
- docker
Description: "Logging with ELK and Docker"
Tags:
- docker
date: 2014-11-21
title: Logging with ELK and Docker
---
Elasticsearch, Logstash and Kibana (known as the "ELK" stack) provide the ability to visualize data from any source.  This is a follow up to my previous Logstash post.  This is updated with the latest versions of the components and "dockerized" for easy use and deployment.

## Elasticsearch
Elasticsearch is a distributed, real-time, indexed datastore.  It uses Lucene at its core to provide full text search functionality.  It is schema free meaning you can throw any type of data at it.  This will be the storage service for all of our logging.

We will start a Docker container based off an Elasticsearch image I have on the Docker Hub (https://registry.hub.docker.com/u/ehazlett/elasticsearch/)

```bash
docker run -it --rm \
    -p 9200:9200 \
    -p 9300:9300 \
    --name es \
    ehazlett/elasticsearch
```

Let's break down that command.  First, the `--rm` will remove this container when it exits.  The `-p 9200:9200` and `-p 9300:9300` tells Docker to expose and bind ports 9200 and 9300 to the host on the same ports.  We need these to be known ports due to the way Kibana expects to connect to Elasticsearch.  You should now see something like the following in the console:

```
[2014-11-21 19:30:08,742][INFO ][node                     ] [Modred the Mystic] version[1.4.0], pid[1], build[bc94bd8/2014-11-05T14:26:12Z]
[2014-11-21 19:30:08,742][INFO ][node                     ] [Modred the Mystic] initializing ...
[2014-11-21 19:30:08,745][INFO ][plugins                  ] [Modred the Mystic] loaded [], sites []
[2014-11-21 19:30:10,693][INFO ][node                     ] [Modred the Mystic] initialized
[2014-11-21 19:30:10,694][INFO ][node                     ] [Modred the Mystic] starting ...
[2014-11-21 19:30:10,763][INFO ][transport                ] [Modred the Mystic] bound_address {inet[/0:0:0:0:0:0:0:0:9300]}, publish_address {inet[/172.17.0.46:9300]}
[2014-11-21 19:30:10,773][INFO ][discovery                ] [Modred the Mystic] elasticsearch/VaWr5ap8QOmLOqQKXm7U0w
[2014-11-21 19:30:14,543][INFO ][cluster.service          ] [Modred the Mystic] new_master [Modred the Mystic][VaWr5ap8QOmLOqQKXm7U0w][ed1e2f0c4776][inet[/172.17.0.46:9300]], reason: zen-disco-join (elected_as_master)
[2014-11-21 19:30:14,559][INFO ][http                     ] [Modred the Mystic] bound_address {inet[/0:0:0:0:0:0:0:0:9200]}, publish_address {inet[/172.17.0.46:9200]}
[2014-11-21 19:30:14,559][INFO ][node                     ] [Modred the Mystic] started
[2014-11-21 19:30:14,567][INFO ][gateway                  ] [Modred the Mystic] recovered [0] indices into cluster_state
```

## Logstash
Logstash is a tool for receiving and processing log data.  You can setup multiple inputs such as stdin or syslog, multiple processors such as adding fields or formatting and mutliple outputs such as stdout or elasticsearch.  We will use a simple config (that is included in the Docker image) that accepts stdin and syslog and outputs to stdout and elasticsearch.  The Docker image is on the Docker Hub (https://registry.hub.docker.com/u/ehazlett/logstash/).

We will start a Logstash container listening on port 5000 for syslog input:

```bash
docker run -it --rm \
    -p 5000:5000 \
    -p 5000:5000/udp \
    --link es:elasticsearch \
    ehazlett/logstash -f /etc/logstash.conf.sample
```

Once again, the `-p 5000:5000` and `-p 5000:5000/udp` tells Docker to listen on port 5000 for both TCP and UDP and forward to port 5000 in the container.  We are using a link to enable the Logstash container to locate the Elasticsearch container.  The `-f /etc/logstash.conf.sample` is the sample config bundled in the container.  You could easily add your own config file using volumes.

Once started you will see a couple warnings about the `tcp` and `udp` plugins using a milestone release.  These can be safely ignored.

## Kibana
Kibana is a UI that provides real-time interaction with data from Elasticsearch.  It can do time series, custom dashboards and a lot more.  We will be using the default dashboard.

In order for Kibana to work, you need to be able to access to Elasticsearch from your client (browser).  This is by design in Kibana.  For this post we will simply access Elasticsearch direct in the browser however, this is not recommended for production setups unless you have other access controls protecting your Elasticsearch instance.  Otherwise, anyone can connect to you Elasticsearch host and see you logs.

```bash
docker run -it --rm -p 80:80 ehazlett/kibana
```

This starts a container from my Kibana image (https://registry.hub.docker.com/u/ehazlett/kibana/).  This binds to your host on port 80.

You should now be able to open a browser to `http://<your-host-ip>/index.html#/dashboard/file/logstash.json` and see the Kibana default logstash dashboard:

![Kibana](/media/kibana.png)

## Logs
Now that you have an "ELK" stack deployed, you can send logs to it.  Any mechanism that sends logs to syslog can use this.  For example in Debian/Ubuntu, you can configure `rsyslog` to forward logs to Logstash by adding `*.*   @@<your-host-ip>:5000` in `/etc/rsyslog.conf`.  Make sure to restart `rsyslog`.  You should start seeing logs in Kibana.

## Docker
For an added bonus, we can easily add centralized logging for Docker containers.  Jeff Linday's wonderful "logspout" application (https://github.com/progrium/logspout) can ship logs to a variety of facilities including syslog:

```bash
docker run --rm \
    -v /var/run/docker.sock:/tmp/docker.sock \
    progrium/logspout syslog://<host-ip>:5000
```

You can now run a test container like so:

```bash
docker run --rm debian:jessie apt-get update
```

You should see the output in Kibana.

![Kibana Logs](/media/kibana-logs.png)

