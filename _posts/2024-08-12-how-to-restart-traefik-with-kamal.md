---
layout: single
title: How to restart Traefik with Kamal?
description: &description >
  Need to restart Traefik for your Kamal deployment?
  Discover the commands for restarting and rebooting Traefik, plus when to use each.
  Keep your routing smooth - check out the guide!
excerpt: *description
date: 2024-08-12
categories:
  - Traefik
tags:
  - Traefik
header:
  og_image: /assets/images/opengraph/2024-08-12-how-to-restart-traefik-with-kamal.png
---

## Restarting Traefik

You can restart Traefik using the following command:

```bash
kamal traefik restart
```

You can also use the separate commands to stop and start Traefik:

```bash
kamal traefik stop
kamal traefik start
```

## Rebooting Traefik


It is worth noting that sometimes you may need to reboot Traefik to apply the changes.
You can do this by running the following command:

```bash
kamal traefik reboot
```
