---
layout: single
title: "Kamal SSL: turn on assume_ssl and force_ssl"
description: &description >
  New Rails 8.1 apps deployed with Kamal ship with assume_ssl and force_ssl commented out.
  Turn them on when you enable SSL in kamal-proxy.
excerpt: *description
date: 2026-11-13
tags:
  - SSL
  - Proxy
  - Rails
header:
  og_image: /assets/images/opengraph/2026-11-13-kamal-ssl-assume-ssl-and-force-ssl.png
---

This article covers the two SSL settings in `config/environments/production.rb` that you need to turn on once kamal-proxy serves your app over HTTPS.

Since Rails 8.1.1, new apps that use Kamal have them commented out:

```ruby
# config/environments/production.rb
# config.assume_ssl = true
# config.force_ssl = true
```

This lets the first `kamal deploy` work on a plain IP address without SSL.
Once you add a domain and turn on SSL, Rails expects you to turn them back on. From the generated `config/deploy.yml`:

```yml
# Using an SSL proxy like this requires turning on config.assume_ssl and config.force_ssl in production.rb!
```

### Turn on SSL in kamal-proxy

```yml
# config/deploy.yml
proxy:
  ssl: true
  host: app.example.com
```

Point the domain's DNS at your server first, so kamal-proxy can get a Let's Encrypt certificate.

### Turn on assume_ssl and force_ssl

```ruby
# config/environments/production.rb
config.assume_ssl = true
config.force_ssl = true
```

Then deploy:

```bash
kamal deploy
```

### What they do

1. **`assume_ssl`**: kamal-proxy handles HTTPS and talks to your app over plain HTTP. This tells Rails every request came in over HTTPS, so generated URLs, redirects and origin checks use `https://`.
2. **`force_ssl`**: redirects HTTP to HTTPS, sends the `Strict-Transport-Security` header and marks cookies as secure.

Turn on both together. With `assume_ssl` on, Rails doesn't redirect kamal-proxy's healthcheck on `/up` either, because it treats that request as HTTPS too.

### Turned them on too early?

If you turn them on while the app is still served over plain HTTP (no `ssl: true` in the proxy), form submissions fail with:

```
HTTP Origin header (http://203.0.113.10) didn't match request.base_url (https://203.0.113.10)
```

Turn on SSL in kamal-proxy, or comment the two lines out again until you do.

**Note:** Behind Cloudflare, set the SSL/TLS encryption mode to **Full**, as the generated `deploy.yml` suggests. With multiple web servers, terminate SSL at your load balancer instead of kamal-proxy, and keep both settings on.
{: .notice--info}

### Upgrading with app:update

On Rails 8.1.1 to 8.1.4.1, `bin/rails app:update` can offer to comment these lines out, even on apps that already have SSL working.
See [Rails 8.1 app:update can disable force_ssl](https://ror.tips/rails/rails-8-1-app-update-can-disable-force-ssl/).

### References

- [rails/rails#56010 - Disable SSL default config for out of the box Kamal deployments](https://github.com/rails/rails/pull/56010){:target="_blank"}
- [Kamal docs - Proxy](https://kamal-deploy.org/docs/configuration/proxy/){:target="_blank"}
