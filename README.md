## Comprehensive Guide to Setting Up Plex, Overseerr, Radarr, Sonarr, Lidarr, Readarr, Prowlarr, SABnzbd, Newshosting, NZBGeek, Tautulli, qBittorrent, and Gluetun Using Docker

### Overview of Each Service

1. **Plex:** A media server that organizes and streams your personal collection of movies, TV shows, music, photos, and more.
2. **Overseerr:** A content request management system that integrates with Plex to allow users to request new media.
3. **Radarr:** An automated movie downloader that searches for and downloads movies based on your preferences.
4. **Sonarr:** An automated TV show downloader that searches for and downloads TV episodes based on your preferences.
5. **Lidarr:** An automated music downloader that searches for and downloads music based on your preferences.
6. **Readarr:** An automated ebook and audiobook downloader that searches for and downloads books based on your preferences.
7. **Prowlarr:** An indexer manager for Radarr, Sonarr, Lidarr, and Readarr.
8. **SABnzbd:** A Usenet download client that handles the downloading of files from Usenet.
9. **Newshosting:** A Usenet service provider that gives you access to Usenet servers.
10. **NZBGeek:** A Usenet indexing service that provides NZB files for downloading.
11. **qBittorrent:** A torrent downloader that handles the downloading of files from torrent sources.
12. **Tautulli:** A monitoring and tracking tool for Plex usage.
13. **Gluetun:** A VPN client container that routes traffic through various VPN providers.

### How It All Works

### Explanation of the Flow

Imagine you have a personal assistant who helps you find, download, organize, and watch your favorite movies, TV shows, music, and books. Here's how it all works together: 

**Plex** is like your media library where everything you own is neatly organized and ready to watch. **Overseerr** is the assistant who takes requests for new content. If you want a new movie, **Radarr** searches for it online and downloads it using **qBittorrent** (for torrents) or **SABnzbd** (for Usenet). Similarly, if you want a new TV show, **Sonarr** does the searching and downloading. For music and books, **Lidarr** and **Readarr** do the same job, respectively. **Prowlarr** helps all these assistants by managing the sources they search from. **Tautulli** keeps track of what you watch and how you use Plex, providing useful statistics. All your internet traffic is kept private and secure by **Gluetun**, which routes everything through a VPN. This way, you get your content automatically, privately, and organized neatly for easy access.

### Diagram

Here’s a representation of how these services interact:

1. **User Requests Content**: Through Overseerr
2. **Content Search & Download**: Managed by Radarr, Sonarr, Lidarr, and Readarr, using Prowlarr to find sources
3. **Download Tools**: Use qBittorrent (for torrents) and SABnzbd (for Usenet)
4. **VPN Protection**: All download traffic is routed through Gluetun for privacy
5. **Content Organization & Access**: Plex organizes and streams the downloaded content, monitored by Tautulli


### Prerequisites

1. **A server or computer to run these services** (Linux is preferred, but you can use Windows or macOS as well).
2. **Basic knowledge of the command line**.
3. **Docker and Docker Compose installed** on your server.

### Step 1: Install Docker and Docker Compose

#### Install Docker

