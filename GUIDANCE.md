# Plane on Wodby

What Wodby sets up for Plane Community Edition on this service.

## Components

The service runs Plane's own images, all at the option's release, on the `wodby/plane` chart:

- `proxy` (primary, port 8080): Caddy, the only public port. It sends `/api`, `/auth` and `/static` to the API,
  `/god-mode` to the admin app, `/spaces` to the public space app, `/live` to the live server, the storage bucket
  (`/uploads`) to the linked S3 storage, and everything else to the web app.
- `web`, `admin`, `space`, `live`, `api`: the web app, God Mode, published projects, real-time page editing and the
  Django API. Each follows the replica count.
- `worker`: Celery workers (`worker-concurrency` setting, default 2 processes per replica), which send email and run
  imports, exports and notifications.
- `scheduler`: Celery beat, at most one replica, recreated on deploy. Its init container applies Plane's migrations,
  registers the instance and sets up the administrator on every deploy; the API and workers wait for the migrations.

Custom code or plugins are not supported: the images are Plane's own.

## Configuration

Link, setting and token variables reach every component through the chart's shared `env`.

- **Database.** `DATABASE_URL` from the `db` link (PostgreSQL).
- **Redis.** `REDIS_URL` from the `redis` link (Valkey or Redis): cache and the live server.
- **RabbitMQ.** `AMQP_URL` from the `amqp` link, default vhost. RabbitMQ 4 refuses the transient queues Celery uses for
  its control messages unless `transient_nonexcl_queues` is permitted; the Plane stack sets
  `RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS` on its RabbitMQ service for that.
- **Storage.** The `storage` link (an S3-compatible service such as Silo) gives `AWS_S3_ENDPOINT_URL`,
  `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `PLANE_S3_UPSTREAM`. The bucket is `uploads`; the API creates it.
  With `USE_MINIO=1` and `MINIO_ENDPOINT_SSL=1`, presigned URLs point at Plane's own HTTPS address, and the proxy
  forwards `/uploads` to the storage with the browser's `Host` header, which the signatures cover. Uploads are limited
  to `FILE_SIZE_LIMIT` (5 MB).
- **Email.** With a mail server linked as `sendmail`, Plane's SMTP settings point at it without TLS
  (`ENABLE_SMTP=1`), sending as `noreply@` the app's host. These, `ENABLE_SIGNUP` and the other instance settings are
  stored on the first deployment; change them later in God Mode, not in the service.
- **URLs.** `WEB_URL` and `CORS_ALLOWED_ORIGINS` are the app's main address.
- **Secrets.** `SECRET_KEY` and `LIVE_SERVER_SECRET_KEY` are generated tokens; changing `SECRET_KEY` signs everybody out.

## Installation and upgrades

- **First deployment:** the `admin-email` and `admin-password` settings (both required) are checked before anything
  is installed; Plane refuses weak passwords. The migrations run, then the administrator is created with
  `admin-name` as the first name, the instance is named after the app and marked set up, so `/god-mode` asks for a
  sign-in and nobody can claim the instance. Telemetry is off. Sign-up is off unless `allow-signup` is `1`.
- **Later deployments:** the migrations of a new release run first; the administrator is left as it is. Changing the
  admin settings afterwards does nothing; change the account in Plane.
- A new install takes several minutes, mostly Plane's migrations.

## Limits

- Plane Community Edition only.
- One bucket on the linked storage; no external S3 or CDN.
- No development workspace.

## Check the result

- `curl -s localhost:8080/-/health` in the proxy container returns `OK`.
- `curl -s https://<host>/api/instances/` shows `"is_setup_done": true`.
- The scheduler pod's `init` container log ends with `[plane-setup]` lines.
