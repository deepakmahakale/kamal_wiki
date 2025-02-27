---
layout: single
title: How to remove an accessory in Kamal - A Troubleshooting Guide
description: &description >
  Encountering conflicts with Kamal accessories?
  Learn how to safely remove an accessory and redeploy your app.
  This post includes commands and tips to resolve common issues.
  Fix your setup - read more!
excerpt: *description
date: 2024-10-07
last_modified_at: 2025-02-27T16:00:00+05:30
categories:
  - Accessory
tags:
  - Accessory
header:
  og_image: /assets/images/opengraph/2024-10-07-how-to-remove-an-accessory-in-kamal.png
redirect_from:
 - /kamal/how-to-remove-an-accessory-in-kamal/
---

There can be scenarios where you need to remove an accessory from your Kamal deployment.
Let it be a conflict or a misconfiguration, you might need to remove an accessory to redeploy your application.
This guide will help you understand how to remove an accessory and redeploy your application.

Now look at the error message below:

```bash
ERROR (SSHKit::Command::Failed): Exception while executing on host 192.168.0.1: docker exit status: 125
docker stdout: Nothing written
docker stderr: docker: Error response from daemon: Conflict. The container name "/kamal-demo-blog-db" is already in use by container "b59.....e25". You have to remove (or rename) that container to be able to reuse that name.
```

This error message indicates that the container name is already in use by another container and one way to resolve this is to remove the accessory and redeploy the application.

## Remove the accessory

You can remove the accessory using the following command:

```bash
kamal accessory remove db
```

Note: since kamal generates the accessory name based on the application name, you dont need to specify the full accessory name.
In this case, the accessory name is `kamal-demo-blog-db` but providing `db` is sufficient.

## Redeploy

After removing the accessory, you can redeploy the application using the following command:

```bash
kamal deploy
```
