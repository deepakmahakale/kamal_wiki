---
layout: single
title: 'Kamal Deployment: From Zero to Hero'
description: &description >
  Learn how to deploy your application with Kamal from scratch.
  This guide covers the basics, configuration, and best practices.
excerpt: *description
date: 2026-01-01
categories:
  - Setup
tags:
  - Deployment
header:
  og_image: /assets/images/opengraph/2026-01-01-kamal-deployment-zero-to-hero.png
---

Kamal lets you deploy web apps anywhere, from a single VPS to a fleet of servers, without needing Kubernetes or a PaaS. This guide walks you through the entire journey, from installing Kamal to rolling back a bad deploy.

For complete config visit [Kamal Config Generator](https://dailydevtools.com/kamal_config)

### 1. Install Kamal

```bash
gem install kamal
```

### 2. Initialize your project

```bash
kamal init
```

This creates two files:

1. `.kamal/secrets`
2. `config/deploy.yml`

We cover installation and initialization in more detail in [How to setup Kamal?]({% post_url 2024-05-15-how-to-setup-kamal %})

### 3. Configure `config/deploy.yml`

```yaml
<% require "dotenv"; Dotenv.load(".env") %>

service: kamal-demo
image: yourregistryusername/kamal-demo
servers:
  web:
    - 12.34.56.78

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

**Note:** If your database needs a container of its own (Postgres, MySQL, SQLite), check out our dedicated guides for [Postgres]({% post_url 2024-10-07-deploying-rails-application-with-kamal-and-postgres %}), [MySQL]({% post_url 2024-10-07-deploying-rails-application-with-kamal-and-mysql %}), and [SQLite]({% post_url 2026-01-01-deploying-rails-application-with-kamal-and-sqlite %}).
{: .notice--info}

### 4. Add your secrets

```bash
# .kamal/secrets
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
RAILS_MASTER_KEY=$(cat config/master.key)
```

```bash
# .env
KAMAL_REGISTRY_PASSWORD=dckr_pat_*******************o
```

Read more on handling secrets in [Managing Secrets with Kamal - A Complete Guide]({% post_url 2027-10-01-managing-secrets-with-kamal %})

### 5. Deploy

```bash
kamal deploy
```

This builds your image, pushes it to the registry, boots up Traefik on your servers, and starts your app containers with zero downtime.

### 6. Rollback if something goes wrong

Find the previous good version:

```bash
kamal app containers -q
```

Then rollback to it:

```bash
kamal rollback [VERSION]
```

We go deeper into this in [How to rollback a deployment with Kamal?]({% post_url 2025-02-17-how-to-rollback-a-deployment-with-kamal %})

### 7. Day-to-day commands

Once you're deployed, these commands cover most of what you'll need:

```bash
kamal app logs -f      # tail logs
kamal app exec --interactive --reuse "bin/rails console"   # rails console
kamal app details       # container status
```

You can also set up shortcuts for these with [aliases]({% post_url 2025-02-11-how-to-add-alias-with-kamal %}), and add extra services like a database or Redis with [accessories]({% post_url 2027-10-01-what-is-kamal-accessory %}).

### FAQs

**Do I need Kubernetes to use Kamal?**
No. Kamal works over SSH and plain Docker, so a single VPS is enough to get started.

**Can I deploy to multiple servers?**
Yes, just list additional hosts under `servers` in `deploy.yml` and Kamal will deploy to all of them.

**What happens during a deploy?**
Kamal builds a new image, pushes it to your registry, pulls it on each server, boots the new containers, waits for health checks to pass, then swaps traffic over through Traefik before stopping the old containers.

**How do I undo a bad deploy?**
Use `kamal rollback [VERSION]` with the previous container's version hash, see the rollback section above.

{% include promo-compact.html %}
