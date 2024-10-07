---
layout: single
title: How to remove an accessory in Kamal?
description: &description >
  This article explains how to remove an accessory in Kamal.
excerpt: *description
date: 2024-10-07
categories:
  - Kamal
tags:
  - Accessory
header:
  og_image: /assets/images/opengraph/2024-10-07-how-to-remove-an-accessory-in-kamal.png
---

```bash
ERROR (SSHKit::Command::Failed): Exception while executing on host 192.168.0.1: docker exit status: 125
docker stdout: Nothing written
docker stderr: docker: Error response from daemon: Conflict. The container name "/kamal-demo-blog-db" is already in use by container "b59.....e25". You have to remove (or rename) that container to be able to reuse that name.
```

## Remove the accessory

You can remove the accessory using the following command:

```bash
kamal accessory remove db
```

## Redeploy

After removing the accessory, you can redeploy the application using the following command:

```bash
kamal deploy
```