1. Open a terminal on your server.
2. Run the following command to install Docker:
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   ```

#### Install Docker Compose

1. Run the following command to install Docker Compose:
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -Po '(?<=tag_name": ")[^"]*')" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

### Step 2: Create a Directory for Your Docker Setup

1. Create a directory for your Docker setup and navigate into it:
   ```bash
   mkdir media-server
   cd media-server
   ```

### Step 3: Create a Docker Compose File

1. Create a `docker-compose.yml` file:
   ```bash
   nano docker-compose.yml
   ```

2. Add the following configuration to your `docker-compose.yml` file:

   ```yaml
   version: '3.8'

   services:
     plex:
       image: linuxserver/plex:latest
       container_name: plex
       network_mode: host
       environment:
         - PUID=${PUID:-1000}
         - PGID=${PGID:-1000}
         - TZ=${TZ:-UTC}
         - VERSION=docker
         - PLEX_CLAIM=${PLEX_CLAIM:-} # Optional: obtain from https://www.plex.tv/claim
       volumes:
         - ${CONFIG_DIR:-./config}/plex:/config
         - ${MEDIA_DIR:-./media}:/media
       restart: unless-stopped
       healthcheck:
         test: ["CMD", "curl", "-f", "http://localhost:32400/web/index.html"]
         interval: 30s
         timeout: 10s
         retries: 3

     overseerr:
       image: linuxserver/overseerr:latest
       container_name: overseerr
       environment:
         - PUID=${PUID:-1000}
         - PGID=${PGID:-1000}
         - TZ=${TZ:-UTC}
       volumes:
         - ${CONFIG_DIR:-./config}/overseerr:/config
       ports:
         - "5055:5055"
       restart: unless-stopped

     radarr:
       image: linuxserver/radarr:latest
       container_name: radarr
       environment:
         - PUID=${PUID:-1000}
         - PGID=${PGID:-1000}
         - TZ=${TZ:-UTC}
       volumes:
         - ${CONFIG_DIR:-./config}/radarr:/config
         - ${MEDIA_DIR:-./media}/movies:/movies
         - ${DOWNLOAD_DIR:-./downloads}:/downloads
       ports:
         - "7878:7878"
       restart: unless-stopped

     sonarr:
       image: linuxserver/sonarr:latest
       container_name: sonarr
       environment:
         - PUID=${PUID:-1000}
         - PGID=${PGID:-1000}
         - TZ=${TZ:-UTC}
       volumes:
         - ${CONFIG_DIR:-./config}/sonarr:/config
         - ${MEDIA_DIR:-./media}/tv:/tv
         - ${DOWNLOAD_DIR:-./downloads}:/downloads
       ports:
         - "8989:8989"
       restart: unless-stopped

     prowlarr:
       image: linuxserver/prowlarr:latest
       container_name: prowlarr
       environment:
         - PUID=${PUID:-1000}
         - PGID=${PGID:-1000}
         - TZ=${TZ:-UTC}
       volumes:
         - ${CONFIG_DIR:-./config}/prowlarr:/config
       ports:
         - "9696:9696"
       restart: unless-stopped

     sabnzbd:
       image: linuxserver/sabnzbd:latest
       container_name: sabnzbd
       environment:
         - PUID=${PUID:-1000}
         - PGID=${PGID:-1000}
         - TZ=${TZ:-UTC}
       volumes:
         - ${CONFIG_DIR:-./config}/sabnzbd:/config
         - ${DOWNLOAD_DIR:-./downloads}:/downloads
         - ${INCOMPLETE_DIR:-./incomplete}:/incomplete-downloads
       ports:
         - "8080:8080"
       restart: unless-stopped

     tautulli:
       image: linuxserver/tautulli:latest
       container_name: tautulli
       environment:
         - PUID=${PUID:-1000}
         - PGID=${PGID:-1000}
         - TZ=${TZ:-UTC}
       volumes:
         - ${CONFIG_DIR:-./config}/tautulli:/config
       ports:
         - "8181:8181"
       restart: unless-stopped

     qbittorrent:
       image: linuxserver/qbittorrent:latest
       container_name: qbittorrent
       environment:
         - PUID=${PUID:-1000}
         - PGID=${PGID:-1000}
         - TZ=${TZ:-UTC}
         - WEBUI_PORT=8080
       volumes:
         - ${CONFIG_DIR:-./config}/qbittorrent:/config
         - ${DOWNLOAD_DIR:-./downloads}:/downloads
       ports:
         - "8080:8080"
       restart: unless-stopped
       depends_on:
         - gluetun

     gluetun:
       image: qmcgaw/gluetun:latest
       container_name: gluetun
       cap_add:
         - NET_ADMIN
       environment:
         - VPNSP=private internet access
         - OPENVPN_USER=<your_pia_username>
         - OPENVPN_PASSWORD=<your_pia_password>
         - TZ=${TZ:-UTC}
       volumes:
         - ${CONFIG_DIR:-./config}/gluetun:/gluetun
       ports:
         - "9091:9091"
         - "8888:8888"
       restart: unless-stopped

   networks:
     default:
       name: media_network

   volumes:
     config:
     media:
     downloads:
     incomplete:
   ```

### Step 4: Adjust Permissions

1. Make sure your user has the correct permissions to access the configuration and media directories. Adjust `PUID` and `PGID` if necessary. To find your user and group ID, run:
   ```bash
   id
   ```

### Step 5: Start the Services

1. Run the following command to start all services:
   ```bash
   docker-compose up -d
   ```

### Step 6: Access the Services

You can access the services in your web browser using your server’s IP address followed by the respective port number. For example:

- **Plex:** http://your-server-ip:32400
- **Overseerr:** http://your-server-ip:5055
- **Radarr:** http://your-server-ip:7878
- **Sonarr:** http://your-server-ip:8989
- **Lidarr:** http://your-server-ip:8686
- **Readarr:** http://your-server-ip:8787
- **Prowlarr:** http://your-server-ip:9696
- **SABnzbd:** http://your-server-ip:8080
- **Tautulli:** http://your-server-ip:8181
- **qBittorrent:** http://your-server-ip:8080


### Step 7: Configure Each Application

#### Plex
1. **Initial Setup:**
   - Open Plex (http://your-server-ip:32400).
   - Follow the on-screen instructions to create an account and set up your server.
2. **Library Setup:**
   - Add libraries for Movies, TV Shows, Music, etc., and point them to the respective directories.

#### Overseerr
1. **Initial Setup:**
   - Open Overseerr (http://your-server-ip:5055).
   - Follow the on-screen instructions to create an account and set up Overseerr.
2. **Connect to Plex:**
   - In the settings, connect Overseerr to your Plex server.

#### Radarr
1. **Initial Setup:**
   - Open Radarr (http://your-server-ip:7878).
   - Follow the on-screen instructions to set up Radarr.
2. **Add Indexers:**
   - Go to Settings > Indexers and add your Usenet indexers (e.g., NZBGeek).
3. **Add Download Client:**
   - Go to Settings > Download Clients and add SABnzbd or qBittorrent.
4. **Add Media Management:**
   - Go to Settings > Media Management and configure the paths to your movie directories.

#### Sonarr
1. **Initial Setup:**
   - Open Sonarr (http://your-server-ip:8989).
   - Follow the on-screen instructions to set up Sonarr.
2. **Add Indexers:**
   - Go to Settings > Indexers and add your Usenet indexers (e.g., NZBGeek).
3. **Add Download Client:**
   - Go to Settings > Download Clients and add SABnzbd or qBittorrent.
4. **Add Media Management:**
   - Go to Settings > Media Management and configure the paths to your TV show directories.

#### Lidarr
1. **Initial Setup:**
   - Open Lidarr (http://your-server-ip:8686).
   - Follow the on-screen instructions to set up Lidarr.
2. **Add Indexers:**
   - Go to Settings > Indexers and add your Usenet indexers (e.g., NZBGeek).
3. **Add Download Client:**
   - Go to Settings > Download Clients and add SABnzbd or qBittorrent.
4. **Add Media Management:**
   - Go to Settings > Media Management and configure the paths to your music directories.

#### Readarr
1. **Initial Setup:**
   - Open Readarr (http://your-server-ip:8787).
   - Follow the on-screen instructions to set up Readarr.
2. **Add Indexers:**
   - Go to Settings > Indexers and add your Usenet indexers (e.g., NZBGeek).
3. **Add Download Client:**
   - Go to Settings > Download Clients and add SABnzbd or qBittorrent.
4. **Add Media Management:**
   - Go to Settings > Media Management and configure the paths to your book directories.

#### Prowlarr
1. **Initial Setup:**
   - Open Prowlarr (http://your-server-ip:9696).
   - Follow the on-screen instructions to set up Prowlarr.
2. **Add Indexers:**
   - Go to Settings > Indexers and add your Usenet indexers.
3. **Connect to Other Apps:**
   - Integrate Prowlarr with Radarr, Sonarr, Readarr, and Lidarr in the Settings.

#### SABnzbd
1. **Initial Setup:**
   - Open SABnzbd (http://your-server-ip:8080).
   - Follow the on-screen instructions to set up SABnzbd.
2. **Configure Usenet Provider:**
   - Go to Config > Servers and add your Usenet provider (Newshosting) details.
3. **Configure Categories:**
   - Go to Config > Categories and set up categories for movies, TV shows, music, etc.

#### NZBHydra2
1. **Initial Setup:**
   - Open NZBHydra2 (http://your-server-ip:5076).
   - Follow the on-screen instructions to set up NZBHydra2.
2. **Add Indexers:**
   - Go to Config > Indexers and add your Usenet indexers.

#### NZBGet
1. **Initial Setup:**
   - Open NZBGet (http://your-server-ip:6789).
   - Follow the on-screen instructions to set up NZBGet.
2. **Configure Usenet Provider:**
   - Go to Settings > News-Servers and add your Usenet provider (Newshosting) details.
3. **Configure Categories:**
   - Go to Settings > Categories and set up categories for movies, TV shows, music, etc.

#### qBittorrent
1. **Initial Setup:**
   - Open qBittorrent (http://your-server-ip:8080).
   - Follow the on-screen instructions to set up qBittorrent.
2. **Add Download Folders:**
   - Go to Settings > Downloads and set the download paths to your preferred directories.

#### Tautulli
1. **Initial Setup:**
   - Open Tautulli (http://your-server-ip:8181).
   - Follow the on-screen instructions to set up Tautulli.
2. **Connect to Plex:**
   - In the settings, connect Tautulli to your Plex server to monitor activity.

#### Gluetun (VPN)
1. **Initial Setup:**
   - Configure your VPN credentials in the Docker Compose file (`OPENVPN_USER` and `OPENVPN_PASSWORD`).
2. **Ensure VPN is Working:**
   - Check that your IP address is masked by the VPN by visiting an IP-checking website from within the qBittorrent web interface.

### Summary Table

| **Category**               | **Name**      | **Description**                                                                                     | **Why It's Useful**                                                                                                      | **Website**                  | **IP/Port**             |
|----------------------------|---------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|------------------------------|-------------------------|
| **Media Server**           | Plex          | A media server that organizes and streams your personal collection of movies, TV shows, music, photos, and more. | Provides a centralized platform to access and stream all your media from any device, anywhere.                           | [Plex](https://www.plex.tv)  | http://your-server-ip:32400 |
| **Content Request Manager**| Overseerr     | A content request management system that integrates with Plex to allow users to request new media.  | Enables users to request new movies and TV shows easily, streamlining content addition.                                  | [Overseerr](https://overseerr.dev) | http://your-server-ip:5055  |
| **Movie Downloader**       | Radarr        | An automated movie downloader that searches for and downloads movies based on your preferences.     | Automates the process of finding and downloading movies, ensuring your collection is always up-to-date.                  | [Radarr](https://radarr.video) | http://your-server-ip:7878  |
| **TV Show Downloader**     | Sonarr        | An automated TV show downloader that searches for and downloads TV episodes based on your preferences. | Automates the process of finding and downloading TV shows, ensuring you never miss an episode.                           | [Sonarr](https://sonarr.tv)   | http://your-server-ip:8989  |
| **Music Downloader**       | Lidarr        | An automated music downloader that searches for and downloads music based on your preferences.      | Keeps your music collection updated by automatically downloading new releases.                                           | [Lidarr](https://lidarr.audio) | http://your-server-ip:8686  |
| **Book Downloader**        | Readarr       | An automated ebook and audiobook downloader that searches for and downloads books based on your preferences. | Ensures your book collection is always growing with new titles by automating the download process.                        | [Readarr](https://readarr.com) | http://your-server-ip:8787  |
| **Indexer Manager**        | Prowlarr      | An indexer manager for Radarr, Sonarr, Lidarr, and Readarr.                                         | Centralizes the management of indexers, simplifying the configuration process for other automation tools.                 | [Prowlarr](https://prowlarr.com) | http://your-server-ip:9696  |
| **Download Client**        | SABnzbd       | A Usenet download client that handles the downloading of files from Usenet.                         | Facilitates the downloading of binary files from Usenet, integrating with automation tools like Radarr and Sonarr.        | [SABnzbd](https://sabnzbd.org) | http://your-server-ip:8080  |
| **Usenet Provider**        | Newshosting   | A Usenet service provider that gives you access to Usenet servers.                                  | Provides the necessary access to Usenet servers for downloading content.                                                  | [Newshosting](https://www.newshosting.com) | N/A                       |
| **Indexer Service**        | NZBGeek       | A Usenet indexing service that provides NZB files for downloading.                                  | Indexes Usenet content and provides NZB files, making it easier to find and download content.                             | [NZBGeek](https://www.nzbgeek.info) | N/A                       |
| **Torrent Downloader**     | qBittorrent   | A torrent downloader that handles the downloading of files from torrent sources.                    | Facilitates the downloading of torrent files, integrating with automation tools like Radarr and Sonarr.                   | [qBittorrent](https://www

.qbittorrent.org) | http://your-server-ip:8080  |
| **Plex Activity Monitor**  | Tautulli      | A monitoring and tracking tool for Plex usage.                                                      | Provides detailed insights into Plex server usage, helping manage and optimize media streaming.                          | [Tautulli](https://tautulli.com) | http://your-server-ip:8181  |
| **VPN**                    | Gluetun       | A VPN client container that routes traffic through various VPN providers.                           | Ensures privacy and security by encrypting traffic and hiding your IP address.                                           | [Gluetun](https://github.com/qdm12/gluetun) | Managed through Docker |

### Conclusion

By following these steps, you will have set up a comprehensive media server and automation system using Plex, Overseerr, Radarr, Sonarr, Lidarr, Readarr, Prowlarr, SABnzbd, Newshosting, NZBGeek, qBittorrent, and Gluetun. Each application has its specific role in managing and automating the downloading, organizing, and streaming of your media content, all while ensuring your privacy and security with a VPN.
