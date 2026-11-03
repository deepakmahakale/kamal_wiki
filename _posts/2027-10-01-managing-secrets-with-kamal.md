---
layout: single
title: 'Managing Secrets with Kamal - A Complete Guide'
description: &description >
  Learn how to manage secrets and environment variables with Kamal.
  This guide covers the basics, configuration, and best practices.
excerpt: *description
date: 2026-01-01
tags:
  - Secrets
header:
  og_image: /assets/images/opengraph/2026-01-01-managing-secrets-with-kamal.png
---

Kamal keeps secrets out of your `deploy.yml` and out of your Docker image by resolving them at deploy time from `.kamal/secrets`. This guide covers how that works and how to keep your env vars secure.

### 1. `.kamal/secrets`

This file tells Kamal where to find each secret. It's evaluated on your machine, not on the server, so it can shell out, read files, or pull from a secrets manager.

```bash
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
RAILS_MASTER_KEY=$(cat config/master.key)
```

**Note:** `.kamal/secrets` should never be committed with real values hardcoded in it. It should only contain references to where the real value comes from.
{: .notice--info}

### 2. `.env`

Local values that `.kamal/secrets` reads from using `$VAR_NAME` go in `.env`.

```bash
KAMAL_REGISTRY_PASSWORD=dckr_pat_*******************o
```

Add `.env` to your `.gitignore` so it never ends up in version control.

```bash
# .gitignore
.env
```

### 3. Reference secrets in `deploy.yml`

```yaml
env:
  secret:
    - RAILS_MASTER_KEY
  clear:
    SOLID_QUEUE_IN_PUMA: true
```

Anything under `secret` is pulled from `.kamal/secrets` and injected into the container at runtime, it's never baked into the image and never shown in `kamal app logs`. Anything under `clear` is a plain, non-sensitive env var.

### 4. Accessory secrets

Accessories like a database or Redis container can have their own secrets too:

```yaml
accessories:
  db:
    image: mysql:8.0
    host: 12.34.56.78
    env:
      secret:
        - MYSQL_ROOT_PASSWORD
```

```bash
# .kamal/secrets
MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD
```

See [Understanding Kamal Accessories]({% post_url 2026-01-01-what-is-kamal-accessory %}) for more on accessory configuration.

### 5. Pushing env changes (Kamal v1)

**Note:** This step is only needed on Kamal v1. If you're using Kamal v2, env vars are pushed automatically as part of `kamal deploy`.
{: .notice--info}

```bash
kamal env push
```

This reads `config/deploy.yml` and your `.env` file, then pushes the resolved env vars to your hosts. We cover this command in more detail in [How to push the env file with Kamal?]({% post_url 2024-08-05-how-to-push-env-file-with-kamal %})

### Troubleshooting

**Secret not found / blank value on the server**
Check that the variable referenced in `.kamal/secrets` actually resolves locally, run `kamal secrets print` (or echo the var) on your machine before deploying.

**Changed a secret but the app still uses the old value**
Secrets are injected at deploy time. Run `kamal deploy` again (or `kamal env push` on v1) after changing a value, restarting the container alone won't pick up a new secret.

**Accidentally committed a secret**
Rotate the credential immediately on the provider's side, then remove it from git history, changing it in `.env` alone is not enough once it's been pushed.

### Best practices

- Never hardcode real secret values in `deploy.yml` or `.kamal/secrets`, only reference where they come from.
- Keep `.env` local and out of git.
- Rotate `RAILS_MASTER_KEY` and registry tokens periodically, especially after someone leaves the project.
- Use `clear` only for values that are safe to see in plain text in logs and `docker inspect`.

{% include promo-compact.html %}
