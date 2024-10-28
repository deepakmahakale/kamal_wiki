---
layout: single
title: How to run rails console on Kamal?
description: &description >
  This article explains how to run the Rails console on Kamal.
excerpt: *description
date: 2024-10-28
categories:
  - Kamal
tags:
  - Rails
  - Console
header:
  og_image: /assets/images/opengraph/2024-10-28-how-to-run-rails-console-on-kamal.png
---

### With Kamal 1.x

With Kamal 1.x, you can run the Rails console using the following command:

```bash
kamal app exec -i --reuse "bin/rails console"
```

### With Kamal 2.0

With the introduction of [aliases](https://kamal-deploy.org/docs/configuration/aliases/) in Kamal 2.0 it's even easier to run the Rails console.

You can define `aliases` in the `deploy.yml` file:

```yaml
aliases:
  console: app exec --reuse -i "bin/rails console"
```

and then run the Rails console using the following command:

```bash
kamal console
```
