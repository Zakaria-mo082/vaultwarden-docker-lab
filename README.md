# vaultwarden-docker-lab

# Self-Hosted Vaultwarden Stack on WSL2 & Docker

My self-hosted password manager setup running Vaultwarden in Docker. It's capped on system resources and locked down behind a Caddy reverse proxy.

---

## 🏗️ Architecture Overview

[ Browser / Extension ]
│
(HTTPS: Port 8443)
▼
[ Caddy Proxy ] ──(Local TLS)
│
(Internal Network)
▼
[ Vaultwarden Server ]
│
(Persistent Data)
▼
[ Docker Volume ]

---

## 🛠️ Tech Stack & Requirements

* **OS / Environment:** Windows 11 with WSL2 (Ubuntu)
* **Container Tools:** Docker Desktop & Docker Compose
* **Management UI:** Portainer-CE
* **Core Application:** Vaultwarden (a lightweight Bitwarden server written in Rust)
* **Reverse Proxy:** Caddy 2 (handles local HTTPS & SSL certificates automatically)

---

## ⚙️ Key Technical Details & Hardening

### 1. Keeping WSL2 Resource Usage in Check (`.wslconfig`)
By default, WSL2 can grab as much host memory as it wants. To keep the rest of the host system running smoothly alongside the containers, I capped WSL2's hardware allocation:
* **RAM Limit:** 2 GB
* **CPU Limit:** 2 Virtual Cores

### 2. Disabling Public Registrations (`SIGNUPS_ALLOWED=false`)
Since this is a private instance, public sign-ups are turned off at the container level. Anyone trying to register an account without an invite gets blocked immediately, keeping unauthorized users out.

![Security Hardening Proof](assets/security-hardening.png)

### 3. Automatic Local HTTPS with Caddy
Modern browsers and Bitwarden browser extensions block non-HTTPS password managers for security reasons. I put Caddy in front of Vaultwarden to automatically generate local SSL certificates and seamlessly route incoming traffic on port `8443` straight to Vaultwarden's backend on port `80`.

### 4. Troubleshooting: Container Networking in Portainer
**Problem:** When initially deploying the Caddy reverse proxy alongside Vaultwarden through Portainer stacks, Caddy failed to resolve the `vaultwarden` container hostname, resulting in proxy connection errors (`502 Bad Gateway`).

**Solution:** Portainer creates isolated custom networks per stack by default. To resolve this, I explicitly defined a shared bridge network (`vw-net`) inside the `docker-compose.yml` and attached both services to it. This ensured internal DNS resolution worked seamlessly between Caddy and Vaultwarden while keeping external access restricted strictly through Caddy's HTTPS port `8443`.

---

## 🚀 Quick Start Guide

