---
title: My Raspberry Pi cluster project
description: Raspberry Pi cluster lessons learned
author: shawn.sesna@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - 
---

Like everyone else, we were told to stay home as this Covid-19 crisis unfolds.  Seeing as how I was going to be stuck at home, I figured I'd do something constructive with my time ... along with a fair amount of gaming ;)  I'd always wanted to create a cluster, but my other half didn't like the idea of having yet even more computers around the house (we're already at ~2:1 ratio of computers to people).  I'd read about people having great success with running the new Raspberry Pi 4 in a cluster with Docker Swarm.  Since the Raspberry Pi is tiny and consumes very little power, the Better half agreed, albeit reluctantly (love you, sweetie!)  Here are some of my experiences.

## Parts
No article about projects such as this would be complete without a parts list :)

- [Raspberry Pi 4 4GB](https://www.amazon.com/gp/product/B07TC2BK1X/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1) (I bought 5)
- [Heatsinks](https://www.amazon.com/gp/product/B082RKKQ2D/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
- [USB-C charge cables](https://www.amazon.com/gp/product/B01I4ZOIQY/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
- [MicroSD cards](https://www.amazon.com/gp/product/B07N73LB4T/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1)
- [USB wall charger](https://www.amazon.com/gp/product/B00YRYS4T4/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1)
- [Short network calbes](https://www.amazon.com/gp/product/B0721RFHT8/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1)
- [Case](https://www.amazon.com/gp/product/B07D5MJ7PQ/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1)

I chose the C4Labs Cloudlet case because it allowed me to access a single Pi without having to disassemble the whole case to get to it like the stackable cases.  The original design of the Cloudlet case has the fans powered by the Pi's.  I didn't want to have to worry about disconnecting cables if I needed to remove a Pi, so I went with the following parts to power the fans

- [Fan power splitter](https://www.amazon.com/gp/product/B082H6D611/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1)
- [3/4 pin to USB converter](https://www.amazon.com/gp/product/B07FNKPPT2/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1)
- [USB extension cable](https://www.amazon.com/gp/product/B00NH13UFQ/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1)

The Cloudlet case has plenty of room for these extra parts.

Lastly you'll need a network switch to connect the Pis to.  I had a spare 8-port lying around so I didn't need to buy another.  The Raspberry Pi 4 comes with a wireless adapter built-in, so you could utilize that as well.

## Creating the cluster
There are tons of blog posts out there that show you how to create a cluster of Raspberry Pis with Docker Swarm.  I found https://howchoo.com/g/njy4zdm3mwy/how-to-run-a-raspberry-pi-cluster-with-docker-swarm easy to follow.  All of the details you need are in the blog post, I won't bore you with all of that jazz :)

## Operating the Swarm
I had played around with Kubernetes (K8s) and Docker before taking on this project, however, I'd never run a Docker Swarm before.  There was some new concepts to learn when running things in a Docker Swarm mode versus a stand-alone Docker instance.  For example, where you would normally use a `docker run` statement, you'd create a service within the swarm to run the container using `docker service create`.  After messing around for a while, I had a fully operational Swarm!

![](docker-swarm.png)

## The lessons
Though it takes very little effort to get your swarm up and running, there were some lessons I learned, some good, some bad.

### Not all containers run on ARM
This lesson was one of the bad ones.   My original intention for this project was to lessen the burden on my VM Hypervisor by running containers.  When searching [Docker Hub](https://hub.docker.com), I found that very few official images would run on the ARM architecture.  I could usually find an image that someone made to run on the Raspberry Pi, but they were usually old and not the current version.

In some cases you can use a work-around and tell Docker not to resolve the image to an architecture.
- `docker service create --name <somename> --no-resolve-image <image>`
- `docker stack deploy --name <somename> --compose-file <somefile> --resolve-image never`

### Stacks!
Stacks are the Docker Swarm way of using docker-compose, in fact it uses a compose file as one of its arguments.  Just like docker-compose, you can define multiple containers within a single file.  By default, a stack deployment will create a network for the stack so they can all talk to each other.  This makes it easy to refer one container to another just by name.  For example, if you created a container that needs a database container, you could simply refer to it by name!  For example, below us a compose file for running [Home Assistant](https://www.home-assistant.io/) on my docker swarm.  This stack consistes of an MQTT service container, a Home assistant container, and a MySQL database server container.  For reasons I'll go into later, I configured Home Assistant to use a MySQL back-end for the Recorder

```
version: "3.3"

services:

  mqtt:
    image: eclipse-mosquitto
    networks:
      - hass
    ports:
      - 1883:1883
    volumes:
      - /mnt/clusterpi/mqtt/db:/db
      - /etc/localtime:/etc/localtime:ro

  home-assistant:
    image: homeassistant/home-assistant
    networks:
      - hass
    ports:
      - 8123:8123
    volumes:
      - /mnt/clusterpi/home-assistant:/config
      - /etc/localtime:/etc/localtime:ro

  mysqldb:
    image: hypriot/rpi-mysql
    networks:
      - hass
    ports:
      - 3350:3306
    environment:
      MYSQL_ROOT_PASSWORD: "MySuperSecretPassword"
      MYSQL_DATABASE: "homeassistant"
      MYSQL_USER: "hassio"
      MYSQL_PASSWORD: "AnotherSuperSecretPassword"
    volumes:
      - /mnt/clusterpi/mysql:/var/lib/mysql

networks:
  hass:
```

In the `configuration.yaml` file, you can see where I created the connection to the mysqldb container

```

# Configure a default setup of Home Assistant (frontend, api, etc)
default_config:

# Uncomment this if you are using SSL/TLS, running in Docker container, etc.
# http:
#   base_url: example.duckdns.org:8123

# Text to speech
tts:
  - platform: google_translate

group: !include groups.yaml
automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml
recorder:
  db_url: mysql://root:MySuperSecretPassword@mysqldb/homeassistant?charset=utf8
```

### One of us!
When working with a docker swarm, all of the members of the swarm act as one.  Looking at the graphic above, we can see that all of the containers are distrubted amongts the members of the swarm.  However, when connecting to a container via an exposed port, it's not necessary to reference the node in which is currently hosting the container.  For example, the graphic above shows the members of the cluster with the unimaginative names clusterpi-1 through 5.  We can see that clusterpi-1 hosts the container named `viz` (which is the web page of the graphic, `visualizer`)  The `viz` container port is mapped to the host port of 80, so I can access the `visualizer` web page via http://clusterpi-1.  I can also access `visualizer` via http://clusterpi-4, even though clusterpi-4 is not the current host of the container.  Go cluster!  Just like a single system, any port that has been mapped cannot be used by another container, meaning the mysql_dev container is the only thing that can be mapped to 3306, all of the other mysql containers (unless they're replicas of mysql_dev) have to use a different port.

### Persistent storage woes
One of the first lessons you learn when playing with containers is that when the container is destroyed, so is all of the data.  In order to prevent this, you need to configure the container to map what it would usually store internally to a place external to the container in what is known as a volume.  Creating volumes on a docker host is fairly easy, however, there's no gauruntee that a container will be run by the same host each time (okay, technically you could do it with constraints), persistent storage on a swarm needs to be in a location that is available to all of the nodes.

Creating volume mappings are not particularly difficult ... unless you're mixing Windows and Linux.  