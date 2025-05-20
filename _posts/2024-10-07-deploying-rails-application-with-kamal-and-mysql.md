---
layout: single
title: Deploying Rails application with Kamal and MySQL - Complete Guide.
description: &description >
  Ready to deploy your Rails app using Kamal and MySQL?
  This detailed guide covers everything from Gemfile updates to database configuration and deployment commands.
  Get started with confidence - follow along!
excerpt: *description
date: 2024-08-12
categories:
  - Accessory
tags:
  - MySQL
header:
  og_image: /assets/images/opengraph/2024-10-07-deploying-rails-application-with-kamal-and-mysql.png?v=2
redirect_from:
  - /kamal/deploying-rails-application-with-kamal-and-mysql/
---

This article assumes the file exists with `sqlite` config:

For complete config visit [Kamal Config Generator](https://dailydevtools.com/kamal_config)

**Note:** This post will guide you through deploying a Rails application with **MySQL** using Kamal. If you're looking to deploy a Rails application with **PostgreSQL**, check out our guide on [Deploying Rails application with Kamal and Postgres]({% post_url 2024-10-07-deploying-rails-application-with-kamal-and-postgres %}).
{: .notice--info}

### Pro Tip:

Run the following command to switch the database from `sqlite` to `mysql2`:

```bash
rails db:system:change --to=mysql
```

Verify the changes and then you can jump to step 4.

### 1. `Gemfile`

First, update the `Gemfile` to use `mysql2` as the database for Active Record.

```diff
-# Use sqlite3 as the database for Active Record
-gem "sqlite3", ">= 2.1"
+# Use mysql2 as the database for Active Record
+gem "mysql2", "~> 0.5"
```

## 2. `Dockerfile`

Next, update the `Dockerfile` to install the `default-mysql-client` and `default-libmysqlclient-dev` packages.

```diff
 # Install base packages
 RUN apt-get update -qq && \
-    apt-get install --no-install-recommends -y curl libjemalloc2 libvips sqlite3 && \
+    apt-get install --no-install-recommends -y curl default-mysql-client libjemalloc2 libvips && \
     rm -rf /var/lib/apt/lists /var/cache/apt/archives


 # Install packages needed to build gems
 RUN apt-get update -qq && \
-    apt-get install --no-install-recommends -y build-essential git pkg-config && \
+    apt-get install --no-install-recommends -y build-essential default-libmysqlclient-dev git pkg-config && \
     rm -rf /var/lib/apt/lists /var/cache/apt/archives
```

### 3. `config/database.yml`

Update the `config/database.yml` to use `mysql2` as the adapter.

**Note:** Make sure to replace the `password` in production block to `<%= ENV["MYSQL_ROOT_PASSWORD"] %>`
{: .notice--info}

```yml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password:
  host: <%= ENV.fetch("DB_HOST") { "127.0.0.1" } %>

development:
  <<: *default
  database: kamal_demo_development

test:
  <<: *default
  database: kamal_demo_test

production:
  primary: &primary_production
    <<: *default
    database: kamal_demo_production
    password: <%= ENV["MYSQL_ROOT_PASSWORD"] %>
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
    - MYSQL_ROOT_PASSWORD
  clear:
    DB_HOST: kamal-demo-db
    SOLID_QUEUE_IN_PUMA: true

volumes:
  - 'kamal_demo_storage:/rails/storage'

asset_path: /rails/public/assets

builder:
  arch: amd64

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

### 5. `.env`

Add the `MYSQL_ROOT_PASSWORD` to the `.env` file.

```sh
KAMAL_REGISTRY_PASSWORD=dckr_pat_*******************o
MYSQL_ROOT_PASSWORD=mysql_*******************_password
```

### 6. `.kamal/secrets`

Add the `MYSQL_ROOT_PASSWORD` to the `.kamal/secrets` file.

```sh
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD

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
