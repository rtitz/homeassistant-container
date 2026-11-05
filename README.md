# Home Assistant Core on Rootless Podman (Cross-Platform)

This repository contains a production-ready configuration to deploy **Home Assistant Core** as a rootless container using **Podman** and **Podman Compose**. It natively handles unprivileged multi-device environments and targets both **macOS** (Development) and **Linux/Raspberry Pi OS** (Production).

## 🚀 Quick Start Commands

To build, (re)create, and start the Home Assistant service, while immediately cleaning up unused dangling images, run the following command chain:

```bash
podman-compose up -d --build --force-recreate && podman image prune -f
```

---

## 📦 Installing Podman & Host Prerequisites (New Raspberry Pi Setup)

If you are setting up a brand-new Raspberry Pi, run these exact command steps to install the runtime stack and prepare the host operating system for persistent rootless container execution:

### 1. Update the Operating System & Install Podman
Ensure all system indices are up to date, then install the native Podman container engine along with its required unprivileged subsystem packages:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
  podman \
  podman-compose \
  podman-docker \
  uidmap \
  fuse-overlayfs
```

### 2. Enable Persistent Background Execution (Lingering)
By default, the Linux kernel terminates rootless containers the moment your active SSH terminal session logs out. Force the system to keep your containers running forever and spin them up instantly on power cycles:
```bash
sudo loginctl enable-linger \$USER
```

### 3. Grant USB Hardware Access Permissions
Add your unprivileged deployment user to the system `dialout` group so your containers can securely talk to physical USB smart-home coordinators (like Zigbee or Z-Wave sticks):
```bash
sudo usermod -aG dialout \$USER
```

### 4. Provision Storage & Apply a System Reboot
Create your primary rootless configuration storage directory path and reboot the hardware to securely apply your new group and lingering configurations:
```bash
mkdir -p ~/app-data/homeassistant && sudo reboot
```

### 5. Verify Your Installation Namespaces
Once the Pi boots back up, log in and verify that the runtime packages and unprivileged user mappings are fully functional:
```bash
podman --version && podman-compose --version
```

---

## 📋 Network Architecture & Platform Tailoring

The Compose manifest isolates platforms using a streamlined **`network_mode`** matching system:

### For Linux Nodes (Production Mode)
```yaml
network_mode: host
```
* **Behavior:** Bypasses virtual bridges to share the host's native network stack. 
* **Advantage:** Enforces the Layer 2 visibility required for Home Assistant to handle smart-home auto-discovery (mDNS/UPnP) and communicate with local exporters (e.g., `solar-monitor` and `wallbox-monitor`) seamlessly via `localhost`.

### For macOS Nodes (Development Mode)
```yaml
network_mode: bridge
```
* **Behavior:** Standard isolated internal bridging network context.
* **Advantage:** Bypasses macOS host-networking constraints to route the ecosystem via port translation flags during development tasks.

---

## 🎛️ USB Hardware Pass-Through (Zigbee / Z-Wave)

Containers are isolated from physical system paths by default. To pass a physical USB coordinator through to your rootless stack, open the `docker-compose.yml` manifest file, uncomment the `devices:` block, and select your hardware line:

```yaml
          # INSTRUCTION: To activate, uncomment 'devices:' and ONE hardware line below:
          #devices:
          #  - /dev/serial/by-id/usb-ITRIAD_Sonoff_Zigbee_3.0_USB_Dongle_Plus_0001-if00-port0:/dev/ttyUSB0
          #  - /dev/serial/by-id/usb-Aeotec_Z-Stick_Gen5_Plus_0001-if00-port0:/dev/ttyACM0
```
*Note: Always map devices by their persistent `/dev/serial/by-id/...` hardware signature paths to prevent device naming swaps when plugging or unplugging items on host reboots.*

---

## 🪵 Log Management

Logs are automatically managed natively via the `k8s-file` driver configuration profile. Active container event tracking logs are capped at **10MB** per file and rotated, keeping a maximum of **3** historical log file fragments to protect host flash storage metrics from fill depletion errors.
