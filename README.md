Here is a structured implementation plan designed for you to feed directly into **Claude Code**. It outlines the architecture, constraints, and specific execution steps using the Vultr CLI and standard Linux tools.

---

### **Project Spec: Vultr Ingress Node**

**Objective:** Provision a Vultr VPS to act as a public ingress gateway for home lab services (excluding Jellyfin). Traffic will enter the VPS via public 443/80, be terminated by Caddy, and routed over Tailscale to home servers.

**Architecture:**

* **Provider:** Vultr (Region: Atlanta `atl`, Plan: High Frequency).
* **OS:** Ubuntu 24.04 LTS.
* **Networking:**
* Public Internet  VPS (Caddy)  Tailscale  Home Server.
* **Exception:** `jellyfin.or-ricon.com` bypasses this VPS (DNS points directly to Home IP).


* **Optimization:** TCP BBR enabled for latency reduction.

---

### **Phase 1: Provisioning (Vultr CLI)**

**Instruction for Claude Code:**
"Use the `vultr-cli` to provision a new instance with the following specifications. If you need an API key, ask the user."

1. **List Plans/Regions:** Confirm availability of "High Frequency" (NVMe) plans in Atlanta (`atl`).
2. **Create Instance:**
* **Region:** Atlanta (`atl`)
* **Plan:** 1 vCore / 1GB RAM High Frequency (approx. $6/mo).
* **OS:** Ubuntu 24.04 x64.
* **Label:** `ingress-gateway-atl`
* **SSH Keys:** Add the user's current public SSH key.


3. **Output:** Wait for the instance to reach `active` status and retrieve the **Public IP**.

### **Phase 2: System Configuration**

**Instruction for Claude Code:**
"SSH into the new server using the retrieved IP. Execute the following setup steps to prepare the environment."

1. **System Updates:** Run `apt update && apt upgrade -y`.
2. **Enable BBR (Latency Optimization):**
* Append `net.core.default_qdisc=fq` and `net.ipv4.tcp_congestion_control=bbr` to `/etc/sysctl.conf`.
* Run `sysctl -p`.


3. **Install Tailscale:**
* Run the official install script: `curl -fsSL https://tailscale.com/install.sh | sh`.
* **Action:** Ask the user for a **Tailscale Auth Key** (Ephemeral, Reusable, tagged `tag:server` recommended).
* Run `tailscale up --authkey <KEY> --ssh`.


4. **Install Caddy:**
* Install the stable release from the official Caddy repository (Debian/Ubuntu instructions).



### **Phase 3: Caddy Configuration**

**Instruction for Claude Code:**
"Configure Caddy to act as the reverse proxy. Create the configuration file at `/etc/caddy/Caddyfile`."

**Draft `Caddyfile` Content:**

```caddy
{
    # Secure admin endpoint
    admin localhost:2019
}

# Define your services here.
# Note: Jellyfin is EXCLUDED from this file as it is served directly from home.

# Example 1: Overseerr (Proxied over Tailscale)
overseerr.or-ricon.com {
    # Replace with the actual Tailscale IP of your home server
    reverse_proxy 100.x.y.z:5055
}

# Example 2: Home Assistant
ha.or-ricon.com {
    reverse_proxy 100.x.y.z:8123
}

```

**Post-Creation Actions:**

1. **Tailscale IPs:** Ask the user for the Tailscale IP addresses of the destination home servers to populate the placeholders (`100.x.y.z`).
2. **Restart:** Run `systemctl restart caddy`.
3. **Verify:** Check status with `systemctl status caddy`.

### **Phase 4: Final Verification**

**Instruction for Claude Code:**

1. **DNS Check:** Remind the user to update their DNS A records for `*.or-ricon.com` (excluding Jellyfin) to point to the new **Vultr VPS IP**.
2. **Connection Test:** Attempt to `curl -v https://overseerr.or-ricon.com` to verify SSL termination and proxy pass.

---

**Next Step:**
Would you like me to generate a **Tailscale Auth Key** link for you to grab a key before you hand this off to Claude?

