---
layout: single
title: "Running Solid Queue on a separate server with Kamal 2"
description: &description >
  Move Solid Queue out of Puma onto its own server with a Kamal job role: deploy.yml, the shared database it needs, scaling workers, and checking it runs.
excerpt: *description
date: 2026-11-12
tags:
  - Solid Queue
  - Roles
header:
  og_image: /assets/images/opengraph/2026-11-12-running-solid-queue-on-a-separate-server-with-kamal.png
---

A new Rails 8 app runs Solid Queue **inside Puma** on the web server, through `SOLID_QUEUE_IN_PUMA: true` in `config/deploy.yml`. That's fine on one server. Once jobs start competing with web requests for CPU and memory, move them to a dedicated **job** role.

## 1. Use a database both servers can reach

This is the step that usually goes wrong.

**SQLite won't work across servers.** Rails 8 apps on SQLite keep the queue database in a file (`storage/production_queue.sqlite3`) on each server. A job server gets its own empty file, so jobs enqueued by the web server are never seen.
{: .notice--warning}

Use PostgreSQL or MySQL that both servers can connect to: a managed database, or an accessory on one of your servers. See [deploying with Kamal and Postgres](/accessory/deploying-rails-application-with-kamal-and-postgres/) or [MySQL](/accessory/deploying-rails-application-with-kamal-and-mysql/).

Then make sure **every** production database points at it, including `queue`. In the Rails 8 PostgreSQL template, `queue` inherits from `primary`, so setting the host once in `default` covers all of them:

```yaml
# config/database.yml
default: &default
  adapter: postgresql
  host: <%= ENV["DB_HOST"] %>
  # ...

production:
  primary: &primary_production
    <<: *default
    database: myapp_production
  queue:
    <<: *primary_production
    database: myapp_production_queue
    migrations_paths: db/queue_migrate
```

## 2. Add a job role and stop running jobs in Puma

```yaml
# config/deploy.yml
servers:
  web:
    - 192.168.0.1
  job:
    hosts:
      - 192.168.0.2
    cmd: bin/jobs

env:
  secret:
    - RAILS_MASTER_KEY
    - POSTGRES_PASSWORD
  clear:
    DB_HOST: 192.168.0.3
    # SOLID_QUEUE_IN_PUMA: true   <- remove this line
```

- **`cmd: bin/jobs`** starts the Solid Queue supervisor, which runs the dispatcher and the workers.
- **Remove `SOLID_QUEUE_IN_PUMA`**, or the web server keeps processing jobs too. Puma only loads the Solid Queue plugin when it's set to something other than empty, `false` or `0`.
- The job role gets the same image and `env` as `web`. Only `web` sits behind kamal-proxy, so the job server needs no open ports.

The job server can also be the **same machine** as the web server. A separate container still gives jobs their own process, and you can set `options: memory:` / `cpus:` on the role to keep them in check.

## 3. Deploy

```bash
kamal deploy
```

The web container runs `bin/rails db:prepare` when it boots (the Rails Docker entrypoint does this for the server command only), which creates and migrates the queue database. The job container doesn't run it.

On the very first deploy, the job container can start before the queue database exists and fail. Run `kamal deploy` again once the web role is up.

## 4. Check that it's working

```bash
kamal app logs -r job        # supervisor and worker startup, then job runs
kamal app logs -r job -f     # follow
```

Ask Solid Queue which processes are registered, from any host:

```bash
kamal app exec 'bin/rails runner "pp SolidQueue::Process.pluck(:kind, :hostname)"'
```

You should see `Supervisor`, `Dispatcher` and `Worker` entries (plus `Scheduler` if you use recurring tasks) with the job server's hostname. If you only see the web server's hostname, Puma is still running jobs: check that `SOLID_QUEUE_IN_PUMA` is gone.

## 5. Scale the workers

`config/queue.yml` controls workers. The Rails 8 template reads the process count from `JOB_CONCURRENCY`:

```yaml
production:
  workers:
    - queues: "*"
      threads: 3
      processes: <%= ENV.fetch("JOB_CONCURRENCY", 1) %>
```

Set it only for the job role:

```yaml
servers:
  job:
    hosts:
      - 192.168.0.2
    cmd: bin/jobs
    env:
      clear:
        JOB_CONCURRENCY: 3
```

Each worker thread needs a database connection, so keep the database `pool` (`RAILS_MAX_THREADS` in the Rails template) at least as large as `threads`, plus a little headroom.

More servers? Add hosts to the `job` role. Every host runs its own supervisor, and Solid Queue's locking stops two workers from taking the same job.

## Not processing jobs?

1. `SolidQueue::Process` shows no job-server processes: the container isn't running. Check `kamal app details -r job`.
2. Processes are there but jobs stay queued: the job server is connected to a different database. Run `kamal app exec -r job 'bin/rails runner "pp SolidQueue::Job.connection_db_config.configuration_hash.slice(:host, :database)"'` and compare it with the web role.
3. Jobs only run on a schedule: check `config/recurring.yml`.

## References

- [Kamal docs: Roles](https://kamal-deploy.org/docs/configuration/roles/){:target="_blank"}
- [Solid Queue on GitHub](https://github.com/rails/solid_queue){:target="_blank"}
- [Stack Overflow: Setting up Solid Queue jobs on a dedicated server with Rails 8 + Kamal 2](https://stackoverflow.com/questions/79371773){:target="_blank"}
