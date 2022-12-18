# searxng-docker

Create a new SearXNG  instance in five minutes using Docker ready to be used behind a proxy like nginx.

## What is included ?

| Name | Description | Docker image | Dockerfile |
| -- | -- | -- | -- |
| [SearXNG](https://github.com/searxng/searxng) | SearXNG by itself | [searxng/searxng:latest](https://hub.docker.com/r/searxng/searxng) | [Dockerfile](https://github.com/searxng/searxng/blob/master/Dockerfile) |
| [Redis](https://github.com/redis/redis) | In-memory database | [redis:alpine](https://hub.docker.com/_/redis) | [Dockerfile-alpine.template](https://github.com/docker-library/redis/blob/master/Dockerfile-alpine.template) |

## How to use it
- [Install docker](https://docs.docker.com/install/)
- [Install docker-compose](https://docs.docker.com/compose/install/) (be sure that docker-compose version is at least 1.9.0)
- install nginx
- Get searxng-docker
  ```sh
  cd /usr/local
  git clone https://github.com/cronyakatsuki/searxng-docker.git
  cd searxng-docker
  ```
- Generate the secret key ```sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml```
- Edit the [searxng/settings.yml](https://github.com/searxng/searxng-docker/blob/master/searxng/settings.yml) file according to your need
- Check everything is working: ```docker-compose up```
- Run SearXNG in the background: ```docker-compose up -d```

## How to access the logs
To access the logs from all the containers use: `docker-compose logs -f`.

To access the logs of one specific container:
- SearXNG: `docker-compose logs -f searxng`
- Redis: `docker-compose logs -f redis`

### Start SearXNG with systemd

You can skip this step if you don't use systemd.

- ```cp searxng-docker.service.template searxng-docker.service```
- edit the content of ```WorkingDirectory``` in the ```searxng-docker.service``` file (only if the installation path is different from /usr/local/searxng-docker)
- Install the systemd unit:
  ```sh
  systemctl enable $(pwd)/searxng-docker.service
  systemctl start searxng-docker.service
  ```
## Basic nginx config
You can skip this step if you wan't to just run it locally

Create a new config for searxng and add paste this.
```
server {

    listen 80;
    listen [::]:80;

    # Your server name
    server_name searx.example.org ;

    # If you want to log user activity, comment these
    access_log /dev/null;
    error_log  /dev/null;

    # X-Frame-Options (XFO) header set to DENY
    add_header X-Frame-Options "DENY";

    # HTTP Strict Transport Security (HSTS) header
    add_header Strict-Transport-Security "max-age=63072000; always";

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # Content Security Policy (CSP)
    add_header Content-Security-Policy "default-src 'self';";

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header Connection       $http_connection;
        proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header X-Scheme         $scheme;
        proxy_buffering                   off;
    }
}
```

Then you can use certbot or any other tool to create ssl certificates.
Make sure to restart nginx.

## Note on the image proxy feature

The SearXNG image proxy is activated by default.

The default [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) allow the browser to access to ```${SEARXNG_HOSTNAME}``` and ```https://*.tile.openstreetmap.org;```.

If some users wants to disable the image proxy, you have to modify [./Caddyfile](https://github.com/searxng/searxng-docker/blob/master/Caddyfile). Replace the ```img-src 'self' data: https://*.tile.openstreetmap.org;``` by ```img-src * data:;```.

## Multi Architecture Docker images

Supported architecture:
- amd64
- arm64
- arm/v7

## How to update ?

To update the SearXNG stack:

```sh
docker-compose pull
docker-compose down
docker-compose up
```

To update this `docker-compose.yml` file:

Check out the newest version on github: [searxng/searxng-docker](https://github.com/searxng/searxng-docker).
