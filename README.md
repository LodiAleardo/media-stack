[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/linuxshots)

# media-stack

A self-hosted media ecosystem that combines media management, streaming, AI-powered recommendations, and VPN.

This stack includes:

- **VPN:** For secure and private media downloading
- **Radarr:** For movie management
- **Sonarr:** For TV show management
- **Prowlarr:** A torrent indexer manager for Radarr/Sonarr
- **qBittorrent:** Torrent client for downloading media
- **Jellyseerr:** To manage media requests
- **Jellyfin:** Open-source media streamer
- **Recommendarr:** For AI-powered movie and show recommendations

## Requirements

- Docker version 28.0.1 or later
- Docker compose version v2.33.1 or later
- Older versions may work, but they have not been tested.

## Deploy the Stack Without VPN  

🚨 **Warning:** Deploying without a VPN is **highly discouraged** as it may expose your IP address when torrenting media.  

To proceed without VPN, run the following command:  

```bash
docker compose --profile no-vpn up -d

# OPTIONAL: Use Nginx as a reverse proxy
# docker compose -f docker-compose-nginx.yml up -d
```

## Configure qBittorrent

- Open qBitTorrent at http://localhost:5080. Default username is `admin`. Temporary password can be collected from container log `docker logs qbittorrent`
- Go to Tools --> Options --> WebUI --> Change password
- Run below commands on the server

```bash
docker exec -it qbittorrent bash # Get inside qBittorrent container

# Above command will get you inside qBittorrent interactive terminal, Run below command in qbt terminal
mkdir /downloads/movies /downloads/tvshows
chown 1000:1000 /downloads/movies /downloads/tvshows
```

## Configure Radarr

- Open Radarr at http://localhost:7878
- Settings --> Media Management --> Check mark "Movies deleted from disk are automatically unmonitored in Radarr" under File management section --> Save
- Settings --> Media Management --> Scroll to bottom --> Add Root Folder --> Browse to /downloads/movies --> OK
- Settings --> Download clients --> qBittorrent --> Add Host (qbittorrent) and port (5080) --> Username and password --> Test --> Save **Note: If VPN is enabled, then qbittorrent is reachable on vpn's service name. In this case use `vpn` in Host field.**
- Settings --> General --> Enable advance setting --> Select Authentication and add username and password
- Indexer will get automatically added during configuration of Prowlarr. See 'Configure Prowlarr' section.

Sonarr can also be configured in similar way.

**Add a movie** (After Prowlarr is configured)

- Movies --> Search for a movie --> Add Root folder (/downloads/movies) --> Quality profile --> Add movie
- All queued movies download can be checked here, Activities --> Queue 
- Go to qBittorrent (http://localhost:5080) and see if movie is getting downloaded (After movie is queued. This depends on availability of movie in indexers configured in Prowlarr.)

## Configure Jellyfin

- Open Jellyfin at http://localhost:8096
- When you access the jellyfin for first time using browser, A guided configuration will guide you to configure jellyfin. Just follow the guide.
- Add media library folder and choose /data/movies/

## Configure Jellyseerr

- Open Jellyfin at http://localhost:5055
- When you access the jellyseerr for first time using browser, A guided configuration will guide you to configure jellyseerr. Just follow the guide and provide the required details about sonarr and Radarr.
- Follow the Overseerr document (Jellyseerr is fork of overseerr) for detailed setup - https://docs.overseerr.dev/ 

## Configure Prowlarr

- Open Prowlarr at http://localhost:9696
- Settings --> General --> Authentications --> Select Authentication and add username and password
- Add Indexers, Indexers --> Add Indexer --> Search for indexer --> Choose base URL --> Test and Save
- Add application, Settings --> Apps --> Add application --> Choose Radarr --> Prowlarr server (http://prowlarr:9696) --> Radarr server (http://radarr:7878) --> API Key --> Test and Save
- Add application, Settings --> Apps --> Add application --> Choose Sonarr --> Prowlarr server (http://prowlarr:9696) --> Sonarr server (http://sonarr:8989) --> API Key --> Test and Save
- This will add indexers in respective apps automatically.

**Note: If VPN is enabled, then Prowlarr will not be able to reach radarr and sonarr with localhost or container service name. In that case use static IP for sonarr and radarr in radarr/sonarr server field (for e.g. http://172.19.0.5:8989). Prowlar will also be not reachable with its container/service name. Use `http://vpn:9696` instead in prowlar server field.**

## Disclaimer  

> Neither the author nor the developers of the code in this repository **condone or encourage** downloading, sharing, seeding, or peering of **copyrighted material**.  
> Such activities are **illegal** under international laws.  
>
> This project is intended for **educational purposes only**.  
