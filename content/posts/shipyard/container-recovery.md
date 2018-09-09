---
Categories:
- shipyard
Description: ""
Tags:
- shipyard
date: 2013-09-22
title: "Shipyard: Container Recovery"
---
Continuing our push towards production grade Docker management, the latest feature released is container recovery.  This allows a container to be marked as "protected" and Shipyard will start monitoring it for failures.  Enabling recovery is as simple as a toggle in the container details.  Here is how it works.

Shipyard uses a queue for background tasks.  With container recovery, a scheduled task runs on a specified interval that monitors the protected containers.  On a side note, we originally used the Python RQ library for tasks but we ran into issues with scheduled tasks and the RQ scheduler.   We decided to switch to Celery but still keep the same Redis backend.  If a protected container exits, stops, is destroyed, etc. Shipyard will launch a new container using the exact same parameters.  It does this by using the cached copy of the container metadata.  To prevent recovery loops, there is a check to see how many times a container has recovered.  If this passes the specified setting, an exception is raised and Shipyard stops attempting recovery.

The recovery alone keeps standalone containers available by re-launching with the same port specs.  However, when used with Shipyard applications it allows a public facing container group to maintain service and heal on failure.

We are pushing hard and committing time and resources from Arcus to drive development.  If you would like to see a feature or improvement please submit an issue or jump on IRC.  And as always, we would love to hear feedback :)
