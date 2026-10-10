# Home Assistant Core on Rootless Podman (Cross-Platform)

This repository contains a production-ready configuration to deploy **Home Assistant Core** as a rootless container using **Podman** and **Podman Compose**. It dynamically adapts to both **macOS** (Development) and **Ubuntu Linux** (Production) using environment variables.

## 🚀 Quick Start Commands

To build, (re)create, and start the Home Assistant service, while immediately cleaning up unused dangling images, run the following command chain:

```bash
podman-compose up -d --build --force-recreate && podman image prune -f
```

## 📋 Environment Configuration (`.env`)

A `.env` file must reside in the same directory as your `docker-compose.yml` to toggle between platforms:

### For macOS (Development Mode)
```ini
HA_NETWORK_MODE=bridge
HA_PORT=8123
```
*Result: Home Assistant will be accessible on your Mac at `http://localhost:8123` via port forwarding.*

### For Ubuntu Linux (Production Mode)
```ini
HA_NETWORK_MODE=host
HA_PORT=8123
```
*Result: Enforces native host networking, which is required for Home Assistant to auto-discover smart home devices (mDNS/UPnP) in your local network.*

---

## 🛠️ Host Prerequisites (Ubuntu Linux only)

Since Podman runs in **rootless mode**, your Linux host user needs explicit permissions to interact with USB serial adapters (e.g., Zigbee/Z-Wave Dongles).

1. Create the persistent configuration directory:
   ```bash
   mkdir -p ~/app-data/homeassistant
   ```
2. Add your user to the `dialout` group:
   ```bash
   sudo usermod -aG dialout $USER
   ```
3. **Crucial:** You must log out and log back in, or **reboot your server**, for these group changes to take effect.

---

## 🪵 Log Management

Logs are automatically managed natively via the `k8s-file` driver. Container logs are capped at **10MB** per file and rotated, keeping a maximum of **3** historical log files to prevent your host storage from running full.
