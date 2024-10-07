---
layout: single
title: How to setup Kamal?
description: &description >
  This article explains how to setup kamal.
excerpt: *description
date: 2024-05-15
last_modified_at: 2024-10-07T20:00:00+05:30
categories:
  - Setup
tags:
  - Installation
header:
  og_image: /assets/images/opengraph/2024-05-15-how-to-setup-kamal.png
---

**Note:** This post has been updated on 7th Oct, 2024 to reflect the changes in Kamal 2.0.
{: .notice--info}

## Installation

```bash
gem install kamal
```

## Initialization

```bash
kamal init
```

The above command will create 2 files

1. `.kamal/secrets`
2. `config/deploy.yml`

## Configuration

### .env

Add registry password to `.env` file. This can be a token from Docker Hub.

```bash
KAMAL_REGISTRY_PASSWORD=dckr_pat_*******************o
```

### .kamal/secrets

Add the following secrets to `.kamal/secrets` file.

```bash
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
RAILS_MASTER_KEY=$(cat config/master.key)
```

### config/deploy.yml

```yaml
<% require "dotenv"; Dotenv.load(".env") %>

service: kamal-demo
image: yourregistryusername/kamal-demo
servers:
  web:
    - 192.168.0.1

proxy:
  ssl: true
  host: example.com

registry:
  username: yourregistryusername
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  secret:
    - RAILS_MASTER_KEY
  clear:
    SOLID_QUEUE_IN_PUMA: true

volumes:
  - 'kamal_demo_storage:/rails/storage'

asset_path: /rails/public/assets

builder:
  arch: amd64
```

### Deploy the application

```bash
kamal deploy
```

The above command will deploy the application to the server.
