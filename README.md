# Apache Guacamole in a Container Secured with TLS

This Docker Compose setup makes it very easy run [Apache Guacamole](https://guacamole.incubator.apache.org/) behind a NGINX reverse proxy with TLS secured.

## Unique Features

* Unlike the other compose examples this has a much simpler setup and complies with common docker practice.
* Image is based on official Apache Guacamole Docker Images
* Nginx Reverse Proxy in front of Guacamole.
* TLS encrypted traffic with Let's Encrypt Support

## Run

Before you start the service configuration three mandatory parameters.
The easiest way is to create a `.env` file in your workdir like this:

```ini
cut > .env <<EOF
POSTGRES_PASSWORD=*****
VIRTUAL_HOST=workshop.8gears.com
LETSENCRYPT_EMAIL=example@8gears.com
EOF
```

It is advised to run the service `init-guac-db` once before starting all other services. This one off job will export the application database schema so Postgres can pick it up later and init the database with those values.

```sh
docker-compose -f https://raw.githubusercontent.com/8gears/containerized-guacamole/master/docker-compose.yml up init-guac-db
```

Finally start Guacamole Services:

```sh
docker-compose -f https://raw.githubusercontent.com/8gears/containerized-guacamole/master/docker-compose.yml up -d
```

Now go to your application https://workshop.domain.org/guacamole and login as guacadmin/guacadmin. 
Don't forget to change the password in the next step.
