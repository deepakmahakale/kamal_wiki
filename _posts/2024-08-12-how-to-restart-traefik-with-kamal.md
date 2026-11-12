---
layout: single
title: "How to restart Traefik or kamal-proxy with Kamal"
description: &description >
  Restart or reboot the proxy with Kamal: kamal proxy restart and reboot in Kamal 2 (kamal-proxy), kamal traefik in Kamal 1, and when to use each.
excerpt: *description
date: 2024-08-12
last_modified_at: 2026-11-12
categories:
  - Traefik
tags:
  - Traefik
header:
  og_image: /assets/images/opengraph/2024-08-12-how-to-restart-traefik-with-kamal.png
---

**Note:** Kamal 2 replaced Traefik with its own proxy, **kamal-proxy**. Use the `kamal proxy` commands on Kamal 2 and `kamal traefik` on Kamal 1.
{: .notice--info}

## Kamal 2: restart kamal-proxy

```bash
kamal proxy restart
```

Restarts the existing proxy container on your servers.

To stop and start it separately:

```bash
kamal proxy stop
kamal proxy start
```

### Reboot kamal-proxy

```bash
kamal proxy reboot
```

`reboot` stops and removes the proxy container, then starts a new one. Use it to upgrade kamal-proxy or apply new proxy settings. It causes a short outage on each server and asks for confirmation.

With several servers, reboot them one at a time:

```bash
kamal proxy reboot --rolling
```

### Check the proxy

```bash
kamal proxy details   # container status
kamal proxy logs      # recent requests and errors
```

## Kamal 1: restart Traefik

You can restart Traefik using the following command:

```bash
kamal traefik restart
```

You can also use the separate commands to stop and start Traefik:

```bash
kamal traefik stop
kamal traefik start
```

### Rebooting Traefik

It is worth noting that sometimes you may need to reboot Traefik to apply the changes.
You can do this by running the following command:

```bash
kamal traefik reboot
```

## Restart vs reboot

| | `restart` | `reboot` |
|---|---|---|
| Container | Same container | New container |
| Picks up new image or boot config | No | Yes |
| Downtime | Brief | Brief, per server (use `--rolling`) |

To restart your **app** rather than the proxy, use `kamal app boot`. See the [Kamal commands cheat sheet](/kamal-commands-cheat-sheet/).

## References

- [kamal proxy commands](https://kamal-deploy.org/docs/commands/proxy/){:target="_blank"}
- [Upgrading to Kamal 2](https://kamal-deploy.org/docs/upgrading/overview/){:target="_blank"}
