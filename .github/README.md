![Docker Image Size (tag)](https://img.shields.io/docker/image-size/brammys/terraria/latest)

# Terraria Server

This image runs the official **Vanilla Terraria** dedicated server inside a container. The image is built for multiple architectures (amd64 and arm64) and is fully configurable via environment variables.

## Available tags

The images are available on [Docker Hub](https://hub.docker.com/r/brammys/terraria) and [ghcr.io](https://github.com/BrammyS/Terraria/pkgs/container/terraria).
The tags following pattern is used:
- `terraria:latest` - Latest release version
- `terraria:<MAJOR>.<MINOR>.<PATCH>` - Release version (e.g. `terraria:1.4.5`)
- `terraria:<MAJOR>.<MINOR>.<PATCH>.<BUILD>` - Specific build version (e.g. `terraria:1.4.5.1`)

## Environment variables

All of the following environment variables are supported by the image entrypoint (they map directly to Terraria server CLI flags).

| Environment variable | Default | Notes |
|---|---|---|
| `TERRARIA_CONFIG` | `/data/configs/serverconfig.txt` | Specifies a configuration file to use |
| `TERRARIA_PORT` |  | Specifies the port to listen on |
| `TERRARIA_IP` |  | Bind IP address |
| `TERRARIA_PASSWORD` |  | Server password (masked in logs) |
| `TERRARIA_SECURE` | `0` | Enable by setting to `1` |
| `TERRARIA_NOUPNP` | `0` | Disable UPnP by setting to `1` |
| `TERRARIA_MAXPLAYERS` |  |  |
| `TERRARIA_MOTD` |  |  |
| `TERRARIA_FORCEPRIORITY` |  |  |
| `TERRARIA_WORLD` |  | Load a world and automatically start the server |
| `TERRARIA_AUTOCREATE` |  | `1`=small, `2`=medium, `3`=large |
| `TERRARIA_SEED` |  | Specifies the world seed when using -autocreate |
| `TERRARIA_BANLIST` |  |  |
| `TERRARIA_DISABLEANNOUNCEMENTBOX` | `0` | Enable by setting to `1` |
| `TERRARIA_ANNOUNCEMENTBOXRANGE` |  | Sets the announcement box text messaging range in pixels, -1 for serverwide announcements. |
| `TERRARIA_EXTRA_ARGS` |  | Appended verbatim to the server command |

## Volumes

The image uses the following path for persistent data:

- Data: `/data`
  - Worlds: `/data/worlds`
  - Configs: `/data/configs`

The image includes a default config file at:

- `/data/configs/serverconfig.txt`

## Examples

### Minimal (named volumes)

```bash
docker run -it --name terraria \
  -p 7777:7777/tcp -p 7777:7777/udp \
  -v terraria_data:/data \
  brammys/terraria
```

### Autocreate a world (named volumes)

```bash
docker run -it --name terraria \
  -p 7777:7777/tcp -p 7777:7777/udp \
  -v terraria_data:/data \
  -e TERRARIA_AUTOCREATE=1 \
  -e TERRARIA_WORLD=testing \
  -e TERRARIA_SEED=testing \
  brammys/terraria
```

### Use a specific world file path

Place a world file in your local `./terraria_data/worlds/` folder, e.g. `your_own_world.wld`, then run:

```bash
docker run -it --name terraria \
  -p 7777:7777/tcp -p 7777:7777/udp \
  -v ./terraria_data:/data \
  -e TERRARIA_WORLD=your_own_world \
  brammys/terraria
```

## docker-compose

Create a `docker-compose.yml` like this:

```yaml
services:
  terraria:
    image: brammys/terraria
    container_name: terraria
    restart: unless-stopped
    ports:
      - "7777:7777/tcp"
      - "7777:7777/udp"
    environment:
      # Examples (optional):
      # TERRARIA_AUTOCREATE: "1"
      # TERRARIA_SEED: "testing"
      # TERRARIA_WORLD: "testing_world"
      # TERRARIA_MAXPLAYERS: "16"
      # TERRARIA_MOTD: "Welcome!"
      # TERRARIA_PASSWORD: "changeme"
      # TERRARIA_SECURE: "1"
      # TERRARIA_NOUPNP: "1"
    volumes:
      - terraria_data:/data

volumes:
  terraria_data:
```

Start it:

```bash
docker compose up -d
```

## Build

The image downloads the dedicated server ZIP from Terraria’s official endpoint:

- `https://terraria.org/api/download/pc-dedicated-server/terraria-server-${VERSION}.zip`

Build from the `vanilla/` folder:

```bash
docker build -t brammys/terraria --build-arg VERSION=<VERSION> .
```

Notes:
- `<VERSION>` must match the version identifier used by the Terraria download endpoint.
- The entrypoint chooses `./TerrariaServer` on `amd64` and `mono ./TerrariaServer.exe` on other architectures.
