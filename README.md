# 🏠 homelab

Personal cybersecurity homelab running on a headless Ubuntu 24.04 server. Built to practice threat detection, log analysis, and SOC workflows using real tools.

![Architecture](homelab-diagram.svg)

---

## 🧰 Stack

| Tool | Purpose | Port |
|---|---|---|
| 🛡️ [Wazuh](https://wazuh.com) | SIEM — real-time threat detection, vulnerability scanning, log aggregation | `:443` |
| 🍯 [Cowrie](https://github.com/cowrie/cowrie) | SSH honeypot — captures attacker IPs, credentials, and commands | `:2222` |
| 📊 [Grafana](https://grafana.com) + [Prometheus](https://prometheus.io) | Server metrics — CPU, RAM, disk, network over time | `:3000` |
| 🟢 [Uptime Kuma](https://github.com/louislam/uptime-kuma) | Service uptime monitoring with alerts | `:3001` |
| 🐳 [Portainer](https://www.portainer.io) | Docker management UI | `:9443` |
| 🔄 [Watchtower](https://containrrr.dev/watchtower/) | Automatic container updates | — |
| 🔒 [Tailscale](https://tailscale.com) | Secure remote access from anywhere | — |

---

## ⚙️ What this lab does

**🔍 Threat detection** — Wazuh monitors the host in real time. SSH brute force attempts, privilege escalation, file integrity changes, and Cowrie honeypot events all generate alerts mapped to MITRE ATT&CK techniques.

**🍯 Honeypot intelligence** — Cowrie runs on port 2222 and pretends to be a vulnerable SSH server. Every attacker IP, attempted password, and command gets logged. Real bots hit it daily.

**🧑‍💻 SOC practice** — I use the lab to simulate attacks, triage alerts in Wazuh, correlate events across log sources, and write incident reports. This mirrors a real Tier 1 SOC analyst workflow.

**📈 Metrics and uptime** — Grafana dashboards show live server health. Uptime Kuma monitors all services and alerts on outages.

---

## 🚀 Setup

### ✅ Requirements
- Ubuntu 22.04+
- Docker and Docker Compose
- 8GB+ RAM (Wazuh needs at least 4GB alone)

### 1️⃣ Install Docker
```bash
sudo apt update && sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update && sudo apt install docker-ce docker-ce-cli \
  containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
```

### 2️⃣ Start Wazuh
```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.11.2
cd wazuh-docker/single-node
sudo sysctl -w vm.max_map_count=262144
docker compose -f generate-indexer-certs.yml run --rm generator
docker compose up -d
```
Dashboard at `https://YOUR_IP` — set `GRAFANA_PASSWORD` env var before starting (defaults to `admin`).

### 3️⃣ Install Wazuh agent on host
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg \
  --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
sudo chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
  https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update && sudo apt install wazuh-agent
sudo sed -i 's/MANAGER_IP/YOUR_IP/' /var/ossec/etc/ossec.conf
sudo systemctl enable --now wazuh-agent
```

### 4️⃣ Start the rest of the stack
```bash
git clone https://github.com/s3vtyq/homelab
cd homelab
docker compose up -d
```

### 5️⃣ Remote access via Tailscale
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --accept-dns=false
sudo systemctl enable tailscaled
```

---

## 🔴 SOC Practice — Example Incident

```
Incident ID  : INC-001
Date         : 2026-03-20
Source IP    : 192.168.0.102
Attack type  : SSH Brute Force
Attempts     : 20 login attempts in 45 seconds
Outcome      : Contained by Cowrie honeypot
Real system  : Not affected
MITRE ATT&CK : T1110.001 — Password Guessing
```

---

## 🏷️ Skills

![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=flat&logo=ubuntu&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=flat&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)
![Tailscale](https://img.shields.io/badge/Tailscale-242424?style=flat&logo=tailscale&logoColor=white)
