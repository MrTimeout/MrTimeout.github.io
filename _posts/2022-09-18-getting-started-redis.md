---
title: Redis Getting Started
author:
  name: MrTimeout
  link: https://github.com/MrTimeout
date: 2019-08-09 20:55:00 +0800
categories: [Database, Redis]
tags: [Redis, Database, Getting Started]
---

This post will guide you how to get your hands on Redis. If you are a totally beginner, this can help you to start.
I am a totally beginner using this technology, so contact me if there is something wrong with the blog or you want to connect.
I am creating a series of Redis to learn more about it. Happy coding.

## What is Redis?

[Redis](https://redis.io/) is an open source, _in-memory data store_ used by millions of developers as a _database_, _cache_, _streaming engine_, and 
_message broker_

## Install

We are going to use [Docker](https://docs.docker.com/engine/install/ubuntu/) because it is really simple to get start and running.

### Three lines

```sh
REDIS_ID=$(docker container run --publish 6379:6379 --rm --detach redis:latest)
docker container ls --all --format "{{.Name}}" $REDIS_ID
docker container exec -it $REDIS_ID redis-cli
```

### One liner

```sh
docker container exec -it \
  $(docker container run --publish 6379:6379 --rm --detach redis:latest) redis-cli
```

## Data types

In redis, we have various data types to play with:

- string
- hash
- list
- set
- ordered set
- streams
- geospatial
- hyperLogLog
- Bitmaps
- BitFields

Well, it sounds interesting, but less talk and more action.