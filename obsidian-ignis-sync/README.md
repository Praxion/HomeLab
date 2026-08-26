# Self-Hosted Obsidian Sync & AI-Organized Second Brain

This repository documents my homelab configuration for hosting, accessing, and synchronizing my Obsidian vault. By combining **Ignis** for clientless browser access, **Syncthing** for continuous peer-to-peer data replication, and a dedicated **Home PC** as my primary database and local LLM processing node, I have built a private, automated "Second Brain."

---

## System Architecture

                  ┌──────────────────────┐
                  │    Mobile / Web      │
                  │  (Remote Capture)    │
                  └──────────┬───────────┘
                             │ HTTPS (via Reverse Proxy)
                             ▼
                 ┌───────────────────────┐
                 │    Home Lab Server    │
                 │  • Ignis (Web App)    │
                 │  • Syncthing (Peer)   │
                 └──────────┬────────────┘
                            │
                            │ Syncthing Protocol
                            │ (Holds edits queue if Home PC is offline)
                            ▼
                 ┌───────────────────────┐
                 │    Home Workstation   │
                 │  • Syncthing (Peer)   │
                 │  • Vault Storage      │
                 │  • Local LLM Agent    │
                 │    (Auto-sorting)     │
                 └───────────────────────┘

---

## Project Blueprint & FAQ

### 1. How am I going to host Obsidian notes on my server?
I host my Obsidian environment on my self-hosted home lab server using **Ignis** [1]. Ignis acts as a compatibility shim that translates Obsidian's desktop Electron and Node APIs into browser-compatible instructions [1]. This allows me to run a lightweight Docker container that serves the official Obsidian application directly to my web browser [1-3]. 

To keep the notes in sync, I run **Syncthing** on the server alongside Ignis [4]. The server functions as an active Syncthing client, mapping the vault folder used by Ignis [4]. This folder synchronizes with my **Home PC**, which acts as the main storage pool and workstation [4]. Keeping the physical files on my Home PC allows me to run local Large Language Models (LLMs) and automated scripting engines directly over the raw Markdown files to handle sorting, semantic search, and metadata tagging.

---

### 2. What am I looking for by hosting Obsidian on my server?
My primary goal is to have an incredibly simple, zero-footprint way to access my notes from any device with a standard web browser (such as locked-down corporate or guest computers) [5, 6]. 

If my network connection is severed due to lack of Wi-Fi or cellular service, my workflow is unaffected. Once reconnected, I can utilize Ignis's built-in file upload system (which supports drag-and-drop, ribbon commands, and right-click menus) to import new materials directly into my vault [7]. Syncthing then ensures that all newly added or modified data seamlessly replicates back to my Home PC workstation [4, 8].

#### Technical Notes on Offline Sync, Delays, and Conflicts:

*   **How does the system handle sync if the Home PC is turned off?**
    Syncthing is naturally stateful and decentralized. If your Home PC is powered down, Syncthing on the server will automatically detect that the peer is offline. Any changes you make inside the Ignis web browser will write directly to the server's disk and be placed into Syncthing's local transfer queue. The files are held safely in their updated state on the server.
*   **Will the files on the server be deleted or overwritten when syncing resumes?**
    No. When your Home PC is turned back on, Syncthing will reconnect, compare the file modification timestamps, and replicate the changes from the server to your PC. Because Syncthing tracks changes using cryptographic file hashes and metadata database states, it knows exactly which machine made the newest edit.
*   **Should uploaded files be named differently to prevent overwriting the Home PC?**
    You do not need to manually change file names. If a file is edited *only* on the server while the Home PC is offline, it will simply update the file on your Home PC upon reconnection. If the *same* file was edited on both the server and the Home PC while they were disconnected, Syncthing's built-in conflict resolution engine will automatically protect your data. Instead of overwriting or deleting anything, it saves both versions: it updates the main file and renames the conflicting file by appending a `.sync-conflict-[date]-[time]` suffix. You can then review and merge the changes safely at your convenience.

---

### 3. How will setting this up help me in my life?
Deploying this infrastructure bridges the gap between structured research and spontaneous ideation. It allows me to:
*   **Capture thoughts instantly**: Quickly jot down ideas or upload documents from any external device [7], knowing they are immediately secured on my personal hardware [1].
*   **Automate background workflows**: Eliminate manual syncing, Git commits, or USB transfers. Every browser edit, organization change, or attachment uploaded through the web interface propagates to my home machine within seconds of reconnection [9, 10].
*   **Centralize project tracking**: Maintain a reliable, always-accessible single source of truth for planning, logging, and personal knowledge management [11].

---

### 4. Will incorporating an AI help in the overall organization of my "Second Brain"?
Yes. Operating a local-first file structure on a dedicated Home PC makes deep AI integration highly effective. By pointing an AI indexing daemon or agent at the local synced directory, the system can:
*   **Enforce clean taxonomies**: Automatically parse file frontmatter, suggest/apply standardized tag structures, and clean up naming inconsistencies.
*   **Connect the dots**: Identify hidden semantic connections between notes across different folders, establishing automatic wiki-style links and updating Obsidian's graph view.
*   **Serve as a personal oracle**: Run locally hosted Retrieval-Augmented Generation (RAG) models over the raw text of my notes to answer questions, summarize long research articles, and surface relevant insights exactly when I need them.

---

## Quick Start Configuration

To deploy the web interface on your home lab server, save the following configuration to `docker-compose.yml` [12]:

```yaml
services:
  ignis:
    image: nobbe/ignis:latest
    ports:
      - "8080:8080"
    environment:
      - PUID=1000 # Run as your host system user
      - PGID=1000
      - WRITE_COALESCE_MS=500 # Debounces rapid writes on network drives (SMB/NFS)
    volumes:
      - ./vaults:/vaults
      - ./data:/app/data
      - obsidian-app:/app/obsidian-app
    restart: unless-stopped
Note: Because Ignis does not feature built-in authentication, you must place an external authentication gateway (like Tailscale, Authelia, or Tinyauth) in front of this container before exposing it to the wider internet
.

***

🔍 Would you like me to draft a companion **Docker Compose** file showing how to run Syncthing as a sidecar container alongside Ignis on your server to complete your GitHub repository configuration?
