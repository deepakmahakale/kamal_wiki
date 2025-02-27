---
layout: single
title: How to rollback a deployment with Kamal?
description: &description >
  Learn how to rollback a deployment in Kamal using the kamal rollback command.
  This guide covers finding the right version, executing the rollback, and verifying successful rollback.
  Ensure your Rails app stays stable - read more!
excerpt: *description
date: 2025-02-17
tags:
  - Rails
  - Rollback
header:
  og_image: /assets/images/opengraph/2025-02-17-how-to-rollback-a-deployment-with-kamal.png
---

Kamal provides an easy way to rollback a deployment using the `kamal rollback` command.

### Command:

```bash
kamal rollback [VERSION]
```

To rollback you need to specify the version you want to rollback to.

To find the version you can use the following command to list the available containers.

```bash
kamal app containers -q
```

```bash
App Host: 37.27.188.95
CONTAINER ID   IMAGE                                                            COMMAND                  CREATED          STATUS                        PORTS     NAMES
df826bc62d45   deepakmahakale/meetup:9533761e01c8810dc5a678675c0907a91cf7935d   "/rails/bin/docker-e…"   26 seconds ago   Up 25 seconds                 80/tcp    meetup-web-9533761e01c8810dc5a678675c0907a91cf7935d
911a7e9baa16   deepakmahakale/meetup:f34e9efe57fb781c89b3ce3f85427bb230b34d9e   "/rails/bin/docker-e…"   10 days ago      Exited (255) 18 seconds ago             meetup-web-f34e9efe57fb781c89b3ce3f85427bb230b34d9e
```


**Note:**<br>
`9533761e01c8810dc5a678675c0907a91cf7935d` is the version that we just deployed (bad version)<br>
`f34e9efe57fb781c89b3ce3f85427bb230b34d9e` is the previous stable version that we want to rollback to.
{: .notice--info}

Once we have identified the version we want to rollback to we can run the following command:

```bash
kamal rollback f34e9efe57fb781c89b3ce3f85427bb230b34d9e
```

This will rollback the deployment to the specified version.
