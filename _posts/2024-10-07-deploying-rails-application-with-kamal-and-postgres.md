---
layout: single
title: Deploying Rails application with Kamal and Postgres - Complete Guide.
description: &description >
  Switching to PostgreSQL for your Rails app?
  Learn how to configure Kamal for seamless deployment with Postgres.
  This guide includes database setup, environment variables, and troubleshooting tips.
  Deploy like a pro - read now!
excerpt: *description
date: 2024-08-12
categories:
  - Accessory
tags:
  - Postgres
header:
  og_image: /assets/images/opengraph/2024-10-07-deploying-rails-application-with-kamal-and-postgres.png
redirect_from:
  - /kamal/deploying-rails-application-with-kamal-and-postgres/
---

This article assumes the file exists with sqlite config:

For complete config visit [Kamal Config Generator](https://dailydevtools.com/kamal_config)

**Note:** This post will guide you through deploying a Rails application with **PostgreSQL** using Kamal. If you're looking to deploy a Rails application with **MySQL**, check out our guide on [Deploying Rails application with Kamal and MySQL]({% post_url 2024-10-07-deploying-rails-application-with-kamal-and-mysql %}).
{: .notice--info}

### Pro Tip:

Run the following command to switch the database from `sqlite` to `PostgreSQL`:

```bash
rails db:system:change --to=postgresql
```

Verify the changes and then you can jump to step 4.

### 1. `Gemfile`

First, update the `Gemfile` to use `pg` as the database for Active Record.

```diff
-# Use sqlite3 as the database for Active Record
-gem "sqlite3", ">= 2.1"
+# Use pg as the database for Active Record
+gem "pg", "~> 1.1"
```

## 2. `Dockerfile`

Next, update the `Dockerfile` to install the `postgresql-client` and `libpq-dev` packages.

```diff
 # Install base packages
 RUN apt-get update -qq && \
-    apt-get install --no-install-recommends -y curl libjemalloc2 libvips sqlite3 && \
+    apt-get install --no-install-recommends -y curl libjemalloc2 libvips postgresql-client && \
     rm -rf /var/lib/apt/lists /var/cache/apt/archives


 # Install packages needed to build gems
 RUN apt-get update -qq && \
-    apt-get install --no-install-recommends -y build-essential git pkg-config && \
+    apt-get install --no-install-recommends -y build-essential git libpq-dev pkg-config && \
     rm -rf /var/lib/apt/lists /var/cache/apt/archives
```

### 3. `config/database.yml`

Update the `config/database.yml` to use `postgres` as the adapter.

**Note:** Make sure to replace the `host` to `<%= ENV["DB_HOST"] %>` and
`password` in production block to `<%= ENV["POSTGRES_PASSWORD"] %>`
{: .notice--info}

```yml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: kamal_demo_development

test:
  <<: *default
  database: kamal_demo_test

production:
  primary: &primary_production
    <<: *default
    host: <%= ENV["DB_HOST"] %>
    database: kamal_demo_production
    username: kamal_demo
    password: <%= ENV["POSTGRES_PASSWORD"] %>
  cache:
    <<: *primary_production
    database: kamal_demo_production_cache
    migrations_paths: db/cache_migrate
  queue:
    <<: *primary_production
    database: kamal_demo_production_queue
    migrations_paths: db/queue_migrate
  cable:
    <<: *primary_production
    database: kamal_demo_production_cable
    migrations_paths: db/cable_migrate
```

### 4. `config/deploy.yml`

Update the `config/deploy.yml` to add environment variables the db accessory.

```yml
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
    - POSTGRES_PASSWORD
  clear:
    DB_HOST: kamal-demo-db
    POSTGRES_USER: kamal_demo
    POSTGRES_DB: kamal_demo_production
    SOLID_QUEUE_IN_PUMA: true

volumes:
  - 'kamal_demo_storage:/rails/storage'

asset_path: /rails/public/assets

builder:
  arch: amd64

accessories:
  db:
    image: postgres:16
    host: 12.34.56.78
    port: 5432
    env:
      clear:
        DB_HOST: kamal-demo-db
        POSTGRES_USER: kamal_demo
        POSTGRES_DB: kamal_demo_production
      secret:
        - POSTGRES_PASSWORD
    files:
      - db/production.sql:/docker-entrypoint-initdb.d/setup.sql
    directories:
      - data:/var/lib/postgresql/data
```

### 5. `.env`

Add the `POSTGRES_PASSWORD` to the `.env` file.

```sh
KAMAL_REGISTRY_PASSWORD=dckr_pat_*******************o
POSTGRES_PASSWORD=postgres_*******************_password
```

### 6. `.kamal/secrets`

Add the `POSTGRES_PASSWORD` to the `.kamal/secrets` file.

```sh
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
POSTGRES_PASSWORD=$POSTGRES_PASSWORD

RAILS_MASTER_KEY=$(cat config/master.key)
```

### 7. Deploy

Once done you should be able to deploy your Rails application with Kamal.

```bash
kamal deploy
```

If you get an error saying the service `app-name-db` already exists you can remove the service using the following command and then re-deploy.:

```bash
kamal accessory remove db
kamal deploy
```

We have a separate guide on [how to remove an accessory in Kamal]({% post_url 2024-10-07-how-to-remove-an-accessory-in-kamal %}).

{% include promo-compact.html %}
