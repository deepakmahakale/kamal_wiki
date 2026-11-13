---
layout: single
title: 'Understanding Kamal Accessories: A Complete Guide'
description: &description >
  Learn how to use Kamal accessories to simplify your deployment process.
  This guide covers the basics, configuration, and best practices.
excerpt: *description
date: 2026-01-01
categories:
  - Accessory
tags:
  - Accessory
header:
  og_image: /assets/images/opengraph/2026-01-01-what-is-kamal-accessory.png
---

An accessory in Kamal is any supporting service your app needs that isn't your app itself, a database, a cache, a background job runner, that Kamal will run as its own Docker container alongside your app.

### What counts as an accessory?

Anything you'd normally run as a separate container:

- **Databases** - Postgres, MySQL
- **Caching** - Redis, Memcached
- **Search** - Elasticsearch, Meilisearch
- **Anything else** your app depends on that isn't the app container itself

### Basic configuration

Accessories are defined under `accessories` in `config/deploy.yml`:

```yaml
accessories:
  db:
    image: mysql:8.0
    host: 12.34.56.78
    port: "127.0.0.1:3306:3306"
    env:
      clear:
        MYSQL_ROOT_HOST: '%'
      secret:
        - MYSQL_ROOT_PASSWORD
    files:
      - db/production.sql:/docker-entrypoint-initdb.d/setup.sql
    directories:
      - data:/var/lib/mysql
```

We cover a full database setup in [Deploying Rails application with Kamal and MySQL]({% post_url 2024-10-07-deploying-rails-application-with-kamal-and-mysql %}) and [...and Postgres]({% post_url 2024-10-07-deploying-rails-application-with-kamal-and-postgres %}).

### Example: Redis as a caching accessory

```yaml
accessories:
  redis:
    image: redis:7
    host: 12.34.56.78
    port: "6379:6379"
    directories:
      - data:/data
```

Point your app at it with a `clear` env var:

```yaml
env:
  clear:
    REDIS_URL: redis://12.34.56.78:6379/0
```

### Key options

- **image** - the Docker image to run
- **host** - which server the accessory runs on
- **port** - host:container port mapping
- **env** - `clear` for plain values, `secret` for values pulled from `.kamal/secrets`, see [Managing Secrets with Kamal]({% post_url 2027-10-01-managing-secrets-with-kamal %})
- **directories** - persists data outside the container so it survives restarts
- **files** - copies local files into the container on boot, useful for seed SQL or config files

### Managing accessories

Boot an accessory for the first time:

```bash
kamal accessory boot db
```

Reboot one after a config change:

```bash
kamal accessory reboot db
```

Tail its logs:

```bash
kamal accessory logs db
```

Remove it entirely:

```bash
kamal accessory remove db
```

**Note:** Kamal names accessories using your app's service name, so `db` is enough, you don't need to pass the full container name.
{: .notice--info}

### Troubleshooting

**Container name already in use**
This happens when an accessory with the same name already exists on the host, usually from a previous deploy with a different config. Remove it and redeploy, see [How to remove an accessory in Kamal]({% post_url 2024-10-07-how-to-remove-an-accessory-in-kamal %}).

```bash
kamal accessory remove db
kamal deploy
```

**Data disappears after a reboot**
Make sure you've set `directories` so the accessory's data lives on a volume outside the container, otherwise it's wiped every time the container restarts.

**App can't connect to the accessory**
Double check the `host` the accessory is running on matches what your app's env var points to, and that the port is actually exposed.
