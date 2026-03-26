# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository provides multi-platform Docker images for Tengine (Alibaba's Nginx distribution). The images support Lua scripting, PCRE2, GMSSL (via Tongsuo), GeoIP2, and Brotli compression.

## Build Commands

Build the default Alpine-based image:
```sh
docker build -t tengine:latest .
```

Build with a specific Tengine version:
```sh
docker build --build-arg TENGINE_VERSION=3.0.0 -t tengine:3.0.0 .
```

Build a specific OS variant:
```sh
docker build -f ubuntu/24.04/Dockerfile -t tengine:ubuntu24 .
docker build -f debian/12/Dockerfile -t tengine:debian12 .
docker build -f fedora/41/Dockerfile -t tengine:fedora41 .
docker build -f rockylinux/9/Dockerfile -t tengine:rockylinux9 .
```

Build multi-platform images:
```sh
docker buildx build --platform linux/amd64,linux/arm64 -t tengine:latest .
```

## Run Commands

Basic container:
```sh
docker run -d -p 80:80 -p 443:443 --name tengine tengine:latest
```

With custom configuration:
```sh
docker run -d -v /etc/nginx:/etc/nginx -v /data/public:/data/public -p 80:80 -p 443:443 --name tengine tengine:latest
```

Test Lua functionality:
```sh
curl http://localhost/get_lua_path
curl http://localhost/get_json
```

## Architecture

### Multi-Stage Dockerfile Structure
- **Builder stage**: Compiles Tengine and all dependencies from source
- **Runtime stage**: Minimal image with compiled binaries and runtime dependencies only

### Directory Structure
- `/etc/nginx/` - Configuration directory (nginx.conf, conf.d/, ssl/)
- `/opt/tengine/` - Tengine home with subdirectories:
  - `lualib/` - Lua libraries (lua-resty-core, lua-resty-lrucache, etc.)
  - `luamod/` - Lua C modules (cjson.so)
  - `luajit/` - LuaJIT installation
  - `Tongsuo/` - GMSSL/crypto library for national standards
  - `pcre2/`, `zlib/`, `geoip/` - Compiled dependencies
- `/var/lib/nginx/` - Temp directories (client_temp, proxy_temp, etc.)
- `/var/log/nginx/` - Log files
- `/docker-entrypoint.d/` - Initialization scripts (executed on container start)

### OS Variants
The repository maintains nearly identical Dockerfiles for each OS in separate directories:
- `alpine/3/Dockerfile` (default, symlinked from root)
- `ubuntu/24.04/Dockerfile`
- `debian/12/Dockerfile`
- `fedora/41/Dockerfile`
- `rockylinux/9/Dockerfile`

Key differences between variants are the package installation commands (`apk add` vs `apt install` vs `dnf install`).

### Key Compiled Components
- **Tengine**: Main server (from alibaba/tengine GitHub)
- **Tongsuo**: GMSSL/TLS support for Chinese national cryptographic standards
- **LuaJIT**: Just-in-time Lua compiler (openresty/luajit2)
- **PCRE2**: Regex with JIT enabled
- **ngx_brotli**: Brotli compression module
- **ngx_http_geoip2_module**: MaxMind GeoIP2 support
- **lua-nginx-module** & **stream-lua-nginx-module**: Lua scripting in nginx

### Environment Variables
- `LUA_PATH` / `LUA_CPATH`: Configured for Lua module discovery
- `PATH`: Includes custom binary directories for luajit, tongsuo, pcre2, geoip

## GitHub Actions

CI/CD builds and pushes images to Docker Hub on push to main. Each OS variant has its own workflow file in `.github/workflows/`. Workflows require:
- `DOCKER_USERNAME` / `DOCKER_PASSWORD` secrets
- `BUILD_VERSION` variable for versioning
