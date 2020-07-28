---
author: "Stephan Lichtenauer"
title: Git (Nomad)
summary: This is a Git server jail that can be deployed via nomad.
tags: ["git", "nomad"]
---

# Overview

This is a Git jail that can be started with ```pot``` but it can also be deployed via ```nomad```.

The git user home directory is in ```/var/db/git``` where also a directory ```.ssh``` for authorized key access resides.    
It is suggested that this directory is mounted from outside the jail when it is run as a ```nomad``` task so that it is persistent (see example below).

Git is started as blocking task when the jail is started (see ```git-nomad+4.sh```).

For more details about ```nomad```images, see [about potluck](https://potluck.honeyguide.net/micro/about-potluck/).

# Nomad Job Description Example

It is suggested to mount the jail directory ```/var/db/git``` from outside as this contains the git database and the ```.ssh/authorized_keys``` file for access:

```
job "examplegit" {
  datacenters = ["datacenter"]
  type        = "service"

  group "group1" {
    count = 1 

    task "www1" {
      driver = "pot"

      service {
        tags = ["git"]
        name = "git-example-service"
        port = "ssh"

         check {
            type     = "tcp"
            name     = "tcp"
            interval = "60s"
            timeout  = "30s"
          }
      }

      config {
        image = "https://potluck.honeyguide.net/git-nomad"
        pot = "git-nomad-amd64-12_1"
        tag = "1.0"
        command = "/usr/local/bin/cook"
        args = [""]

       mount = [
         "/mnt/s3/web/git:/var/db/git"
       ]
        port_map = {
          ssh = "22"
        }
      }

      resources {
        cpu = 200
        memory = 64

        network {
          mbits = 10
          port "ssh" {}
        }
      }
    }
  }
}
```