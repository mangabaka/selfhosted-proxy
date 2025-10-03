# selfhosted-proxy

Turn-key self-hosted (caching) MangaBaka proxy

## About

This repository makes it easy to run a caching Nginx proxy in front of the MangaBaka API and CDN.

A setup like this is required if you want to use our API or CDN content directly on your own site, since we don't allow directly (un-proxied) usage of our services.

We prevent direct access via CORS, preventing JavaScript to query the API and the browser from fetching images.

## I already have my own Nginx setup / I don't like Docker / I don't like this project

No problem!

You can just grab the nginx configuration file in `config/nginx/default.conf.dist` and adapt it to your own setup - there are no dependencies or requirements - everything else in this repository are just for convenience and ease of use

## How it works

Nginx is configured as a caching proxy with the following endpoints available

- `/api/` proxying the MangaBaka API at `https://api.mangabaka.dev`
- `/cdn/` proxuing the MangaBaka CDN at `https://cdn.mangabaka.dev`

Meaning that

- `https://api.mangabaka.dev/v1/series/1` becomes `${your_host}/api/v1/series/1`
- `https://cdn.mangabaka.dev/imgproxy/plain/{preset}/{url}.{format}` becomes `${your_host}/cdn/imgproxy/plain/{preset}/{url}.{format}`

## Repository layout

The repo has the following structure:

- `compose.yml` Docker Compose config file
- `config/compose/nginx-proxy/` Docker Compose configuration for the Nginx proxy
- `config/compose/cloudflare-tunnel/` Docker Compose configuration for the Cloudflare Tunnel sidecar
- `config/nginx/default.conf.dist` Nginx configuration file
- `.env.default` environment file with default values

## Setup

Ensure Docker and Docker Compose is installed and reasonably modern

```shell
cp config/nginx/default.conf.dist config/nginx/default.conf
cp compose.yml.dist compose.yml
cp .env.dist .env
```

### Deployment 1: Expose Nginx to host network port

1. Uncomment the `config/compose/nginx-proxy/network-host-port` line in `compose.yml`
1. Review the host port in your `.env` file (`NGINX_HOST_PORT`)
1. `docker compose up -d proxy`
1. Done

### Deployment 2: Expose via Cloudflare Tunnel

1. Uncomment the two Cloudflare Tunnel lines in `compose.yml`
1. Review the `CLOUDFLARE_TUNNEL_TOKEN` key in your `.env` file
1. Use these settings in your `published application route` in Cloudflare Tunnel Web UI
    1. `Service -> Type`  must be `http`
    1. `Service -> URL` must be `mangabaka-proxy-nginx:80`
1. Create your tunnel and wait for DNS to propagate

### Verification

1. Accessing `/cdn/` (leading slash!) should return`OK`
1. Accessing `/api/v1/series/search?q=test` should return a bunch of JSON

## Customization

You can overwrite or extend any of the provided Docker Compose configuration by adding a service with the same name in `compose.yml`.

You can, of course, also just fork or edit the hell out of this repo.

## Contributing

We're always happy to review Pull Requests with improvements, features, documentation, bug fixes and what not.
