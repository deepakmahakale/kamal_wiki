---
layout: single
title: How to push the env file with Kamal?
description: &description >
  This article explains how to manage and push the env file to your hosts with kamal.
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

## Push the env file to your hosts

```bash
kamal env push
```

The above command uses the configuration in `config/deploy.yml` and env variables from `.env` file to push the env files to the hosts.
