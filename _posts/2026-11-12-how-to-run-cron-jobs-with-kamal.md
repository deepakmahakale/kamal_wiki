---
layout: single
title: "How to run cron jobs with Kamal"
description: &description >
  Run cron jobs for a Rails app deployed with Kamal: a cron role with supercronic, a crontab in your repo, env vars, logs, or Solid Queue recurring tasks.
excerpt: *description
date: 2026-11-12
tags:
  - Cron
  - Roles
header:
  og_image: /assets/images/opengraph/2026-11-12-how-to-run-cron-jobs-with-kamal.png
---

Kamal has no cron command of its own. You run cron in its own container, as a separate **role** that uses the same image as your app.

There are two good options for a Rails app:

1. **Solid Queue recurring tasks**, if your app already uses Solid Queue (the Rails 8 default). No extra container.
2. **A `cron` role running supercronic**, for anything else: shell scripts, rake tasks, or apps without Solid Queue.

## Option 1: Solid Queue recurring tasks

If jobs already run through Solid Queue, schedule them in `config/recurring.yml` and you're done:

```yaml
# config/recurring.yml
production:
  nightly_report:
    class: NightlyReportJob
    schedule: every day at 2am
  refresh_feeds:
    command: "Feed.refresh_all"
    schedule: every 15 minutes
```

They run wherever Solid Queue runs: inside Puma with `SOLID_QUEUE_IN_PUMA`, or on a [separate job server](/running-solid-queue-on-a-separate-server-with-kamal/). Need to remove one later? See [how to delete recurring tasks in Solid Queue](https://ror.tips/rails/how-to-delete-recurring-tasks-in-solid-queue/).

## Option 2: a cron role with supercronic

### Why supercronic and not cron?

Kamal's docs show a role running Debian's `cron`. With the Dockerfile Rails generates, that runs into two problems:

- **The container runs as a non-root `rails` user**, and the `cron` daemon needs root.
- **`cron` clears the environment**, so jobs don't see `RAILS_ENV`, `RAILS_MASTER_KEY` or your database settings.

[Supercronic](https://github.com/aptible/supercronic) is a cron built for containers. It runs as any user, keeps the environment, and logs to stdout, so `kamal app logs` shows the output.

### 1. Install supercronic in the Dockerfile

Add this to the final stage of your `Dockerfile`, **before** the `USER 1000:1000` line (it writes to `/usr/local/bin`). The Rails Dockerfile already installs `curl`.

```dockerfile
# Latest releases: https://github.com/aptible/supercronic/releases
ENV SUPERCRONIC_URL=https://github.com/aptible/supercronic/releases/download/v0.2.51/supercronic-linux-amd64 \
    SUPERCRONIC_SHA1SUM=1692066e350a88ff4d7d6474fe3c7a6c41ec5e89 \
    SUPERCRONIC=supercronic-linux-amd64

RUN curl -fsSLO "$SUPERCRONIC_URL" \
 && echo "${SUPERCRONIC_SHA1SUM}  ${SUPERCRONIC}" | sha1sum -c - \
 && chmod +x "$SUPERCRONIC" \
 && mv "$SUPERCRONIC" "/usr/local/bin/${SUPERCRONIC}" \
 && ln -s "/usr/local/bin/${SUPERCRONIC}" /usr/local/bin/supercronic
```

Building for ARM servers (`builder: arch: arm64`)? Use the `supercronic-linux-arm64` stanza from the release page instead. Each file has its own checksum.

### 2. Add a crontab to the repo

```
# config/crontab
# Times are in the container's timezone (UTC unless you set TZ)

# 02:00 every day
0 2 * * * bin/rails runner 'NightlyReport.generate'

# every 15 minutes
*/15 * * * * bin/rails runner 'Feed.refresh_all'

# a rake task
30 3 * * 0 bin/rails cleanup:old_uploads
```

Commands run from the image's working directory (`/rails`), so `bin/rails` works as-is.

### 3. Add a cron role in `config/deploy.yml`

```yaml
servers:
  web:
    - 192.168.0.1
  cron:
    hosts:
      - 192.168.0.1
    cmd: supercronic /rails/config/crontab
```

The `cron` role uses the same image and the same `env` as `web`, so jobs get your secrets and database settings. Only the primary role (`web`) sits behind kamal-proxy, so the cron container gets no web traffic.

### 4. Deploy

```bash
kamal deploy
```

Kamal boots a `cron` container next to the web one.

**Use one cron host only.** Every host in the `cron` role runs the full crontab, so two hosts means every job runs twice.
{: .notice--warning}

### Check that it works

```bash
kamal app logs -r cron       # supercronic logs each run and its output
kamal app logs -r cron -f    # follow
kamal app details -r cron    # is the container up?
```

Test a single command in the same environment before it's scheduled:

```bash
kamal app exec -r cron 'bin/rails runner "puts NightlyReport.count"'
```

### Changing the schedule

Edit `config/crontab` and run `kamal deploy`. The crontab is part of the image, so it changes with every deploy, just like your code.

## Which one should I use?

| | Solid Queue recurring | Cron role (supercronic) |
|---|---|---|
| Extra container | No | Yes |
| Runs | Active Job classes or Ruby code | Any command |
| Retries and failed-job tracking | Yes, via Solid Queue | No |
| Best for | Rails apps already on Solid Queue | Scripts, rake tasks, non-Solid-Queue apps |

## References

- [Kamal docs: Cron](https://kamal-deploy.org/docs/configuration/cron/){:target="_blank"}
- [Kamal docs: Roles](https://kamal-deploy.org/docs/configuration/roles/){:target="_blank"}
- [Supercronic](https://github.com/aptible/supercronic){:target="_blank"}
- [Solid Queue: Recurring tasks](https://github.com/rails/solid_queue#recurring-tasks){:target="_blank"}
