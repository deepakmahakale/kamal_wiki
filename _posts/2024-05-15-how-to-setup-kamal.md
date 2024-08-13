---
layout: single
title: How to setup Kamal?
description: &description >
  This article explains how to setup kamal.
excerpt: *description
date: 2024-05-15
categories:
  - Setup
tags:
  - Installation
header:
  og_image: /assets/images/opengraph/2024-05-15-how-to-setup-kamal.png
---

## Installation

```bash
gem install kamal
```

## Initialization

```bash
kamal init
```

The above command will create 2 files

1. .env
2. config/deploy.yml

## Configuration

### .env

```bash
KAMAL_REGISTRY_PASSWORD=""
RAILS_MASTER_KEY=""
```

### config/deploy.yml

```yaml
service: <SERVICE_NAME>
image: <REGISTRY_USERNAME>/<IMAGE_NAME>
servers:
  - 192.168.0.1
  - 192.168.0.2
registry:
  username: <REGISTRY_USERNAME>
  password:
    - KAMAL_REGISTRY_PASSWORD
env:
  secret:
    - RAILS_MASTER_KEY
```
