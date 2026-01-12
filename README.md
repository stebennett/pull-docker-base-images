# Pull Docker Base Images

A GitHub Action that scans directories for Dockerfiles and pulls all base images from `FROM` statements.

## Why Use This?

When building Docker images in CI, base images need to be pulled from registries. If you're using a private registry, you need to authenticate before building. This action:

- **Centralizes image discovery** - No need to maintain a separate list of base images
- **Simplifies private registry usage** - Log in once, then pull all base images before building
- **Reduces duplication** - Image tags are defined only in Dockerfiles, not repeated in CI config

## Prerequisites

If your Dockerfiles use base images from private registries, you must authenticate with `docker/login-action` (or equivalent) **before** running this action.

## Usage

```yaml
- name: Pre-pull base images
  uses: stebennett/pull-docker-base-images@v1
  with:
    directories: 'backend,web'
```

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `directories` | Yes | Comma-separated list of directories to scan for Dockerfiles |

## Example Workflow

```yaml
name: Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: your-registry.io
          username: ${{ secrets.REGISTRY_USER }}
          password: ${{ secrets.REGISTRY_TOKEN }}

      - name: Pre-pull base images
        uses: stebennett/pull-docker-base-images@v1
        with:
          directories: 'backend,web,services'

      - name: Build Docker image
        uses: docker/build-push-action@v6
        with:
          context: ./backend
          push: false
```

## Features

- Scans specified directories for all `Dockerfile*` files (including `Dockerfile.dev`, `Dockerfile.prod`, etc.)
- Handles multi-stage builds (`FROM node:20 AS builder`)
- Automatically deduplicates images across multiple Dockerfiles
- Skips `FROM scratch` (not a pullable image)
- Provides clear output showing which Dockerfiles were found and which images are being pulled

## How It Works

1. Parses the comma-separated `directories` input
2. Finds all files matching `Dockerfile*` in each directory
3. Extracts `FROM` statements from all Dockerfiles
4. Parses image names (handles `AS stage` syntax)
5. Deduplicates the image list
6. Pulls each unique image using `docker pull`

## Limitations

- Does not resolve `ARG` variables in `FROM` statements (e.g., `FROM $BASE_IMAGE`)
- Only scans top-level of each directory (not recursive)

## License

MIT
