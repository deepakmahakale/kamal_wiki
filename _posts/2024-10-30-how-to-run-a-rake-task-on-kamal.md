---
layout: single
title: How to run a rake task on Kamal?
description: &description >
  This article explains how to run a rake task on Kamal.
excerpt: *description
date: 2024-10-30
tags:
  - Rails
  - Rake Task
header:
  og_image: /assets/images/opengraph/2024-10-30-how-to-run-a-rake-task-on-kamal.png
redirect_from:
 - /kamal/how-to-run-a-rake-task-on-kamal/
---

To run a rake task on Kamal, you can use the following command:

**Syntax:**

```bash
kamal app exec -i 'bundle exec rake <task_name>'
```

**Example:**

```bash
kamal app exec -i 'bundle exec rake users:import'
```
