---
layout: "docs"
page_title: "task Stanza - Job Specification"
sidebar_current: "docs-job-specification-task"
description: |-
  The "task" stanza creates an individual unit of work, such as a Docker
  container, web application, or batch processing.
---

# `task` Stanza

The `task` stanza creates an individual unit of work, such as a Docker
container, web application, or batch processing.

```hcl
job "docs" {
  group "example" {
    task "server" {
      # ...
    }
  }
}
```

## `task` Parameters

- `artifact` <code>([Artifact][]: nil)</code> - Defines an artifact to download
  before running the task. This may be specified multiple times to download
  multiple artifacts.

- `config` `(map<string|string>: nil)` - Specifies the driver configuration,
  which is passed directly to the driver to start the task. The details of
  configurations are specific to each driver, so please see specific driver
  documentation for more information.

- `constraint` <code>([Constraint][]: nil)</code> - Specifies user-defined
  constraints on the task. This can be provided multiple times to define
  additional constraints.

- `driver` - Specifies the task driver that should be used to run the
  task. See the [driver documentation](/docs/drivers/index.html) for what
  is available. Examples include `docker`, `qemu`, `java`, and `exec`.

- `env` <code>([Env][]: nil)</code> - Specifies environment variables that will
  be passed to the running process.

- `kill_timeout` `(string: "5s")` - Specifies the duration to wait for an
  application to gracefully quit before force-killing. Nomad sends an `SIGINT`.
  If the task does not exit before the configured timeout, `SIGKILL` is sent to
  the task.

- `logs` <code>([Logs][]: nil)</code> - Specifies logging configuration for the
  `stdout` and `stderr` of the task.

- `meta` <code>([Meta][]: nil)</code> - Specifies a key-value map that annotates
  with user-defined metadata.

- `resources` <code>([Resources][]: required)</code> - Specifies the minimum
  resource requirements such as RAM, CPU, disk, and network.

- `service` <code>([Service][]: nil)</code> - Specifies integrations with
  [Consul][] for service discovery. Nomad automatically registers when a task
  is started and de-registers it when the task dies.

- `user` `(string: <varies>)` - Specifies the user that will run the task. This
  defaults to the same user as the Nomad client. This can only be set on Linux
  platforms.

## `task` Examples

The following examples only show the `task` stanzas. Remember that the
`task` stanza is only valid in the placements listed above.

### Docker Container

This example defines a task that starts a Docker container as a service. Docker
is just one of many drivers supported by Nomad. Read more about drivers in the
[Nomad drivers documentation](/docs/drivers/index.html).

```hcl
task "server" {
  driver = "docker"
  config {
    image = "hashicorp/http-echo"
    args  = ["-text", "hello world"]
  }

  resources {
    network {
      mbits = 250
    }
  }
}
```

### Metadata and Environment Variables

This example uses custom metadata and environment variables to pass information
to the task.

```hcl
task "server" {
  driver = "exec"
  config {
    command = "/bin/env"
  }

  meta {
    my-key = "my-value"
  }

  env {
    MY_KEY = "${meta.my-key}"
  }

  resources {
    cpu = 20
  }
}
```

### Service Discovery

This example creates a service in Consul. To read more about service discovery
in Nomad, please see the [Nomad service discovery documentation][service].

```hcl
task "server" {
  driver = "docker"
  config {
    image = "hashicorp/http-echo"
    args  = ["-text", "hello world"]
  }

  service {
    tags = ["default"]

    check {
      type     = "tcp"
      interval = "10s"
      timeout  = "2s"
    }
  }

  resources {
    cpu = 20
  }
}
```

[artifact]: /docs/job-specification/artifact.html "Nomad artifact Job Specification"
[consul]: https://www.consul.io/ "Consul by HashiCorp"
[constraint]: /docs/job-specification/constraint.html "Nomad constraint Job Specification"
[env]: /docs/job-specification/env.html "Nomad env Job Specification"
[meta]: /docs/job-specification/meta.html "Nomad meta Job Specification"
[resources]: /docs/job-specification/resources.html "Nomad resources Job Specification"
[logs]: /docs/job-specification/logs.html "Nomad logs Job Specification"
[service]: /docs/service-discovery/index.html "Nomad Service Discovery"
