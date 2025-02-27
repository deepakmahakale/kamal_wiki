---
layout: single
title: Accessing the rails console on Kamal?
description: &description >
  Learn how to open the Rails console in Kamal for both version 1.x and 2.0.
  This guide covers commands and aliases to make debugging and management easy. Start exploring your app.
excerpt: *description
date: 2024-10-28
last_modified_at: 2025-02-27T16:00:00+05:30
tags:
  - Rails
  - Console
header:
  og_image: /assets/images/opengraph/2024-10-28-how-to-run-rails-console-on-kamal.png
redirect_from:
 - /kamal/how-to-run-rails-console-on-kamal/
---

The Rails console is a powerful tool for debugging, testing, and managing your Rails application. You can use it to interact with your application's models, run queries, etc.

A common use case would be to check some data related to users, products or specific scenarios like users with specific attributes, etc.

### With Kamal 2.0

With the introduction of [aliases](https://kamal-deploy.org/docs/configuration/aliases/) in Kamal 2.0 it's easier to run the Rails console.

You can define `aliases` in the `deploy.yml` file:

```yaml
aliases:
  console: app exec --reuse -i "bin/rails console"
```

and then run the Rails console using the following command:

```bash
kamal console
```

### With Kamal 1.x

With Kamal 1.x, you can run the Rails console using the following command:

```bash
kamal app exec -i --reuse "bin/rails console"
```
