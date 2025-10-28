---
layout: post
title: "Self-Hosted Media Server with Docker on Ubuntu (Jellyfin, Sonarr, Radarr)" 
categories: self-hosted
date : 2025-01-18 09:14:50
---

## Disclaimer
Before proceeding, please note that this article is for educational purposes only. It aims to provide information on setting up a self-hosted streaming server, but it is your responsibility to ensure compliance with all local, national, and international laws. I assume no liability for any misuse of this information.

---

This guide documents the process of setting up a self-hosted streaming server. The services covered include Jellyfin, Sonarr, Radarr, Lidarr, qBittorrent, Jackett, Flaresolverr, Ombi, and Heimdall, all installed on Docker containers in Ubuntu. Here’s a brief overview of these services:

- **Jellyfin**: A media server for watching your downloaded content.
- **Jackett**: Provides torrent indexers.
- **Sonarr**: Automates TV show downloads.
- **Radarr**: Automates movie downloads.
- **Lidarr**: Automates music downloads.
- **qBittorrent**: A torrent downloader.
- **Ombi**: Handles user requests for media.
- **Heimdall**: A dashboard for easy access to all services.
- **Flaresolverr**: Solves captchas for certain indexers.

---

## Requirements

**Hosting**: You’ll need a machine to host this setup. Options include an AWS EC2 instance, Raspberry Pi, mini PC, or your laptop.

**Command-Line knowledge**: Familiarity with Linux command-line interface is essential.

**Operating System**: This guide uses Ubuntu, so a basic understanding of Linux is recommended.


----

## Getting Started

### Step 1: Create the Directory Structure

Organise your server with the following folder structure:

```
mkdir ~/server  
mkdir ~/server/media ~/server/media/movies  ~/server/media/music  ~/server/media/tv
mkdir ~/server/torrents ~/server/torrents/incomplete  ~/server/torrents/movies  ~/server/torrents/music  ~/server/torrents/tv 
mkdir ~/server/compose ~/server/compose/media-server
```

The folder `media` stores the downloaded files.<br>
The folder `torrents` stores the files being downloaded.<br>
The folder `compose` stores the the services.

You should have the following directories:

```
server/
├── compose
│   └── media-server
├── media
│   ├── movies
│   ├── music
│   └── tv
└── torrents
    ├── incomplete
    ├── movies
    ├── music
    └── tv
```

Ensure Docker has the appropriate permissions to access the media folder:

```
sudo chown -R $USER:$USER /media
chmod -R 775 /media
```


### Step 2: Install Docker and Docker Compose

Update your system and install Docker:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install docker.io docker-compose -y
```

Enable and start Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### Step 3: Create a docker-compose.yml File

Use a text editor to create the docker-compose.yml file:

```
nano ~/server/compose/media-server/docker-compose.yml
```

Add the following content:

```yaml
version: "3.8"
services:
    # torrent indexer
    jackett:
        container_name: jackett
        image: linuxserver/jackett
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        volumes:
          - '/home/ubuntu/server/configs/jackett:/config'
          - '/home/ubuntu/server/torrents:/downloads'
        ports:
          - '9117:9117'
        restart: unless-stopped

    #tv shows
    sonarr:
        container_name: sonarr
        image: linuxserver/sonarr
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        ports:
          - '8989:8989'
        volumes:
          - '/home/ubuntu/server/configs/sonarr:/config'
          - '/home/ubuntu/server:/data'
        restart: unless-stopped

    # movies
    radarr:
        container_name: radarr
        image: linuxserver/radarr
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        ports:
          - '7878:7878'
        volumes:
          - '/home/ubuntu/server/configs/radarr:/config'
          - '/home/ubuntu/server:/data'
        restart: unless-stopped

    #music
    lidarr:
        container_name: lidarr
        image: ghcr.io/linuxserver/lidarr
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        volumes:
          - '/home/ubuntu/server/configs/liadarr:/config'
          - '/home/ubuntu/server:/data'
        ports:
          - '8686:8686'
        restart: unless-stopped

    jellyfin:
        container_name: jellyfin
        image: jellyfin/jellyfin:latest
        network_mode: bridge
        ports:
          - "8096:8096"
        volumes:
          - '/home/ubuntu/server/configs/jellyfin:/config'
          - '/home/ubuntu/server/media:/data/media'
        restart: unless-stopped

    # requesting movies
    ombi:
        container_name: ombi
        image: ghcr.io/linuxserver/ombi
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        volumes:
          - '/home/ubuntu/server/configs/ombi:/config'
        ports:
          - '3579:3579'
        restart: unless-stopped

    # torrent downloader
    qflood:
        container_name: qflood
        image: hotio/qflood
        ports:
          - "8080:8080"
          - "3005:3000"
        environment:
          - PUID=1000
          - PGID=1000
          - UMASK=002
          - TZ=Asia/Kolkata
          - FLOOD_AUTH=false
        volumes:
          - '/home/ubuntu/server/configs/qflood:/config'
          - '/home/ubuntu/server/torrents:/data/torrents'
        restart: unless-stopped

    # dashboard
    heimdall:
        container_name: heimdall
        image: ghcr.io/linuxserver/heimdall
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        volumes:
          - '/home/ubuntu/server/configs/heimdall:/config'
        ports:
          - 8090:80
        restart: unless-stopped

    # captcha solver
    flaresolverr:
        container_name: flaresolverr
        image: 'ghcr.io/flaresolverr/flaresolverr:latest'
        ports:
          - '8191:8191'
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Kolkata
        restart: unless-stopped
