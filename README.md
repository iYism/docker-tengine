# Tengine Docker Image

[![Build](https://github.com/IYism/docker-tengine/actions/workflows/build-alpine3-image.yml/badge.svg)](https://github.com/IYism/docker-tengine/actions/workflows/build-alpine3-image.yml)
[![Docker Pulls](https://img.shields.io/docker/pulls/sungyism/tengine.svg)](https://hub.docker.com/r/sungyism/tengine)
[![Docker Image Size](https://img.shields.io/docker/image-size/sungyism/tengine/latest)](https://hub.docker.com/r/sungyism/tengine)
[![License](https://img.shields.io/badge/License-BSD%202--Clause-blue.svg)](LICENSE)

> [!NOTE]
> This repository is maintained with [Claude Code](https://claude.ai/code). All changes are AI-driven.

A production-ready, multi-architecture Docker image for [Tengine](https://github.com/alibaba/tengine), Alibaba's high-performance web server based on Nginx. Optimized for modern web applications with Lua scripting, GMSSL support, and advanced compression.

## Features

| Feature | Description |
|---------|-------------|
| **GMSSL/NTLS** | National cryptographic algorithms (SM2/SM3/SM4) via [Tongsuo](https://github.com/Tongsuo-Project/Tongsuo) |
| **Lua Scripting** | Dynamic request handling with [LuaJIT](https://github.com/openresty/luajit2) and [lua-nginx-module](https://github.com/openresty/lua-nginx-module) |
| **PCRE2 JIT** | High-performance regular expressions with JIT compilation |
| **Brotli Compression** | Modern compression algorithm for faster content delivery |
| **GeoIP2** | IP geolocation using MaxMind databases |
| **Multi-Platform** | Supports `linux/amd64` and `linux/arm64` architectures |
| **Multi-OS** | Available for Alpine, Ubuntu, Debian, Fedora, and Rocky Linux |

## Quick Start

```bash
# Pull from Docker Hub
docker pull sungyism/tengine:latest

# Run container
docker run -d -p 80:80 -p 443:443 --name tengine sungyism/tengine:latest

# Test Lua functionality
curl http://localhost/get_json
```

## Available Tags

| Tag | Base Image | Architecture |
|-----|------------|--------------|
| `latest` | Alpine 3 | amd64, arm64 |
| `3.0.0-alpine-3` | Alpine 3 | amd64, arm64 |
| `3.0.0-ubuntu-24.04` | Ubuntu 24.04 | amd64, arm64 |
| `3.0.0-debian-12` | Debian 12 | amd64, arm64 |
| `3.0.0-fedora-41` | Fedora 41 | amd64 |
| `3.0.0-rockylinux-9` | Rocky Linux 9 | amd64 |

## Usage

### Basic Usage

```bash
docker run -d \
  -p 80:80 \
  -p 443:443 \
  --name tengine \
  sungyism/tengine:latest
```

### Custom Configuration

```bash
docker run -d \
  -p 80:80 \
  -p 443:443 \
  -v /path/to/nginx.conf:/etc/nginx/nginx.conf \
  -v /path/to/conf.d:/etc/nginx/conf.d \
  -v /path/to/ssl:/etc/nginx/ssl \
  -v /path/to/html:/data/public \
  --name tengine \
  sungyism/tengine:latest
```

### Docker Compose

```yaml
version: '3.8'
services:
  tengine:
    image: sungyism/tengine:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./conf.d:/etc/nginx/conf.d
      - ./ssl:/etc/nginx/ssl
    restart: unless-stopped
```

### GMSSL/NTLS Configuration

```nginx
server {
    listen 443 ssl;
    server_name localhost;

    enable_ntls on;

    # SM2 sign certificate
    ssl_sign_certificate /etc/nginx/ssl/SS.cert.pem;
    ssl_sign_certificate_key /etc/nginx/ssl/SS.key.pem;

    # SM2 encryption certificate
    ssl_enc_certificate /etc/nginx/ssl/SE.cert.pem;
    ssl_enc_certificate_key /etc/nginx/ssl/SE.key.pem;

    ssl_ciphers "ECC-SM2-SM4-CBC-SM3:ECC-SM2-SM4-GCM-SM3:ECDHE-SM2-SM4-CBC-SM3:ECDHE-SM2-SM4-GCM-SM3";
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        return 200 "GMSSL connection established: $ssl_protocol";
    }
}
```

## Build from Source

```bash
# Build default Alpine image
docker build -t tengine:local .

# Build with specific Tengine version
docker build --build-arg TENGINE_VERSION=3.0.0 -t tengine:3.0.0 .

# Build specific OS variant
docker build -f ubuntu/24.04/Dockerfile -t tengine:ubuntu24 .
docker build -f debian/12/Dockerfile -t tengine:debian12 .
docker build -f fedora/41/Dockerfile -t tengine:fedora41 .
docker build -f rockylinux/9/Dockerfile -t tengine:rockylinux9 .

# Build multi-platform image
docker buildx build --platform linux/amd64,linux/arm64 -t tengine:latest .
```

## Built-in Modules

<details>
<summary>Click to expand module list</summary>

### Standard Nginx Modules

- threads, file-aio, http_ssl, http_v2, http_realip
- http_addition, http_xslt, http_image_filter, http_geoip
- http_sub, http_dav, http_flv, http_mp4
- http_gunzip, http_gzip_static, http_auth_request
- http_random_index, http_secure_link, http_degradation
- http_slice, http_stub_status, mail, mail_ssl
- stream, stream_ssl, stream_realip, stream_geoip
- stream_ssl_preread, stream_sni

### Tengine Built-in Modules

| Module | Description |
|--------|-------------|
| ngx_tongsuo_ntls | GMSSL/NTLS national cryptography support |
| mod_config | Dynamic configuration |
| mod_dubbo | Dubbo RPC integration |
| mod_strategy | Load balancing strategy |
| ngx_http_concat_module | CSS/JS concatenation |
| ngx_http_proxy_connect_module | HTTP CONNECT method support |
| ngx_http_reqstat_module | Request statistics |
| ngx_http_sysguard_module | System resource protection |
| ngx_http_upstream_check_module | Active health checks |
| ngx_http_upstream_dynamic_module | Dynamic upstream resolution |
| ngx_http_upstream_dyups_module | Dynamic upstream management |
| ngx_http_upstream_session_sticky_module | Session persistence |
| ngx_http_upstream_vnswrr_module | Virtual node smooth weighted round-robin |

### Third-party Modules

| Module | Description |
|--------|-------------|
| [ngx_devel_kit](https://github.com/vision5/ngx_devel_kit) | Nginx Development Kit |
| [echo-nginx-module](https://github.com/openresty/echo-nginx-module) | Echo directives for debugging |
| [headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module) | Set/clear input/output headers |
| [ngx_http_geoip2_module](https://github.com/leev/ngx_http_geoip2_module) | GeoIP2 lookup |
| [ngx_brotli](https://github.com/google/ngx_brotli) | Brotli compression |
| [lua-nginx-module](https://github.com/openresty/lua-nginx-module) | Embed Lua into Nginx |
| [stream-lua-nginx-module](https://github.com/openresty/stream-lua-nginx-module) | Lua for stream module |

### Lua Libraries

| Library | Version |
|---------|---------|
| [lua-resty-core](https://github.com/openresty/lua-resty-core) | 0.1.31 |
| [lua-resty-lrucache](https://github.com/openresty/lua-resty-lrucache) | 0.15 |
| [lua-resty-lock](https://github.com/openresty/lua-resty-lock) | 0.09 |
| [lua-resty-string](https://github.com/openresty/lua-resty-string) | 0.16 |
| [lua-cjson](https://github.com/openresty/lua-cjson) | 2.1.0.14 |

</details>

## Directory Structure

| Path | Description |
|------|-------------|
| `/etc/nginx/` | Configuration directory |
| `/etc/nginx/nginx.conf` | Main configuration file |
| `/etc/nginx/conf.d/` | Virtual host configurations |
| `/etc/nginx/ssl/` | SSL/TLS certificates |
| `/opt/tengine/` | Tengine home directory |
| `/opt/tengine/lualib/` | Lua libraries |
| `/opt/tengine/luamod/` | Lua C modules |
| `/var/log/nginx/` | Log files |
| `/docker-entrypoint.d/` | Initialization scripts |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `LUA_PATH` | Lua module search path |
| `LUA_CPATH` | Lua C module search path |

## Exposed Ports

- `80` - HTTP
- `443` - HTTPS

## Contributing

Contributions are welcome! Please feel free to submit issues, pull requests, or suggestions.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Tengine](https://github.com/alibaba/tengine) - High-performance web server based on Nginx
- [Tongsuo](https://github.com/Tongsuo-Project/Tongsuo) - Modern cryptographic library
- [OpenResty](https://github.com/openresty) - Lua modules and LuaJIT
