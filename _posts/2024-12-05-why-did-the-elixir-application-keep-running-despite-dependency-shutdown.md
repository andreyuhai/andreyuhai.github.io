---
title: Why did the Elixir application keep running despite dependency shutdown?
date: 2024-12-05
tags: [elixir, erlang, applications, application-restart-strategy]
description: Discover how we resolved a production incident in an Elixir-based RPC service caused by PgBouncer downtime and application restart strategies. Explore the impact of restart_type on applications.
---

Recently, we've had an incident with one of our RPC services on production. The root cause was an issue with a PgBouncer instance going down. Although PgBouncer was eventually restored and became operational, our service remained partially functional until the pods were restarted. In this article, I’ll document the root cause of the issue and how we fixed it, effectively answering the question posed in the title.

## The incident

During the incident, our service appeared to be partially functional. While we were still receiving and responding to some RPC requests, others were failing due to errors like this:

```elixir
** (RuntimeError) could not lookup Ecto repo Platform.Repo because it was not started or it does not exist
    (ecto 3.10.3) lib/ecto/repo/registry.ex:22: Ecto.Repo.Registry.lookup/1
    (ecto 3.10.3) lib/ecto/repo/supervisor.ex:160: Ecto.Repo.Supervisor.tuplet/2
    (platform 0.0.1) lib/repo.ex:6: Platform.Repo.all/2
```

That means we could only respond to requests that didn't interact with `Platform.Repo`, however that error still didn't tell us much, except that the `Platform.Repo` we were looking for in the registry couldn't be found for some reason while handling an RPC call. But why was this happening?

## Root cause

Upon further investigation we've found logs below from that service's pods.

```elixir
Application platform exited: shutdown
```

That meant our application `platform`, one of our RPC service's dependencies, which manages `Platform.Repo` had shut down.
But if `platform` was down, how was our service still running?
The answer lay in how the application was started. Since this was one of our legacy apps, it was still using Mix (instead of releases) to start the server on production.
Specifically we were using a task to start the application and all its dependencies like this:

```elixir
Application.ensure_all_started(:our_legacy_rpc)
```

Here is the key difference:
In a `Mix` release, applications are started with a `restart_type` of `permanent`. However when using `Application.ensure_all_started/1`, the `restart_type` defaults to `temporary`. According to the documentation:

> :temporary - if app terminates, it is reported but no other applications are terminated (the default).

This explains why our service kept running even though one of its critical dependencies (`platform`) had shut down. Erlang does nothing about shutdown applications, which makes sense, since the top-level supervisor of the application will attempt to restart its children up to `max_retries` within `max_seconds` ([source](https://hexdocs.pm/elixir/Supervisor.html#module-supervisor-strategies-and-options)) before giving up and eventually shutting down the whole application.

## Resolution

To mitigate the issue we basically started all applications with a `restart_type` of `permanent` like so:

```elixir
Application.ensure_all_start(:our_legacy_rpc, type: :permanent)
```

This guarantees that, if any dependency of our application (or the application itself) goes down for any reason, everything will be shutdown which will trigger Kubernetes to restart the container because of failing liveness checks.

> :permanent - if app terminates, all other applications and the entire node are also terminated.

### Sources

Some links that helped me understand how Erlang manages applications and the role of `restart_type`.

- http://erlang.org/documentation/doc-5.2/doc/design_principles/applications.html
- https://blog.plataformatec.com.br/2015/04/build-embedded-and-start-permanent-in-elixir-1-0-4/
- https://stackoverflow.com/a/44665998/4796762
- https://stackoverflow.com/a/39662718/4796762
