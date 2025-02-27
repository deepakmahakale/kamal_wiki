---
layout: single
title: How to push the env file with Kamal?
description: &description >
  Learn how to push your environment variables to hosts using Kamal's env push command.
  This post covers configuration, best practices, and how to secure your secrets.
  Simplify your deployment - Read more!
excerpt: *description
date: 2024-08-05
categories:
  - Setup
tags:
  - Env
  - Secrets
header:
  og_image: /assets/images/opengraph/2024-08-05-how-to-push-env-file-with-kamal.png
---

**Note:** This post is intended for kamal v1. If you're using Kamal v2 you don't need to push the env anymore.
{: .notice--info}

## Push the env file to your hosts

```bash
kamal env push
```

The above command uses the configuration in `config/deploy.yml` and env variables from `.env` file to push the env files to the hosts.
