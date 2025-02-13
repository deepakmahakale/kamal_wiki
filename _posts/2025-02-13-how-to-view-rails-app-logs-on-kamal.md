---
layout: single
title: How to view Rails app logs on Kamal?
description: &description >
  This article explains how to view Rails app logs on Kamal.
excerpt: *description
date: 2025-02-13
tags:
  - Rails
  - Logs
header:
  og_image: /assets/images/opengraph/2025-02-13-how-to-view-rails-app-logs-on-kamal.png
---

With the introduction of [aliases](https://kamal-deploy.org/docs/configuration/aliases/) in Kamal 2.0 it's easier to check the Rails app logs.

Kamal setup provides a default aliases for `logs`. But, if it's not available, you can add your own `aliases` to the `deploy.yml` file.

```yaml
aliases:
  logs: app logs -f
```

and then run the following command:

```bash
$ kamal logs

INFO Following logs on 37.xx.xxx.xx.
```
