# AstroWind Deployment Guide for Coolify with Traefik

This guide explains how to deploy AstroWind using Coolify with Traefik as a reverse proxy.

## Prerequisites

- VPS with Traefik running on ports 80, 443, 8000, 8080
- Coolify installed and configured
- Domain name pointing to your VPS

## Configuration Overview

The application is configured to:

- Run internally on port 3000
- Use Traefik for SSL termination and routing
- Include health checks for monitoring
- Optimize static asset caching

## Deployment Steps

### 1. Update Domain Configuration

Before deploying, update the domain in the configuration files:

**In `docker-compose.yml` and `coolify.yaml`:**

```yaml
- 'traefik.http.routers.astrowind.rule=Host(`your-domain.com`)'
```

Replace `your-domain.com` with your actual domain.

### 2. Deploy with Coolify

1. **Connect your repository** to Coolify
2. **Use the `coolify.yaml`** configuration file
3. **Set environment variables** if needed:
   - `NODE_ENV=production`

### 3. Traefik Network Setup

Ensure your Traefik network exists:

```bash
docker network create traefik
```

### 4. Verify Deployment

After deployment, check:

- Container is running: `docker ps`
- Health check: Visit `https://your-domain.com/health`
- Application: Visit `https://your-domain.com`

## Configuration Files

### Nginx Configuration (`nginx/nginx.conf`)

- Listens on port 3000 internally
- Optimized for static assets
- Includes security headers
- Health check endpoint at `/health`

### Docker Configuration

- **Dockerfile**: Multi-stage build with Nginx
- **docker-compose.yml**: Traefik integration
- **coolify.yaml**: Coolify-specific configuration

## Traefik Labels Explained

- `traefik.enable=true`: Enable Traefik for this service
- `traefik.http.routers.astrowind.rule=Host(...)`: Route based on domain
- `traefik.http.routers.astrowind.entrypoints=websecure`: Use HTTPS
- `traefik.http.routers.astrowind.tls.certresolver=letsencrypt`: Auto SSL
- `traefik.http.services.astrowind.loadbalancer.server.port=3000`: Internal port

## Troubleshooting

### Container won't start

- Check if Traefik network exists
- Verify domain configuration
- Check logs: `docker logs astrowind`

### SSL issues

- Ensure Let's Encrypt resolver is configured in Traefik
- Check domain DNS settings
- Verify Traefik dashboard for certificate status

### Health check failures

- Ensure Nginx is running on port 3000
- Check if `/health` endpoint is accessible
- Verify container logs for errors

## Performance Optimizations

The configuration includes:

- Gzip compression
- Static asset caching (1 year)
- Security headers
- Optimized Nginx settings

## Monitoring

- Health check endpoint: `/health`
- Traefik dashboard for routing status
- Container logs for application issues

## Customization

To customize the deployment:

1. Update domain in configuration files
2. Modify Nginx settings in `nginx/nginx.conf`
3. Adjust Traefik labels as needed
4. Add environment variables if required
