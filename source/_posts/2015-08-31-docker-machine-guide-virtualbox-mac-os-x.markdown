---
layout: post
title: "Docker Machine guide (VirtualBox on Mac OS X)"
date: 2015-08-31 10:21:06 +0200
comments: true
categories:
- docker
- docker-machine
- guide
- virtualbox
- macosx
---

*(VirtualBox on Mac OS X)*

This guide is a combination of official docs and usage experience.

## Installation

*Install virtualbox first
[virtualbox downloads](https://www.virtualbox.org/wiki/Downloads).*

Look at this page: https://github.com/docker/machine/releases/ and pick latest
release. At the time of writing this is `v0.4.1`.

Now, assign version number to an environment variable, together with your
architecture:

```bash
DOCKER_MACHINE_VERSION=v0.4.1
DOCKER_MACHINE_ARCH=darwin-amd64
```

Now download `docker-machine` binary and put it onto your `PATH` (recommended
is `~/bin/`):

```bash
mkdir -p ~/bin
URL=https://github.com/docker/machine/releases/download/${DOCKER_MACHINE_VERSION}/docker-machine_${DOCKER_MACHINE_ARCH}
OUTPUT=~/bin/docker-machine
curl -L ${URL} > ${OUTPUT}
chmod +x ${OUTPUT}
```

If you still haven't yet, put `~/bin/` on your `PATH` with `export
PATH=$PATH:$HOME/bin` into your `.bashrc` or `.zshrc` (or whatever shell you
use).

## Installing docker client

```bash
URL=https://get.docker.com/builds/Darwin/x86_64/docker-latest
OUTPUT=~/bin/docker
curl -L ${URL} > ${OUTPUT}
chmod +x ${OUTPUT}
```

## Creating your first docker machine

```bash
docker-machine create -d virtualbox dev
```

This is how you create docker machine with name `dev` having virtualbox as a
backend.

But after some time you will encounter a problem of running out of memory. So
my recommended command to create your primary development docker machine is
this:

```bash
docker-machine create -d virtualbox dev --virtualbox-memory "5120"
```

This will create virtualbox VM with enough memory to run low-to-moderate size
clusters with `docker-compose`. Which should be enough for development.

This should be your primary docker machine that is always activated and used.
There is no need to destroy and re-create this `dev` machine unless you are
testing some edge-cases. And better to use additional docker machine with
different name for this.

## Connecting to your `dev` docker machine

```bash
eval $(docker-machine env dev)
```

After this command, you will have everything you need to run `docker` in the same terminal:

```bash
# Using alpine image here because it is only ~5 MB
docker run -it --rm alpine echo 'hello world'
```

You should see:

```
# .. pulling alpine image here .. and:
hello world
```

It might be annoying to run `eval $(docker-machine env dev)` each time you open
new terminal. So feel free to put this line into your `.bashrc` or `.zshrc` or
whatever shell you use:

```bash
# in .bashrc:
eval $(docker-machine env dev)
```

If you have just powered on your Mac (or just stopped your docker machine) you
will experience this error:

```
Error: Host does not exist: dev
```

In that case just start it with:

```bash
docker-machine start dev
```

And re-open your terminal.

## Dealing with docker machine's IP address

Fact: docker machine's IP address stays the same, usually 192.168.99.100,
unless:

- you destroy your docker machine `dev`, create another VirtualBox VM and
  create docker machine `dev` afterwards, or
- you have custom VirtualBox configuration.

Given that docker machine's IP address stays the same or changes very rarely,
you can simply put its IP address in your `/etc/hosts`.

First, figure out current docker machine's IP address:

```bash
docker-machine ip dev
```

And put it in `/etc/hosts`:

```
# in /etc/hosts

# Put IP address previous command has returned here:
192.168.99.100 docker-dev
```

To test that it works correclty try to run:

```bash
docker run -it --rm -p 80:80 nginx
```

And now reach `http://docker-dev` in your browser - you should see default
Nginx page.

If you want to refer docker machine `dev` in your scripts, it is better to use
`$(docker-machine ip dev)` capabilities for that. For example, `curl`-ing the
page we have seen in browser just now:

```bash
curl $(docker-machine ip dev)
```

For teams it would make sense to agree on the same name for primary development
docker machine. `dev` works just great!

*NOTE: personally, I use both `docker-dev` and just `dev` as a hostname to type
less, but that might clash with something else, so `docker-dev` it is.*

## Upgrading

To upgrade `docker-machine` or `docker` binaries, just follow `Installation`
instructions again.

To upgrade `docker` server inside of already running docker machine, use:

```bash
docker-machine upgrade dev
```

This will update to the latest version of `docker` server and `boot2docker`
image.

## Re-creating a fresh `dev` docker machine

```bash
docker-machine rm dev
docker-machine create -d virtualbox dev --virtualbox-memory "5120"
```

## Further reading

- [Overview of Docker Machine](https://docs.docker.com/machine/)
- [Get started with Docker Machine and a local VM](https://docs.docker.com/machine/get-started/)

Happy hacking! [@waterlink000 on twitter](https://twitter.com/waterlink000).
