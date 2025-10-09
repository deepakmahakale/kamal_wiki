---
layout: single
title: How to use local docker registry with kamal?
description: &description >
  Want to deploy your Rails app without setting up a remote Docker registry?
  Learn how to use Kamal's new local Docker registry feature to streamline your deployment process.
  Save 10-15 seconds per deployment and simplify your setup!
excerpt: *description
date: 2025-10-09
categories:
  - Registry
tags:
  - Docker
  - Registry
header:
  og_image: /assets/images/opengraph/2025-10-09-how-to-use-local-docker-registry-with-kamal.png
---

This article covers the new local Docker registry feature introduced in Kamal that allows you to deploy applications without setting up a remote Docker registry.

### What is the Local Docker Registry Feature?

Kamal now supports using a local Docker registry for pushing and pulling application images. When you configure your registry server to start with `"localhost"`, Kamal will:

1. Start a local Docker registry on the specified port
2. Push your app image to this local registry
3. Use SSH port forwarding to make the registry accessible from your app servers

This can reduce deployment time by 10-15 seconds by eliminating the need to push/pull from remote registries.

### Configuration

Update your `config/deploy.yml` to use a local registry:

```yml
<% require "dotenv"; Dotenv.load(".env") %>

service: my-app
image: my-app
servers:
  web:
    - 192.168.1.100

registry:
  server: "localhost:5555"

env:
  secret:
    - RAILS_MASTER_KEY
```

**Note:** You don't need to specify username/password when using a local registry.
{: .notice--info}

### How It Works

When Kamal detects a localhost registry configuration:

1. **Local Registry Startup**: Kamal starts a Docker registry container on your deployment machine
2. **Image Push**: Your application image is pushed to this local registry
3. **SSH Port Forwarding**: Kamal creates an SSH tunnel from your app servers to the local registry
4. **Image Pull**: App servers pull the image through the SSH tunnel

### Benefits

- **Faster Deployments**: Eliminates remote registry push/pull time
- **Simplified Setup**: No need to configure Docker Hub, GHCR, or other remote registries
- **Development Friendly**: Perfect for staging environments or local development
- **Reduced Dependencies**: Less reliance on external services

### When to Use Local Registry

**Good for:**
- Development and staging environments
- Quick prototyping and testing
- Situations where you want minimal external dependencies
- Teams that prefer to keep images local during development

**Consider Remote Registry for:**
- Production deployments
- Multi-developer teams needing shared image access
- CI/CD pipelines
- Long-term image storage and versioning

### Troubleshooting

If you encounter issues:

```bash
# Check if the local registry is running
docker ps | grep registry

# Verify SSH port forwarding
kamal app details
```

This feature makes Kamal even more developer-friendly by removing the friction of setting up remote registries for development workflows.

{% include promo-compact.html %}