```
Note: in the paths above, `ubuntu` refers to my local user.

### Step 4: Launch the stack

Run the following command to start the containers:

```
docker-compose up -d
```

----

## Configuring your server


### Configuring Heimdall: Your Centralized Dashboard

Heimdall is a lightweight, customizable dashboard designed to act as a central hub for accessing all your services. Follow these steps to set it up:

**Access Heimdall:** 

Open your browser and navigate to `http://<your-server-ip>:8090`.

**Set a Password (Optional):**
- Navigate to the "Users" page.
- Set an admin password to restrict access if desired.
- If you enable the password, untick the option "Allow public access to front - Only enforced if a password is set" to restrict unauthorized access.

**Add Your Services:**
- Go to the Dashboard.
- Click on the "Application List" and then the "+" button to add a new application.
- For each service (e.g., qBittorrent, Sonarr, Radarr), provide the following details:
    - Name: Enter the name of the service (e.g., "qBittorrent").
    - URL: Input the service's URL (e.g., `http://<your-server-ip>:8080` for qBittorrent).
    - Icon: Heimdall will automatically fetch an icon if available. You can also upload your own.
    - Application Type: If the service is listed (e.g., Sonarr, Radarr), select the specific type to enable optional advanced integration.
    - Save: Click "Save" to finalize adding the service.

**Repeat for All Services:** 

Add all the services you want to access through Heimdall by repeating the above steps.
<br>
<br>
Heimdall will now serve as your centralised dashboard, providing quick and organised access to all your configured services.



### Configuring qBittorrent: Your Torrent Downloader

qBittorrent is a powerful torrent downloader that seamlessly integrates with Sonarr, Radarr, and Lidarr to automate torrent management. Access it through `http://<your-server-ip>:8080`.

**Initial Access**

Default Credentials:

  - Username: `admin`
  - Password: `adminadmin`

It's highly recommended to change these credentials:

- Navigate to Tools → Options → WebUI.
- Update the username and password for added security.

#### Configuration Steps

**1. Set Default Download Paths:**

- Go to the Downloads section in the settings.
- Change the Default Save Path to: `/data/torrents/`
- Enable Keep incomplete torrents in and set the path to: `/data/torrents/incomplete/`

**2. Create Categories for Media:** Categories help organize your downloads and move them to the correct folders.

- Right-click on the Categories section in the sidebar and select Add Category.
- Create the following categories:
    - TV:
        - Name: TV
        - Path: `/data/torrents/tv`
    - Movies:
        - Name: Movies
        - Path: `/data/torrents/movies`
    - Music:
        - Name: Music
        - Path: `/data/torrents/music`

Ensure the paths match the directories you created earlier to store media files.

By following these steps, qBittorrent will automatically organise and move completed downloads into the appropriate folders, streamlining your media server setup.




### Configuring Jackett: Your Torrent Indexer

Jackett acts as a bridge between your download automation tools (like Sonarr, Radarr, and Lidarr) and torrent trackers. Access it at `http://<your-server-ip>:9117`.

**Log In and Secure Jackett**

- Open Jackett in your browser.
- At the bottom of the page, set up a password to secure access to Jackett.

#### Add Indexers

**1. Find Indexers:**

