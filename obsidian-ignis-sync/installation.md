Here is how you can set this up and what you need to keep in mind:

1. Enable LXC Nesting (Crucial Proxmox Step)
Because Ignis is distributed and run as a Docker container, you will need to run Docker inside your LXC.
If you are using Proxmox, you must go to your LXC's Settings -> Resources/Options, edit the Features flag, and check Nesting. Without nesting enabled, Docker will fail to start inside the LXC.

2. The Shared Folder Strategy
Both applications must point to the exact same directory on your LXC's filesystem (for example, /opt/obsidian/vault).
Ignis will read this folder to display and edit your notes in the browser.
Syncthing will watch this exact same folder and sync any changes to your Home PC.

3. Unified Docker Compose for your LXC
Once Docker is installed inside your LXC, you can deploy both services using a single docker-compose.yml file to manage them together:
version: '3.8'

```
services:
  ignis:
    image: nobbe/ignis:latest
    container_name: ignis-web
    ports:
      - "8080:8080"
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /opt/obsidian/vault:/vaults # Pointing to the shared local vault
      - ./ignis-data:/app/data
      - obsidian-app:/app/obsidian-app
    restart: unless-stopped

  syncthing:
    image: syncthing/syncthing:latest
    container_name: syncthing-agent
    ports:
      - "8384:8384"       # Web GUI port
      - "22000:22000/tcp" # Sync protocol port
      - "22000:22000/udp"
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /opt/obsidian/vault:/var/syncthing/ObsidianVault # Same shared local vault
      - ./syncthing-config:/var/syncthing/config
    restart: unless-stopped

volumes:
  obsidian-app:
```

⚠️ A Final Operational Warning
As a reminder, you should not enable the LiveSync community plugin inside the Ignis browser UI itself. Because LiveSync stores its synchronization database in the browser's local storage, opening Ignis in a new tab will make it look like a new device to the sync engine, which can trigger massive re-indexing and cause data corruption.
By hosting them together in your LXC, you let Ignis act as your pure web editor 
and let Syncthing silently handle 100% of the syncing back to your Home PC in the background.
