# AstroWind Deployment Guide for Coolify with Nixpacks

This guide explains how to deploy AstroWind using Coolify with Nixpacks buildpack and Traefik as a reverse proxy.

## Prerequisites

- VPS with Traefik running on ports 80, 443, 8000, 8080
- Coolify installed and configured
- Domain name pointing to your VPS

## Configuration Overview

The application is configured to:

- Use Nixpacks for automatic build detection
- Build with `npm run build` (Astro's built-in build process)
- Serve static files with `npx serve` on port 3000
- Use Traefik for SSL termination and routing
- Include health checks for monitoring

## Deployment Steps

### 1. Update Domain Configuration

Before deploying, update the domain in the `coolify.yaml` file:

```yaml
- 'traefik.http.routers.astrowind.rule=Host(`your-domain.com`)'
```

Replace `your-domain.com` with your actual domain.

### 2. Deploy with Coolify

1. **Connect your repository** to Coolify
2. **Use the `coolify.yaml`** configuration file
3. **Nixpacks will automatically detect** the Node.js/Astro project
4. **Set environment variables** if needed:
   - `NODE_ENV=production`
   - `PORT=3000`

### 3. Traefik Network Setup

Ensure your Traefik network exists:

```bash
docker network create traefik
```

### 4. Verify Deployment

After deployment, check:

- Container is running: `docker ps`
- Health check: Visit `https://your-domain.com/`
- Application: Visit `https://your-domain.com`

## Configuration Files

### Nixpacks Configuration (`nixpacks.toml`)

- Automatically installs Node.js and npm
- Runs `npm ci` to install dependencies
- Builds with `npm run build`
- Starts with `npx serve dist -p 3000`

### Coolify Configuration (`coolify.yaml`)

- **buildpack**: nixpacks
- **buildCommand**: npm run build
- **startCommand**: npx serve dist -p 3000
- **Traefik labels**: For reverse proxy integration

### Package.json Scripts

- **build**: `astro build` - Builds the static site
- **serve**: `npx serve dist -p 3000` - Serves the built files

## Traefik Labels Explained

- `traefik.enable=true`: Enable Traefik for this service
- `traefik.http.routers.astrowind.rule=Host(...)`: Route based on domain
- `traefik.http.routers.astrowind.entrypoints=websecure`: Use HTTPS
- `traefik.http.routers.astrowind.tls.certresolver=letsencrypt`: Auto SSL
- `traefik.http.services.astrowind.loadbalancer.server.port=3000`: Internal port

## Troubleshooting

### Build failures

- Check Node.js version compatibility (requires ^18.17.1 || ^20.3.0 || >= 21.0.0)
- Verify all dependencies are in package.json
- Check build logs in Coolify dashboard

### Container won't start

- Check if Traefik network exists
- Verify domain configuration
- Check logs: `docker logs astrowind`
- Ensure `serve` package is available (installed via npx)

### SSL issues

- Ensure Let's Encrypt resolver is configured in Traefik
- Check domain DNS settings
- Verify Traefik dashboard for certificate status

### Health check failures

- Ensure the application is running on port 3000
- Check if the root path `/` is accessible
- Verify container logs for errors

## Performance Optimizations

Nixpacks automatically provides:

- Optimized Node.js runtime
- Efficient dependency installation
- Static file serving with `serve`
- Traefik handles SSL termination and caching

## Monitoring

- Health check endpoint: `/` (root path)
- Traefik dashboard for routing status
- Container logs for application issues
- Coolify dashboard for build and deployment status

## Customization

To customize the deployment:

1. Update domain in `coolify.yaml`
2. Modify build/start commands in `coolify.yaml`
3. Adjust Traefik labels as needed
4. Add environment variables if required
5. Customize `nixpacks.toml` for specific build requirements

## Benefits of Nixpacks Approach

- **Simpler**: No custom Dockerfile needed
- **Automatic**: Nixpacks detects Astro project automatically
- **Optimized**: Uses Astro's built-in build process
- **Maintainable**: Less custom configuration to maintain
- **Fast**: Efficient builds with Nixpacks caching
