---
layout: post
title: "Running kitchen-docker tests with upstart"
date: 2015-08-05 19:10:36 +0200
comments: true
categories:
  - chef
  - ruby
  - test-kitchen
  - kitchen-docker
  - docker
  - ubuntu
  - upstart
---

TL;DR:

```yml .kitchen.yml
# .kitchen.yml
---
driver:
  name: docker
  use_sudo: false              # this depends if you need to do `sudo` to run `docker` command or not
  disable_upstart: false
  image: ubuntu-upstart:14.04
  run_command: /sbin/init

platforms:
  - name: ubuntu-14.04
```

It is possible because there is this official base image specifically for upstart: https://registry.hub.docker.com/_/ubuntu-upstart/.

After making your `.kitchen.yml` look like this, just use `kitchen` as you would normally would.

Happy coding! [@waterlink000 on twitter](https://twitter.com/waterlink000).
