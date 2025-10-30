# 🧩 Minecraft Docker Images (LAN-slide)

This repository provides versioned Docker images for running **Spigot-based Minecraft servers**, maintained by [LAN-slide](https://github.com/lanslide-team).  
Each Minecraft version is kept in its own branch (for example, `v1.21.9`, `v1.22.0`) as a **known-good build**, pinned to matching Spigot and Java versions.

---

## 🏗️ Repository Structure

```text
minecraft-docker/
├── base/    → Base Spigot server (no plugins)
├── build/   → Variant image with plotworld
├── map/     → Variant image with preloaded world/map
└── .github/ → CI workflow (auto-builds to GHCR)
```

- **`base/`** – compiles Spigot for the target Minecraft version using [BuildTools](https://www.spigotmc.org/wiki/buildtools/).  
- **`build/`** – extends the base image with plugins or server packs (e.g. EssentialsX, LuckPerms).  
- **`map/`** – extends the base image with a pre-generated world or event map.  
- **`.github/workflows/publish.yml`** – automatically builds and pushes all three images to [GHCR](https://ghcr.io) when a `v*` branch is pushed.

---

## 🧬 Version-per-Branch Workflow

Each branch corresponds to a specific Minecraft version and Java runtime.  
Pinned versions are set at the top of `base/Dockerfile`:

```Dockerfile
ARG MC_VERSION=1.21.9
ARG JAVA_MAJOR=21
```

To create a new version branch:
```bash
git fetch origin
git checkout v1.21.9
git checkout -b v1.22.0
sed -i 's/MC_VERSION=1\.21\.8/MC_VERSION=1.22.0/' base/Dockerfile
git commit -am "Bump to Minecraft 1.22.0"
git push --set-upstream origin v1.22.0
```

The CI workflow automatically builds and publishes.
Therefore, you can pull directly from github:

```bash
docker pull ghcr.io/lanslide-team/minecraft-base:v1.21.9
docker pull ghcr.io/lanslide-team/minecraft-build:v1.21.9
docker pull ghcr.io/lanslide-team/minecraft-map:v1.21.9
```

🚀 Building Locally
Build the base image

```bash
docker build -t ghcr.io/lanslide-team/minecraft-base:v1.21.9 ./base
```

Build plugin or map variants

```bash
docker build -t ghcr.io/lanslide-team/minecraft-build:v1.21.9 ./build
docker build -t ghcr.io/lanslide-team/minecraft-map:v1.21.9 ./map
```

Run the server
```bash
docker run -d \
  --name spigot_1218 \
  --network mc-macvlan --ip 192.168.1.50 \
  -e MOTD="LAN-slide Minecraft 1.21.9" \
  -v /srv/minecraft/1.21.9:/mc \
  ghcr.io/lanslide-team/minecraft-base:v1.21.9
```  
  
💡 Using macvlan gives your container its own LAN IP, so you don’t need to expose ports (-p).

⚙️ Environment Variables
Common configuration options (all optional):

| Variable      | Default                         | Description                          |
|---------------|---------------------------------|--------------------------------------|
| EULA          | FALSE                           | Must be TRUE to start the server     |
| MOTD          | Welcome to the Minecraft Server | Server message of the day            |
| MAX_PLAYERS   | 100                             | Maximum number of players            |
| GAMEMODE      | creative                        | Default game mode                    |
| DIFFICULTY    | 1                               | World difficulty (0–3)               |
| ENABLE_RCON   | true                            | Enable RCON                          |
| RCON_PASSWORD | DEFAULT_ADMIN_PASSWORD          | RCON password                        |
| MEMORY        | 6G                              | JVM heap size for Xms/Xmx            |
| ONLINE_MODE   | true                            | Enforce online player authentication |
| LEVEL_NAME    | world                           | World folder name                    |

See base/entrypoint.sh for the full list.

🧰 CI/CD Integration

Every push to a version branch (v*) triggers:

Build of base, build, and map images

Push to GitHub Container Registry (GHCR)

Automatic tagging under:

```bash
ghcr.io/lanslide-team/minecraft-base:vX.Y.Z
ghcr.io/lanslide-team/minecraft-build:vX.Y.Z
ghcr.io/lanslide-team/minecraft-map:vX.Y.Z
```

🧩 Related Projects
LAN-slide – community BYOC LAN & gaming events

The Gamers Retreat – multi-day gaming retreat with BYOD servers

MatchZy – CS2/CS:GO match management plugin

📜 License
MIT © LAN-slide Team
