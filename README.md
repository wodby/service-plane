# Plane service for Kubernetes on Wodby

Run Plane Community Edition project management on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Plane.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [Plane application stack](https://github.com/wodby/stack-plane)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `plane` |
| Type | Application service |
| Versions | `1` |
| Endpoints | `plane`: `http` 8080 |
| Service links | PostgreSQL, Redis, RabbitMQ and S3 storage, required; SMTP server |
| Settings | Admin email (required), Admin password (required), Admin first name, Allow sign-up, API workers, Background worker processes |
| Workloads | proxy (primary), web, admin, space, live, api, worker, scheduler |
| Chart | [`wodby/plane`](https://github.com/wodby/charts/tree/main/plane) |
| Images | Plane's own `makeplane/*` images |

## Use this service

Use this service through [Plane application stack](https://github.com/wodby/stack-plane), or reference `plane` from a
custom Wodby stack with PostgreSQL, Redis or Valkey, RabbitMQ and an S3-compatible storage service such as
[Silo](https://github.com/wodby/service-silo).

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
