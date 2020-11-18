# Apache Guacamole in a Container with TLS

This Docker Compose setup makes it very easy (only 3 cli commands) to run [Apache Guacamole](https://guacamole.incubator.apache.org/) behind a NGINX reverse proxy TLS secured with Let's Encrypt.

*Although this repo is 3 years old it is still up to date and uses the latest images.*

## Unique Features

* Unlike other solutions this setup is much simpler to setup and is inline with docker/docker-compse best practice.
* Here we use official Apache Guacamole Docker Images [guacamole/guacamole:latest](https://hub.docker.com/r/guacamole/) always up to date.
* Automatically created and configured Nginx Reverse Proxy in front of the Guacamole Service.
* TLS encrypted traffic with Let's Encrypt for your public domain.
* Minimal configuration of only *two* mandatory environment variables.

## Run

Before you start the service, define three mandatory variables.
The easiest way is to create a `.env` file in your working directory eg.:

### Step 1 - Define your Domain and DB Password

```ini
cut > .env <<EOF
POSTGRES_PASSWORD=*****
VIRTUAL_HOST=workshop.8gears.com
LETSENCRYPT_EMAIL=user@domain.com
EOF
```

Run the service `init-guac-db` once before starting all other services. This one off job will export the application database schema so Postgres can pick it up when it starts and initialize the database with values and schema for Guacamole.

### Step 2 - Populate DB

Copy the `docker-compose.yml` from this repository to your computer and do:

```sh
docker-compose up init-guac-db
```
The job should start and terminate after the schema is created:


### Step 3 - Start Guacamole and other Services:

```sh
docker-compose up -d
```

Now go to your application https://workshop.domain.org/guacamole and login as guacadmin/guacadmin. 
Don't forget to change the password in the next step.