- In Jackett’s interface, click Add Indexer.
- Use the search bar or browse the list to locate the torrent trackers you wish to add (e.g., public or private trackers).

**2. Add an Indexer:** Click the Add button next to the desired indexer.

**3. Configure the Indexer:**

- For Public Trackers: Minimal configuration is required.
- For Private Trackers:
    - You may need to input login credentials or API keys.
    - These can typically be found in your account settings on the tracker’s website.

**4. Test the Connection:** Once configured, test the indexer to ensure it is functioning correctly.

**5. Copy the API Key**

- Navigate to the API Key section in Jackett (visible on the main page).
- Copy the API key for integration with Sonarr, Radarr, and Lidarr.

Jackett is now ready to provide torrent search results to your automation tools, enhancing your media server's functionality!




### Sonarr (TV Shows)

Sonarr automates the download, organization, and management of TV shows. Access it at `http://<your-server-ip>:8989`.

**Configure Download Clients**

- Navigate to Settings → Download Clients.
- Add qBittorrent:
    - Host: Enter your server's IP address (e.g., YOUR_SERVER_IP).
    - Port: Set to 8080.
    - Username and Password: Enter the credentials you configured for qBittorrent.
    - Category: Type `TV` (or the category name you set in qBittorrent for TV shows).
    - Test the connection to ensure it's working, then click Save.

**Add Indexers**

- Navigate to Settings → Indexers.
- For each indexer you added in Jackett:
    - Click Add and choose Torznab.
    - Enter the Torznab feed URL for the indexer from Jackett.
    - Paste the API Key from Jackett.
    - Select the relevant categories (e.g., TV categories).
    - Test the connection and click Save.

**Set the Root Folder**

- Go to Settings → Media Management → Root Folders.
- Add `/data/media/tv` as the root folder for TV shows.


### Repeat for Radarr (Movies) and Lidarr (Music)

- Radarr (Movies):
    - Access it at `http://<your-server-ip>:7878`.
    - Use /data/media/movies as the root folder.
    - Set the qBittorrent category to Movies.

- Lidarr (Music):
    - Access it at `http://<your-server-ip>:8686`.
    - Use /data/media/music as the root folder.
    - Set the qBittorrent category to Music.

Follow similar steps for configuring download clients, adding indexers, and defining root folders for each service. This ensures all media types are downloaded, organized, and stored in the correct directories.


### Ombi

Access it at `http://<your-server-ip>:3579`.

Ombi is a user-friendly platform for managing media requests. Setting it up is straightforward:

- Open the URL in your browser.
- Follow the on-screen instructions to complete the initial setup:
    - Configure your media server (e.g., Jellyfin).
    - Set up user authentication and permissions if needed.
    - Enable notifications for updates on requests.

That's it! Ombi is now ready to handle requests for your media library.

### Jellyfin (Viewing Platform)

Access it at `http://<your-server-ip>:8096`.

Jellyfin is your media playback hub. Setting it up is simple:

- Open the URL in your browser.
- Follow the guided tutorial to configure Jellyfin:
    - Set up your library folders (e.g., `/data/media/tv`, `/data/media/movies`, `/data/media/music`).
    - Choose metadata options and language preferences.
    - Create user accounts if you have multiple users.

Once the setup is complete, your media server will be ready for streaming. Enjoy your personalized media experience!

-r via Docker and integrate it with Sonarr and Radarr to automatically download subtitles for your media.

--- 

## Final Testing

After setting up all services and applying these final touches, verify the entire pipeline:

- Request Media
    - Use Ombi to request a movie, TV show, or music.

- Download and Storage
    - Confirm qBittorrent downloads the media to the correct folder.
    - Check if Sonarr, Radarr, or Lidarr moves the files to their respective library directories.

- Media Availability
    - Ensure the media appears in Jellyfin with the correct metadata and subtitles.

With everything in place, you can now enjoy your secure and private self-hosted streaming server! 🎉

---

## Additional Touches

These are not yet covered in this guide.

**Set Up SSL Certificates for Secure Access**
- Use tools like Let's Encrypt to obtain free SSL certificates.
- Configure your reverse proxy (e.g., Nginx or Traefik) to enable HTTPS for all your services.

**Bind Services to a VPN for Privacy**
- Ensure your torrent traffic is routed through a VPN to protect your privacy.
- Use Docker containers like gluetun or OpenVPN for VPN integration with your services.

**Install Bazarr for Automated Subtitle Management**
- Deploy Bazar
