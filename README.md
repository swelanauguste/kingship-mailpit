# Shared Mailpit

Mailpit captures development email and stores it in `./mailpit-data`.

## Start Mailpit

This project uses the same external `shared-proxy` Docker network as the
`shared-services` stack. The network must exist before starting Mailpit. If it
has not already been created, run:

```sh
docker network create shared-proxy
```

Start Mailpit:

```sh
docker compose up -d
```

The web inbox is available at <http://localhost:8025>. Host applications can
send SMTP mail to `localhost:1025`. Both published ports retain their existing
bindings to all host interfaces.

## Connect another Compose project

Attach the service that sends email to `shared-proxy` and configure its mail
settings to use host `shared-mailpit`, port `1025`, without authentication or
TLS. Environment variable names depend on the application; this example uses
`SMTP_HOST` and `SMTP_PORT`:

```yaml
services:
  app:
    # Keep your existing image/build and other settings.
    environment:
      SMTP_HOST: shared-mailpit
      SMTP_PORT: "1025"
    networks:
      - default
      - proxy

networks:
  default: {}
  proxy:
    external: true
    name: shared-proxy
```

Use the container port `1025` for SMTP between projects. `localhost` inside an
application container refers to that container itself.

Nginx on `shared-proxy` can reach the web inbox at `http://shared-mailpit:8025`.
Mailpit must be running before applications send mail; separate Compose
projects have independent startup lifecycles.
