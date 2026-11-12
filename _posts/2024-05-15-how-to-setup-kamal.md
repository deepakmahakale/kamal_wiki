---
layout: single
title: "How to install and set up Kamal for a Rails app"
description: &description >
  Install Kamal, run kamal init, set KAMAL_REGISTRY_PASSWORD and deploy.yml, then run kamal setup for the first deploy. Updated for Kamal 2.
excerpt: *description
date: 2024-05-15
last_modified_at: 2026-11-12
categories:
  - Setup
tags:
  - Installation
header:
  og_image: /assets/images/opengraph/2024-05-15-how-to-setup-kamal.png
---

**Note:** This post has been updated for Kamal 2.
{: .notice--info}

## Installation

New Rails 8 apps already include Kamal: there's `gem "kamal"` in the `Gemfile`, a `bin/kamal` binstub, and `config/deploy.yml`. Skip to [Configuration](#configuration).

For other apps, add it to the Gemfile:

```bash
bundle add kamal --group development
```

Or install it globally:

```bash
gem install kamal
```

You'll also need Docker on your machine to build the image. On the servers, `kamal setup` installs Docker for you if it's missing.

## Initialization

```bash
kamal init
```

The above command will create 2 files

1. `.kamal/secrets`
2. `config/deploy.yml`

It also adds sample hooks in `.kamal/hooks`.

## Configuration

### .env

Add registry password to `.env` file. This can be a token from Docker Hub.

```bash
KAMAL_REGISTRY_PASSWORD=dckr_pat_*******************o
```

`KAMAL_REGISTRY_PASSWORD` is the password Kamal uses to push your image to the registry and pull it on the servers. Use an access token, not your account password:

| Registry | Token | `registry` settings |
|---|---|---|
| Docker Hub | Account settings → Personal access tokens (read & write) | `username: your-docker-id` |
| GitHub (GHCR) | Personal access token with `write:packages` | `server: ghcr.io`, `username: your-github-username` |

No registry at all? Kamal can run a local one. See [how to use a local Docker registry with Kamal](/registry/how-to-use-local-docker-registry-with-kamal/).

Keep `.env` out of git.

### .kamal/secrets

Add the following secrets to `.kamal/secrets` file.

```bash
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
RAILS_MASTER_KEY=$(cat config/master.key)
```

More in [managing secrets with Kamal](/managing-secrets-with-kamal/).

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

- `servers.web`: your server's IP. Kamal connects over SSH as `root` by default. Set `ssh: user:` for another user.
- `proxy.host`: point this domain's DNS at the server first, so `ssl: true` can get a Let's Encrypt certificate.
- `builder.arch`: the server's CPU (`amd64` or `arm64`).

## First deploy: `kamal setup`

```bash
kamal setup
```

Run this the first time. It installs Docker on the servers if needed, logs in to the registry, boots any accessories (databases, Redis), starts kamal-proxy and deploys the app.

### Deploy the application

```bash
kamal deploy
```

Use `kamal deploy` for every deploy after that.

## Next steps

- [Open a Rails console on Kamal](/how-to-run-rails-console-on-kamal/)
- [View Rails app logs](/how-to-view-rails-app-logs-on-kamal/)
- [Roll back a deployment](/how-to-rollback-a-deployment-with-kamal/)
- [Kamal commands cheat sheet](/kamal-commands-cheat-sheet/)
