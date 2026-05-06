# 🔭 AI-Based Observability Project

> **AI-powered observability pipeline using OpenTelemetry, Honeycomb, and Claude MCP on AWS EC2 with Kubernetes**

![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazon-aws)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Minikube-blue?logo=kubernetes)
![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-Demo-blueviolet?logo=opentelemetry)
![Honeycomb](https://img.shields.io/badge/Honeycomb-MCP-yellow)
![Claude](https://img.shields.io/badge/Claude-AI%20Debugging-orange)

---

## 📌 Project Overview

This project builds a complete **AI-powered observability stack** that lets you debug distributed systems using **natural language** instead of writing complex queries.

Instead of manually digging through dashboards, you simply ask:
> *"Why is my checkout service slow?"*
> *"Show me the 10 slowest endpoints in the last hour."*

And get detailed technical answers backed by real telemetry data.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                  AWS EC2 (t3.xlarge)                 │
│                                                       │
│  ┌─────────────────────────────────────────────┐    │
│  │           Minikube (Kubernetes)              │    │
│  │                                              │    │
│  │   OTel Demo App (Astronomy Shop)            │    │
│  │   20+ Microservices                         │    │
│  │   ├── frontend-proxy                        │    │
│  │   ├── cartservice                           │    │
│  │   ├── checkoutservice                       │    │
│  │   ├── productcatalogservice                 │    │
│  │   ├── kafka, redis, postgresql              │    │
│  │   └── otel-collector ──────────────────────┼──┐ │
│  └─────────────────────────────────────────────┘  │ │
└───────────────────────────────────────────────────┼─┘
                                                    │
                                                    ▼
                                          ┌─────────────────┐
                                          │  Honeycomb.io   │
                                          │  (Traces/Logs/  │
                                          │   Metrics)      │
                                          └────────┬────────┘
                                                   │
                                                   ▼
                                          ┌─────────────────┐
                                          │  Claude Desktop │
                                          │  + MCP Server   │
                                          │  (AI Debugging) │
                                          └─────────────────┘
```

---

## 🛠️ Tech Stack

| Component | Tool | Purpose |
|---|---|---|
| Cloud | AWS EC2 (t3.xlarge) | Host the entire stack |
| Container Runtime | Docker | Run Minikube containers |
| Kubernetes | Minikube | Local K8s cluster |
| App | OTel Demo (Astronomy Shop) | 20+ microservices workload |
| Package Manager | Helm | Deploy OTel demo app |
| Telemetry | OpenTelemetry Collector | Collect traces/logs/metrics |
| Backend | Honeycomb.io | Store and visualize telemetry |
| AI Layer | Claude Desktop + MCP | Natural language debugging |
| Editor | VS Code + Remote SSH | Remote development |

---

## ✅ Prerequisites

- AWS Account (free tier or paid)
- Windows local machine
- VS Code installed
- Claude Desktop installed
- Honeycomb.io account (free tier)
- Node.js installed (for MCP server)

---

## 🚀 Step-by-Step Setup

### ⚡ STEP 1: Launch AWS EC2 Instance

#### EC2 Configuration
| Setting | Value |
|---|---|
| Name | Ai-observability-project |
| AMI | Ubuntu 22.04 LTS |
| Instance Type | t3.xlarge (4 vCPU, 16GB RAM) |
| Key Pair | Create new → save .pem file safely |
| Storage | 30 GiB gp3 |

#### Security Group Rules
| Type | Port | Source |
|---|---|---|
| SSH | 22 | My IP |
| Custom TCP | 8080 | Anywhere (0.0.0.0/0) |
| Custom TCP | 443 | Anywhere (0.0.0.0/0) |

#### SSH into EC2 (Windows PowerShell)
```powershell
cd Downloads
icacls "Ai-observability-project.pem" /inheritance:r /grant:r "%USERNAME%:R"
ssh -i "Ai-observability-project.pem" ubuntu@<YOUR_EC2_PUBLIC_IP>
```

#### VS Code Remote SSH Setup
Add to `C:\Users\<username>\.ssh\config`:
```
Host ec2-observability
    HostName <YOUR_EC2_PUBLIC_IP>
    User ubuntu
    IdentityFile C:\Users\<username>\Downloads\Ai-observability-project.pem
```
Then: `CTRL+SHIFT+P` → `Remote-SSH: Connect to Host` → `ec2-observability`

> ⚠️ **Tip:** If SSH times out, your IP may have changed. Update Security Group SSH rule to "My IP" again. Use Elastic IP to fix this permanently.

---

### 🔧 STEP 2: Install Required Tools on EC2

#### Update System
```bash
sudo apt update && sudo apt upgrade -y
```

#### Install Docker
```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

sudo usermod -aG docker $USER
newgrp docker

# Verify
docker --version
```

#### Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify
kubectl version --client
```

#### Install Minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64

# Verify
minikube version
```

#### Install Helm
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

---

### ☸️ STEP 3: Start Minikube Cluster

```bash
minikube start --cpus=4 --memory=10240 --driver=docker
```

> ⚠️ Takes 3-5 minutes. Wait for "Done!" message.

#### Verify Cluster
```bash
kubectl get nodes
```
Expected output:
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   Xm    v1.35.x
```

> **Why these resources?**
> - `--cpus=4` → 20+ microservices need CPU
> - `--memory=10240` → Each service uses 200-500MB RAM
> - `--driver=docker` → Uses Docker as container runtime

---

### 🍯 STEP 4: Honeycomb.io Setup

1. Go to [https://ui.honeycomb.io](https://ui.honeycomb.io)
2. Sign up (free tier available)
3. Create an Environment (e.g., `Ai-otel`)
4. Go to **Settings → API Keys → Ingest tab**
5. Create new Ingest API Key → name it `otel-demo`
6. Copy the key (starts with `hcaik_...`)

> ⚠️ **Security:** Never share your API key publicly, in screenshots, or in chat!

---

### 📦 STEP 5: Install OTel Helm Chart & Deploy

#### Add OTel Helm Repo
```bash
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
```

#### Create Namespace
```bash
kubectl create namespace otel-demo
```

#### Create values.yaml
```bash
nano values.yaml
```

Paste the following (replace `YOUR_API_KEY_HERE` with your Honeycomb Ingest key):

```yaml
opentelemetry-collector:
  enabled: true

  config:
    receivers:
      otlp:
        protocols:
          http: {}
          grpc: {}

    processors:
      batch: {}

    exporters:
      otlphttp/honeycomb:
        endpoint: https://api.honeycomb.io
        headers:
          x-honeycomb-team: "YOUR_API_KEY_HERE"
          x-honeycomb-dataset: "otel-demo"

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlphttp/honeycomb]

        metrics:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlphttp/honeycomb]

        logs:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlphttp/honeycomb]
```

Save: `CTRL+X → Y → Enter`

#### Deploy OTel Demo App
```bash
helm upgrade --install otel-demo open-telemetry/opentelemetry-demo \
  -n otel-demo -f values.yaml
```

> ⚠️ Takes 5-10 minutes to pull all images.

#### Verify Pods
```bash
# Check status
kubectl get pods -n otel-demo

# Watch live
watch kubectl get pods -n otel-demo
```

---

### 🌐 STEP 6: Access the Astronomy Shop App

#### Port Forward
```bash
kubectl port-forward -n otel-demo svc/frontend-proxy 8080:8080 --address 0.0.0.0
```

#### Open in Browser
```
http://<YOUR_EC2_PUBLIC_IP>:8080
```

You should see the **Astronomy Shop** e-commerce storefront! 🛍️

---

### 📊 STEP 7: Verify Honeycomb Dashboard

1. Open [https://ui.honeycomb.io](https://ui.honeycomb.io)
2. Select dataset: **otel-demo**
3. Verify:
   - Traces arriving ✅
   - Span waterfalls ✅
   - Latency graphs ✅
   - Service map ✅
   - Error rates ✅

---

### 🤖 STEP 8: Set Up Honeycomb MCP with Claude Desktop

This is the **AI magic** — connecting Honeycomb telemetry to Claude for natural language debugging!

#### Option A — Claude.ai Connectors (Easiest! ✅)
1. Go to [https://claude.ai](https://claude.ai)
2. Settings → **Connectors**
3. Find **Honeycomb** → Click Connect
4. Authenticate with OAuth
5. Done! Start asking questions in Claude chat!

#### Option B — Claude Desktop + MCP Server

Install Honeycomb MCP server globally:
```cmd
npm install -g @kajirita2002/honeycomb-mcp-server
```

Add to Claude Desktop config (`%APPDATA%\Claude\claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "honeycomb": {
      "command": "C:\\Users\\<username>\\AppData\\Roaming\\npm\\honeycomb-mcp-server.cmd",
      "env": {
        "HONEYCOMB_API_KEY": "YOUR_INGEST_API_KEY_HERE"
      }
    }
  }
}
```

> ⚠️ Use **Ingest API key** (`hcaik_...`) not Management key (`hcamk_...`)

Restart Claude Desktop → verify in Settings → Developer → Honeycomb shows **running** ✅

---

### 💬 STEP 9: Natural Language AI Debugging

Now ask Claude in natural language:

#### 🔥 Performance Queries
```
Show me the slowest endpoint in the last 20 minutes
List the top 5 slowest services
Which endpoints have p95 latency > 500ms?
What caused the latency spike in the last hour?
Show me traces where checkout took more than 500ms
```

#### ❗ Error Queries
```
List endpoints with the highest error rate
Which service produced the most errors today?
Show me the recent 100 errors grouped by service
```

#### 📊 Traffic Queries
```
Which service handled the most requests in the last hour?
Show me the busiest endpoints right now
What are the top 10 endpoints by request volume?
```

#### 🔍 Debugging Queries
```
Explain the cause of the latest latency spike
Summarize anomalies in the last 30 minutes
Why is checkout slow today?
Show me traces where frontend took more than 1 second
```

---

## 📋 Quick Reference Commands

```bash
# Check Minikube status
minikube status

# List all OTel pods
kubectl get pods -n otel-demo

# List all services
kubectl get svc -n otel-demo

# View pod logs
kubectl logs -n otel-demo <pod-name>

# Access app in browser
kubectl port-forward -n otel-demo svc/frontend-proxy 8080:8080 --address 0.0.0.0

# Stop Minikube (save AWS costs!)
minikube stop

# Restart Minikube
minikube start --cpus=4 --memory=10240 --driver=docker

# Redeploy OTel demo
helm upgrade --install otel-demo open-telemetry/opentelemetry-demo -n otel-demo -f values.yaml

# Edit values.yaml
nano values.yaml
```

---

## 🔧 Troubleshooting

| Issue | Fix |
|---|---|
| SSH connection timeout | Update Security Group SSH rule to "My IP" |
| VS Code Remote SSH fails | Check .pem file path in SSH config |
| Pods in Pending state | Wait longer — images still downloading |
| Port forward not working | Check service name: `kubectl get svc -n otel-demo` |
| Honeycomb not receiving data | Verify Ingest API key in values.yaml |
| EC2 IP changed | Use Elastic IP to fix permanently |
| MCP unauthorized error | Use Ingest key (`hcaik_`) not Management key (`hcamk_`) |

---

## 💡 Pro Tips

- **Stop EC2 when not practicing** → saves costs (~$0.17/hr for t3.xlarge)
- **Have 16-20GB RAM locally?** → Skip AWS entirely! Run Minikube directly on your Windows/Mac/Linux machine using Docker Desktop — saves money and works faster with no SSH needed
- **Use Elastic IP** → prevents IP change on every restart
- **Use Claude.ai Connectors** → easiest way to connect Honeycomb MCP (one click!)
- **Watch pods live** → `watch kubectl get pods -n otel-demo`
- **Keep API keys safe** → never share in chat, screenshots, or public repos

---

## 📁 Project Structure

```
ai-observability-project/
│
├── values.yaml          # OTel collector config → routes to Honeycomb
├── README.md            # This file
└── .gitignore           # Ignore sensitive files
```

> ⚠️ **Never commit values.yaml with real API keys to GitHub!**

Add to `.gitignore`:
```
values.yaml
*.pem
```

---

## 🏆 What You Learn

- AWS EC2 provisioning and SSH access
- Kubernetes cluster management with Minikube
- Helm chart deployment and configuration
- OpenTelemetry instrumentation and data routing
- Honeycomb observability platform
- Model Context Protocol (MCP) integration
- AI-powered debugging with Claude
- Real-world DevOps observability workflow

---

## 📚 References

- [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo)
- [Honeycomb.io](https://honeycomb.io)
- [Honeycomb MCP](https://mcp.honeycomb.io)
- [Minikube Docs](https://minikube.sigs.k8s.io/docs/)
- [Original Video by Cloud Champ](https://youtu.be/cQtudMvEYOU)

---

## 👤 Author

**Yashvardhan Sahu**
- Built on AWS EC2 with real hands-on experience
- Part of DevOps job-ready learning journey

---

*⭐ Star this repo if it helped you!*