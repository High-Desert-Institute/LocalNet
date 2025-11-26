---
layout: project
title: "LocalNet"
slug: localnet
guilds:
    - lorekeepers
repo: "https://github.com/High-Desert-Institute/LocalNet"
summary: >-
    A resilient off-grid application stack that gives every outpost a local digital hub
    for knowledge, AI, and mesh-integrated services.
---

# LocalNet

**LocalNet** is the resilient, off-grid server stack for the [High Desert Institute](https://highdesertinstitute.org) outposts. It is designed to be a "set-it-and-forget-it" digital hub that empowers regional communities to run their own local tools while integrating seamlessly with the wider mesh network.

## Mission Alignment
As part of the High Desert Institute's mission to *build a foundation for the survival of humanity*, LocalNet provides the critical digital infrastructure required for communities to function independently of the traditional internet grid. It is a primary project of the **Lorekeepers' Guild**.

THe history of cyberpunk literature is a history of asking the question; if the future sucks because megacorporations are ruining the world, and the government is too weak to stop them, ho can people use technology in unexpected ways to find a way to be ok through social and environmental collapse? LocalNet is part of our answer to that question. Communities can run local tools like GitLab, Nextcloud, and local AI assistants without relying on centralized services that may be compromised or unavailable in times of crisis.

Additionally, Meshtastic-LLM allows these services to integrate with long-range, low-bandwidth mesh networks, ensuring that even in the absence of traditional internet connectivity, communities can still communicate and access communication, vital information, and tools.

## Key Features

### 1. The Digital Hearth
LocalNet serves as an applicaiton server on the local intranet at each outpost, hosting rich services on low-power hardware (e.g., Raspberry Pi, mini PCs, Steam Machines):
*   **The Library**: Hosting a vast, offline-first repository of survival, technical, and cultural knowledge.
*   **The Librarian**: A local, privacy-focused LLM (Large Language Model) that acts as an intelligent interface to The Library, answering complex questions without internet access.
*   **Pubsub**: A lightweight publish-subscribe messaging system to facilitate communication between local applications and services and maintain updated records and library content.
*   **Kasm Workspaces**: Containerized applications accessible via web browser, providing a variety of tools and environments for users.
*   **Collaboration Tools**: Local instances of GitLab, Nextcloud, and other collaboration platforms to facilitate teamwork and resource sharing within the community.
*   **Knoledge Management Systems**: GitLab/Pages for documentation, wikis, and project management.
*   **Open WebUI**: An open source chat interface for interacting with all the bleeding edge LLMs.
*   **Image Generation**: Local AI-powered image generation tools for creative and practical applications.
*   **Code Generation**: Local AI-powered code generation tools to assist with software development and automation.
*   **Music Generation**: Local AI-powered music generation tools for creative expression and ambiance.
*   **Offline-First Design**: All services are designed to function fully offline, ensuring reliability in remote locations.
*   **Internet-In-A-Box**:
    - Complete local copy of Wikipedia
    - OpenStreetMaps
    - Local copies of other educational and reference materials

### 2. Mesh Integration (The Cyberpony Express)
LocalNet integrates directly with **The Cyberpony Express**, HDI's public Meshtastic-based mesh network.
*   Acts as a bridge between the local high-bandwidth intranet and the low-bandwidth, long-range LoRa mesh.
*   Facilitates secure communication between disparate outposts (e.g., High Ground, Mammoth, Slab City).
*   Hosts the **BBS**, allowing for asynchronous messaging, bulletin boards, and lightweight applications over the mesh.
*   Reords data from regional sensors for things like weather forecasting, environmental monitoring, and resource management. 

### 3. Community Autonomy
*   **Hands-Off Operation**: Designed for stability in remote locations with minimal maintenance.
*   **Modular Tool Stacks**: Allows each outpost to deploy additional local tools specific to their guild needs (e.g., inventory for Artificers, sensor logging for Floramancers).
*   **Robot Orchestration**: Integrate with local robotics for tasks like environmental monitoring, agriculture, construction, and security.

### 4. Simulation & Recreation
LocalNet hosts persistent game worlds that serve as both recreation and educational simulations, deeply integrated with the outpost's knowledge systems:
*   **Game Servers**: Hosting for **Minecraft** and **Factorio** to provide shared creative spaces and logistical simulations.
*   **Integrated MUDs**: Text-based Multi-User Dungeons that function as rich, low-bandwidth virtual environments. These MUDs are not just games but interfaces; they integrate with the **KMS**, **The Library**, and **Expert Systems** like **The Librarian**, allowing users to interact with outpost data and AI agents within the game world.

## Architecture
LocalNet is designed to run on edge devices and integrates with:
*   **Meshtastic** hardware for LoRa communications.
*   **Local Storage** for The Library.
*   **Local Compute** for The Librarian AI services.
