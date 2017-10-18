# Guacamole in a Container Secured with TLS

This Compose file allows you to run Apache Guacamole behind NGINX reverse proxy with TLS secured.

## Unique Features

* Unlike the other compose examples this has a much simpler setup and complies with common docker practice.
* Image is based on official Apache Guacamole Docker Images
* Nginx Reverse Proxy in front of Guacamole.
* TLS encrypted traffic with Let's Encrypt Support

## Run

Before you start teh service some minimal configuration is needed.
The easiest way is to create a `.env` file in your workdir with the following four required entires:

```ini
cut > .env <<EOF
POSTGRES_USER=guacadb
POSTGRES_PASSWORD=*****
VIRTUAL_HOST=workshop.8gears.com
LETSENCRYPT_EMAIL=example@8gears.com
EOF
```

It is advised to run db `init-guac-db` once at the beginning to export the database schema so Postgres can pick it up and init the database. Otherwise it is not guaranteed that the file `initdb.sql` is created before postgres starts.

```sh
docker-compose up init-guac-db
```

Finally start Guacamole Services:
```sh
docker-compose up -d
```

Now go to your application https://workshop.domain.org/guacamole and login as guacadmin/guacadmin. 
Don't forget to change the password in the next step.
