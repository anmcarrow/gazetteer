# Traefik Configuration Guide

This docker-compose setup includes labels for integration with an external Traefik reverse proxy.

## Prerequisites

- Running Traefik instance with:
  - `web` entrypoint (typically port 80)
  - `websecure` entrypoint (typically port 443)
  - Let's Encrypt certificate resolver configured
  - External network named `traefik-network` (or configured differently in `.env`)

## Setup Instructions

### 1. Create the Traefik network (if it doesn't exist)

```bash
docker network create traefik-network
```

### 2. Configure your domain

Copy the example environment file and edit it:

```bash
cp .env.example .env
```

Edit `.env` and set your domain:

```env
DOMAIN=your-domain.com
TRAEFIK_NETWORK=traefik-network
CERT_RESOLVER=letsencrypt
```

### 3. Deploy the application

```bash
docker-compose up -d
```

## Configuration Details

The docker-compose.yml includes the following Traefik labels:

- **HTTP Router**: Listens on port 80 and redirects to HTTPS
- **HTTPS Router**: Serves the application with TLS certificate
- **Service**: Points to container port 80 (nginx)
- **Middleware**: Automatic HTTP to HTTPS redirect

### Labels Explained

```yaml
traefik.enable=true
# Enables Traefik for this container

traefik.http.routers.ouw-gazetteer.rule=Host(`your-domain.com`)
# HTTP router matches requests to your domain

traefik.http.routers.ouw-gazetteer-secure.tls.certresolver=letsencrypt
# Uses Let's Encrypt for automatic TLS certificates

traefik.http.services.ouw-gazetteer.loadbalancer.server.port=80
# Routes traffic to nginx on port 80 inside the container
```

## Local Development

For local development without Traefik, the container still exposes port 8001:

```bash
docker-compose up -d
```

Access at: http://localhost:8001

## Standalone vs Traefik Mode

The configuration supports both modes:

- **Standalone**: Access via `localhost:8001` (no Traefik needed)
- **Traefik**: Access via your configured domain with automatic HTTPS

To disable Traefik integration, set `traefik.enable=false` in labels or remove the `networks` section.

## Troubleshooting

### Container not accessible via domain

1. Check Traefik is running: `docker ps | grep traefik`
2. Check network exists: `docker network ls | grep traefik`
3. Check container is on network: `docker inspect muckraker-app | grep -A 10 Networks`
4. Check Traefik logs: `docker logs traefik`

### DNS not resolving

Ensure your domain's DNS A/AAAA records point to your server's IP address.

### Certificate issues

Check that your Traefik cert resolver name matches the one in `.env`:

```bash
docker exec traefik cat /etc/traefik/traefik.yml | grep certResolver
```
