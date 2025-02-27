---
layout: single
title: "Simplify Commands: How to add and use alias in Kamal?"
description: &description >
  Unlock the magic of aliases in Kamal to streamline your workflow.
  Learn how to set up custom aliases for tasks like running the Rails console or accessing logs.
  Boost your productivity - explore the guide!
excerpt: *description
date: 2025-02-11
tags:
  - Alias
header:
  og_image: /assets/images/opengraph/2025-02-11-how-to-add-alias-with-kamal.png
---

With the introduction of
[aliases](https://kamal-deploy.org/docs/configuration/aliases/)
in Kamal 2.0 it's easier to add aliases to the `deploy.yml` file and run them using the `kamal` command.

Kamal setup provides a few default aliases like `console`, `shell`, `logs`, and `dbc`.
You can add your own aliases to the `deploy.yml` file.

```yaml
# config/deploy.yml
aliases:
  console: app exec --interactive --reuse "bin/rails console"
  shell: app exec --interactive --reuse "bash"
  logs: app logs -f
  dbc: app exec --interactive --reuse "bin/rails dbconsole"
```

Now you can run the alias using the `kamal` command.

**Open rails console**

```bash
$ kamal console

Loading production environment (Rails 8.0.1)
meetup(prod)> Post.count
# => 1
```

**Open bash**
```bash
$ kamal shell

rails@37:/rails$ ls
Gemfile  Gemfile.lock  README.md  Rakefile  app  bin  config  config.ru  db  lib  log  public  script  storage  test  tmp  vendor
```

**See rails app logs**
```bash
$ kamal logs

INFO Following logs on 37.xx.xxx.xx.
```

**Open rails dbconsole (sqlite, psql or whatever database you are using)**

```bash
$ kamal dbc

SQLite version 3.40.1 2022-12-28 14:03:47
Enter ".help" for usage hints.

sqlite> select count(*) from posts;
1
```
