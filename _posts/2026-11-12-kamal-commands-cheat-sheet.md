---
layout: single
title: "Kamal commands cheat sheet (Kamal 2)"
description: &description >
  Every Kamal 2 command you'll use day to day: setup, deploy, rollback, logs, console, exec, accessories, proxy and secrets, with examples.
excerpt: *description
date: 2026-11-12
tags:
  - Commands
  - Cheatsheet
header:
  og_image: /assets/images/opengraph/2026-11-12-kamal-commands-cheat-sheet.png
---

All commands run from your app's directory and read `config/deploy.yml`. On a Rails 8 app, use `bin/kamal`.

## Deploy

| Command | What it does |
|---|---|
| `kamal init` | Create `config/deploy.yml`, `.kamal/secrets` and sample hooks |
| `kamal setup` | First deploy: install Docker on servers, boot accessories, start the proxy, deploy |
| `kamal deploy` | Build, push and deploy the app |
| `kamal redeploy` | Deploy without bootstrapping servers, starting the proxy or pruning |
| `kamal rollback VERSION` | Go back to an earlier version ([how to rollback](/how-to-rollback-a-deployment-with-kamal/)) |
| `kamal app version` | Show the version running on the servers |
| `kamal app images` | List image versions on the servers |

Setup guide: [how to install and set up Kamal](/setup/how-to-setup-kamal/).

## Logs and debugging

```bash
kamal app logs              # recent app logs
kamal app logs -f           # follow
kamal app logs --grep ERROR # filter
kamal proxy logs            # kamal-proxy request logs
kamal details               # status of every container
kamal audit                 # who deployed what, when
```

More in [how to view Rails app logs on Kamal](/how-to-view-rails-app-logs-on-kamal/).

## Run commands in the app

```bash
kamal app exec -i 'bin/rails console'      # Rails console
kamal app exec 'bin/rails db:migrate'      # one-off command
kamal app exec -i --reuse bash             # shell in the running container
kamal server exec 'df -h'                  # command on the host, not the container
```

Guides: [Rails console](/how-to-run-rails-console-on-kamal/), [rake tasks](/how-to-run-a-rake-task-on-kamal/), [aliases](/how-to-add-alias-with-kamal/) to shorten these.

## App lifecycle

| Command | What it does |
|---|---|
| `kamal app boot` | Start the app, or reboot it if it's running |
| `kamal app stop` / `kamal app start` | Stop or start the existing app container |
| `kamal app maintenance` | Show a 503 maintenance page |
| `kamal app live` | Back to normal |
| `kamal prune` | Remove old images and containers |

## Accessories (databases, Redis, …)

```bash
kamal accessory boot db        # start (use all for every accessory)
kamal accessory reboot db      # recreate the container
kamal accessory logs db
kamal accessory exec db --reuse -i 'psql -U postgres'
kamal accessory remove db      # removes container, image and data directory
```

See [what is a Kamal accessory](/accessory/what-is-kamal-accessory/) and [how to remove an accessory](/accessory/how-to-remove-an-accessory-in-kamal/).

## Proxy

```bash
kamal proxy restart
kamal proxy reboot --rolling
kamal proxy details
```

More in [how to restart kamal-proxy](/traefik/how-to-restart-traefik-with-kamal/).

## Secrets and config

```bash
kamal secrets print     # resolved secrets (debugging)
kamal config            # full config, including secrets
kamal registry login
```

See [managing secrets with Kamal](/managing-secrets-with-kamal/).

## Deploy lock

```bash
kamal lock status
kamal lock release      # after a deploy was interrupted
```

## Useful flags

| Flag | Effect |
|---|---|
| `-d staging` | Use `config/deploy.staging.yml` |
| `-r web` | Only these roles |
| `-h 1.2.3.4` | Only these hosts |
| `-p` | Primary host only |
| `-v` | Verbose output |

## References

- [Kamal commands](https://kamal-deploy.org/docs/commands/view-all-commands/){:target="_blank"}

{% include promo-compact.html %}
