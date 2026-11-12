---
layout: single
title: "How to push env variables with Kamal (env push in Kamal 1 and 2)"
description: &description >
  kamal env push only exists in Kamal 1. In Kamal 2, secrets live in .kamal/secrets and are pushed on deploy. Here's how both versions work.
excerpt: *description
date: 2024-08-05
last_modified_at: 2026-11-12
categories:
  - Setup
tags:
  - Env
  - Secrets
header:
  og_image: /assets/images/opengraph/2024-08-05-how-to-push-env-file-with-kamal.png
---

**Short answer:** `kamal env push` was removed in Kamal 2. Secrets now go in `.kamal/secrets` and Kamal sends them to your servers on every `kamal deploy`.
{: .notice--info}

## Kamal 2: secrets are pushed on deploy

### 1. Put the values in `.kamal/secrets`

`kamal init` creates this file. It uses dotenv syntax and can run commands:

```bash
# .kamal/secrets
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
RAILS_MASTER_KEY=$(cat config/master.key)
DATABASE_PASSWORD=$(op read op://Production/db/password)
```

Don't commit real values. Read them from your shell environment, a file that's not in git, or a password manager.

### 2. List them in `config/deploy.yml`

```yaml
env:
  clear:
    RAILS_LOG_LEVEL: info
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_PASSWORD
```

`clear` values are written into the config as-is. `secret` names are looked up in `.kamal/secrets`.

### 3. Deploy

```bash
kamal deploy
```

Kamal writes the env file on each server as part of the deploy. There's no separate push step.

**Changed a secret?** Run `kamal deploy` (or `kamal redeploy`) again so the new value reaches the containers.
{: .notice--info}

### Check what Kamal will send

```bash
kamal secrets print
```

Prints the resolved secrets locally. Use it to debug, not in CI logs.

### Different values per destination

For `kamal deploy -d staging`, Kamal reads `.kamal/secrets.staging`. Values shared by every destination can go in `.kamal/secrets-common`.

More in [managing secrets with Kamal](/managing-secrets-with-kamal/).

## Kamal 1: `kamal env push`

In Kamal 1, secrets were read from a `.env` file and pushed separately:

```bash
kamal env push
```

The above command uses the configuration in `config/deploy.yml` and env variables from `.env` file to push the env files to the hosts.

After changing `.env`, run `kamal env push` and then restart the app (`kamal app boot`) so containers pick up the new values.

Upgrading from Kamal 1? `kamal upgrade` moves your servers to Kamal 2. Then move the values from `.env` into `.kamal/secrets`.

{% include promo-compact.html %}
