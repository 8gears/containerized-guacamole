# Apache Guacamole in a Container with TLS

This Docker Compose setup makes it very easy (only 3 cli commands) to run [Apache Guacamole](https://guacamole.apache.org/) behind a NGINX reverse proxy TLS secured with Let's Encrypt.

> [!NOTE]
> Verified working on 15 September 2026 with Docker 29.7 and Compose v5.3.
> The whole path was exercised end to end: database schema import, proxy routing,
> the Let's Encrypt HTTP-01 challenge and a Guacamole REST login.

## Unique Features

* Unlike other solutions this setup is much simpler to setup and is inline with docker/docker-compose best practice.
* Here we use official Apache Guacamole Docker Images [guacamole/guacamole:latest](https://hub.docker.com/r/guacamole/) always up to date.
* Automatically created and configured Nginx Reverse Proxy in front of the Guacamole Service, using [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) and [acme-companion](https://github.com/nginx-proxy/acme-companion).
* TLS encrypted traffic with Let's Encrypt for your public domain.
* Minimal configuration of only *three* mandatory environment variables.

## Run

Before you start the service, define the mandatory variables.
The easiest way is to create a `.env` file in your working directory eg.:

### Step 1 - Define your Domain and DB Password

```ini
cat > .env <<EOF
POSTGRES_PASSWORD=*****
VIRTUAL_HOST=workshop.8gears.com
LETSENCRYPT_HOST=workshop.8gears.com
LETSENCRYPT_EMAIL=user@domain.com
EOF
```

`VIRTUAL_HOST` is the name NGINX routes on, `LETSENCRYPT_HOST` is the name the
certificate is issued for. Keep them identical unless you serve the same
instance under several names, in which case `VIRTUAL_HOST` takes a comma
separated list and `LETSENCRYPT_HOST` only the public one.

> [!NOTE]
> The domain in `LETSENCRYPT_HOST` must already resolve to this host and ports
> 80 and 443 must be reachable from the internet. Let's Encrypt validates by
> fetching `http://<your-domain>/.well-known/acme-challenge/...` over plain
> port 80, so an HTTP-only firewall rule or a NAT that forwards 443 alone
> makes issuance fail.

### Step 2 - Populate DB

Copy the `docker-compose.yml` from this repository to your computer.

Run the service `init-guac-db` once before starting all other services. This one off job will export the application database schema so Postgres can pick it up when it starts and initialize the database with values and schema for Guacamole.

```sh
docker compose up init-guac-db
```

The job should start and terminate after the schema is created. Running it again
is harmless, it detects the existing dump and skips.

### Step 3 - Start Guacamole and other Services

Finally we can start Guacamole.

```sh
docker compose up -d
```

Now go to `https://workshop.8gears.com/guacamole` and login as `guacadmin` / `guacadmin`.
Don't forget to change the password in the next step.

## Advanced Topics

### Extensions

Extensions (`.jar`) are added through a template `GUACAMOLE_HOME`, not by
mounting into `/opt/guacamole/extensions`. The container generates a fresh
`GUACAMOLE_HOME` under `/tmp` on every start and symlinks the template contents
into it, so anything written directly into `/opt/guacamole/extensions`
disappears on restart.

Uncomment the following in the `guac` service and drop your jars into `./extensions`:

```yaml
    environment:
      GUACAMOLE_HOME: /etc/guacamole
    volumes:
      - ./extensions:/etc/guacamole/extensions:ro
```

See also a [working example](https://github.com/8gears/containerized-guacamole/issues/3#issuecomment-932015027) from contributor @marekschneider.

### Testing certificates

Set `LETSENCRYPT_TEST=true` in your `.env` to use the Let's Encrypt staging CA
while you are still sorting out DNS or firewall rules. Staging certificates are
not trusted by browsers but do not count against the rate limits.

### Image pinning

`nginx-proxy` and `acme-companion` are pinned to their release lines
(`1.11-alpine` and `2.8`) rather than the floating `alpine` / `latest` tags,
which track untagged commits past the last release.

### PostgreSQL version

The `postgres` image is pinned to `17-alpine` on purpose.

> [!NOTE]
> PostgreSQL 18 moved `PGDATA` from `/var/lib/postgresql/data` to
> `/var/lib/postgresql/18/docker`. Following `postgres:latest` past 17 makes the
> container refuse to start against the volume layout used here and writes its
> real data into an anonymous volume. Upgrade deliberately with `pg_upgrade`,
> not by moving the tag.

## Troubleshooting

### `cannot create /init/initdb.sql: Permission denied`

The Guacamole image runs as uid 1001 while Docker creates named volumes owned by
root. The `init-guac-db` service therefore runs as `user: root`, which is already
set in the compose file. If you copied an older `docker-compose.yml`, add it.

### Let's Encrypt cannot reach port 80

The proxy serves `/.well-known/acme-challenge/` on plain HTTP without redirecting
to HTTPS, so a `Connection refused` here is almost always outside the compose
project: port 80 blocked upstream, DNS not pointing at this host, or another
service already bound to port 80. Check with:

```sh
docker compose logs nginx-letsencrypt
curl -I http://<your-domain>/.well-known/acme-challenge/test
```

### Certificates are reissued on every restart

Make sure the `acme` volume is present and mounted at `/etc/acme.sh` in the
`nginx-letsencrypt` service. Without it the ACME account key is lost on restart
and you will hit Let's Encrypt rate limits.
