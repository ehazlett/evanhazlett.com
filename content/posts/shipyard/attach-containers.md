---
Categories:
- shipyard
Description: ""
Tags:
- shipyard
date: 2013-07-12
title: "Shipyard: Container Attaching"
---
One of the great features of [Docker](http://docker.io) is the ability to run interactive containers.
You can attach to a container which connects you to a terminal.  From there it is just
like any other virtual machine or instance.  I have wanted something like this in
Shipyard since I started it, but it was on the back burner.  Until a few days ago.

[ukd1](https://github.com/ukd1) had mentioned in #docker that they were building a
web based terminal utilizing websockets to attach to containers from the browser.
ukd1 and their team worked on it that afternoon and by the evening there was a pull
request for Shipyard.

In order for it to work, Docker needs websocket support in the api.  There is a pull
request open but has not been merged as of this writing.  To get it working however,
is rather simple:

Clone the latest Docker and checkout the latest stable tag (currently v0.4.8):

    ```
git clone https://github.com/dotcloud/docker.git
```

```
git checkout v0.4.8
```

Add the upstream from the pull request [dotcloud/docker#1146](https://github.com/dotcloud/docker/issues/1146):

```
git remote add ws https://github.com/benoitc/docker.git
```

Create a websocket branch (optional):

    ```
git checkout -b websockets
```

Merge code from pull request:

```
git pull ws feature/attach_ws
```

Build new Docker binary:

```
make
```

Now you can run `bin/docker` and use Shipyard to access the console via websockets.

The console support is still in branch (`terminal`) in the Shipyard repository until the
1146 pull request gets merged and released as stable in Docker.

Here is a quick screencast showing the feature: [http://www.youtube.com/watch?v=aLlp-kemx_s](http://www.youtube.com/watch?v=aLlp-kemx_s)
