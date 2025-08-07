# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains Docker images ("yolks") for the Pterodactyl game server management panel. These are base images that provide runtime environments for various programming languages and game servers.

## Repository Structure

The codebase is organized into distinct image categories:

- **oses/** - Base OS images (Alpine, Debian) with core packages
- **games/** - Game-specific runtime environments (Rust servers, Source engine)
- **installers/** - Images used for installation scripts, not for running servers
- **java/** - Java runtime images (versions 8-21, including OpenJ9 variants)
- **nodejs/** - Node.js runtime images (versions 12-20)
- **python/** - Python runtime images (versions 3.7-3.11)
- **go/** - Golang runtime images (versions 1.14-1.17)

## Image Architecture

Each runtime follows a consistent pattern:
1. Version-specific Dockerfile in subdirectories (e.g., `java/17/Dockerfile`)
2. Shared entrypoint script at the language root (e.g., `java/entrypoint.sh`)
3. Multi-platform support (linux/amd64 and linux/arm64)

## Common Development Tasks

### Building Images Locally

```bash
# Build a specific image (example: Java 17)
docker build -t test-java17 -f java/17/Dockerfile java/

# Build with specific platform
docker build --platform linux/amd64 -t test-java17 -f java/17/Dockerfile java/
```

### Testing Images

```bash
# Run an image interactively
docker run -it --rm test-java17 /bin/bash

# Test with Pterodactyl startup command simulation
docker run -it --rm -e STARTUP="java -version" test-java17
```

### Adding a New Version

When adding a new version to an existing runtime:
1. Create a new subdirectory with the version number
2. Add the Dockerfile in that directory
3. Update the corresponding workflow file in `.github/workflows/`
4. Ensure the Dockerfile references the shared entrypoint script with `COPY ./../entrypoint.sh /entrypoint.sh`

## GitHub Actions Workflows

Workflows are located in `.github/workflows/`:
- **base.yml** - Builds OS base images
- **games.yml** - Builds game server images
- **java.yml** - Builds Java runtime images
- **nodejs.yml** - Builds Node.js runtime images
- **python.yml** - Builds Python runtime images
- **go.yml** - Builds Go runtime images
- **installers.yml** - Builds installer images

Images are automatically built:
- On push to master for changed paths
- Monthly via cron schedule
- Manually via workflow_dispatch

## Image Registry

All images are published to GitHub Container Registry (ghcr.io):
- Base OS: `ghcr.io/pterodactyl/yolks:{os}`
- Games: `ghcr.io/pterodactyl/games:{game}`
- Runtimes: `ghcr.io/pterodactyl/yolks:{language}_{version}`
- Installers: `ghcr.io/pterodactyl/installers:{os}`

## Important Conventions

1. All Dockerfiles must include the MIT license header
2. Images must run as the `container` user (UID 1000)
3. Working directory should be `/home/container`
4. Entrypoint scripts handle `{{VARIABLE}}` substitution for Pterodactyl
5. Include essential debugging tools (curl, iproute2, etc.) in base packages
6. Set `TZ` environment variable support for timezone configuration
7. Print runtime version information on startup for debugging