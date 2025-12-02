# Initial work on creating a Factorio server Docker Compose stack that uses Tailscale for networking.

* Tailscale **sidecar** joins your tailnet.
* Factorio server shares Tailscale’s network namespace.
* Saves **and mods** live on the host in `/srv/factorio` (so you can also export them via Samba).
* Updating is just: `docker compose pull && docker compose up -d`.

```yaml
services:
  tailscale-factorio:
    image: tailscale/tailscale:stable
    container_name: tailscale-factorio
    hostname: factorio
    restart: unless-stopped
    volumes:
      - ./tailscale-state:/var/lib/tailscale
    environment:
      # Tailscale config
      TS_STATE_DIR: /var/lib/tailscale
      TS_USERSPACE: "false"            # use kernel networking (needs tun + NET_ADMIN)
      TS_AUTH_ONCE: "true"             # only auth once; keep state on disk

      # Set these in a .env file
      TS_AUTHKEY: ${TS_AUTHKEY}      # optional: auth key from Tailscale admin
      TS_EXTRA_ARGS: "--advertise-tags=tag:game-server"  # optional tags for ACLs
    cap_add:
      - net_admin
      - sys_module
    devices:
      - /dev/net/tun:/dev/net/tun

  factorio:
    image: factoriotools/factorio:stable
    container_name: factorio
    # Share the Tailscale network namespace
    network_mode: "service:tailscale-factorio"
    restart: unless-stopped
    depends_on:
      - tailscale
    volumes:
      # Host directory for all Factorio data (saves, mods, config)
      # On the host:
      #   Saves  -> /srv/factorio/saves
      #   Mods   -> /srv/factorio/mods
      #   Config -> /srv/factorio/config
      - /srv/factorio:/factorio
    environment:
      # If you set a specific save name, Factorio will load that.
      # For example: SAVE_NAME=main  -> main.zip
      SAVE_NAME: ${SAVE_NAME:-main}
      LOAD_LATEST_SAVE: "false"       # "true" to always load latest instead
      # RCON (optional but handy)
      RCON_PORT: "27015"
      RCON_PASSWORD: ${RCON_PASSWORD:-changeme}
      # You can also add things like:
      # SERVER_NAME: "My Factorio Server"
      # SERVER_DESCRIPTION: "Running in Docker over Tailscale"
```

### How to use this

1. **Create a directory** for the stack, e.g.:

   ```bash
   mkdir -p ~/factorio-stack
   cd ~/factorio-stack
   ```

2. **Save the file** above as `docker-compose.yml` in that directory.

3. **Create directories for data/state:**

   ```bash
   sudo mkdir -p /srv/factorio
   mkdir -p tailscale-state
   ```

4. (Optional but recommended) **Create a ************************`.env`************************ file** in the same directory:

   ```bash
   cat > .env << 'EOF'
   # Factorio
   SAVE_NAME=main
   RCON_PASSWORD=supersecret

   # Tailscale
   TS_AUTHKEY=tskey-auth-REPLACE_ME
   EOF
   ```

   Or leave `TS_AUTHKEY` empty and do the “click login URL from `docker logs tailscale-factorio`” flow once.

5. **Start / update the stack:**

   ```bash
   docker compose pull
   docker compose up -d
   ```

   * `pull` grabs the latest `factoriotools/factorio` and `tailscale/tailscale`.
   * `up -d` recreates containers as needed, but **keeps**:

     * `/srv/factorio` (your saves, mods, config; Samba can export this)
     * `./tailscale-state` (your Tailscale login state)

6. **Connect from your client:**

   * From any device on your tailnet:

     * Use `factorio:34197` (UDP) as the server address, or
     * Use the Tailscale IP of this node (shown in the Tailscale admin UI).

   Factorio is *not* exposed on your LAN or the public internet; it’s only reachable via Tailscale.

---

### Making yourself admin (equivalent to `/promote cjtrowbridge`)

Instead of sending `/promote` every time the server starts, Factorio lets you **persist admin status** via `server-adminlist.json` in the data directory. Since `/srv/factorio` is mounted into the container as `/factorio`, you can do this on the host:

1. Create or edit `/srv/factorio/server-adminlist.json`:

   ```bash
   sudo tee /srv/factorio/server-adminlist.json > /dev/null << 'EOF'
   [
     { "name": "cjtrowbridge", "admin": true }
   ]
   EOF
   ```

2. Restart the stack so Factorio reloads its config:

   ```bash
   docker compose up -d
   ```

Now every time the server starts, `cjtrowbridge` is already promoted, with no need to send `/promote` by hand or via RCON.

---

### Mods on the host

With the `- /srv/factorio:/factorio` volume, your mods are just files on the host here:

```bash
ls /srv/factorio/mods
```

Drop your `.zip` mod files and `mod-list.json` into `/srv/factorio/mods`; the container will see them at `/factorio/mods` and Factorio will load them normally.

This also means your mods – along with saves and config – are automatically included in any Samba share you point at `/srv/factorio`.

---

### Helper script: ensure Samba share + rebuild the stack

If you want a simple helper script that both **ensures a Samba share** for `/srv/factorio` and **rebuilds the containers**, you can use something like this (Debian/Ubuntu-ish host; tweak for your distro):

Save as `update_factorio_stack.sh` next to your `docker-compose.yml`:

```bash
#!/usr/bin/env bash
set -euo pipefail

FACTORIO_PATH="/srv/factorio"
SMB_CONF="/etc/samba/smb.conf"
SHARE_NAME="factorio"

# 1) Ensure Factorio data dir exists
sudo mkdir -p "${FACTORIO_PATH}"

# Optionally give your user ownership so you can manage files directly
sudo chown -R "${USER}:${USER}" "${FACTORIO_PATH}" || true

# 2) Ensure a Samba share exists for /srv/factorio
if ! grep -q "^\[${SHARE_NAME}\]" "${SMB_CONF}"; then
  echo "Adding Samba share [${SHARE_NAME}] -> ${FACTORIO_PATH} to ${SMB_CONF}" >&2
  sudo bash -c "cat >> '${SMB_CONF}' << EOF

[${SHARE_NAME}]
   path = ${FACTORIO_PATH}
   browseable = yes
   read only = no
   guest ok = no
   create mask = 0660
   directory mask = 0770
EOF
"
fi

# 3) Restart Samba (try common service names, ignore failures)
if command -v systemctl >/dev/null 2>&1; then
  sudo systemctl restart smbd || sudo systemctl restart samba || true
fi

# 4) Rebuild / update the Docker stack
docker compose pull
docker compose up -d
```

Then:

```bash
chmod +x update_factorio_stack.sh
./update_factorio_stack.sh
```

That will:

* Make sure `/srv/factorio` exists and is writable by you.
* Add (once) a `[factorio]` Samba share pointing at `/srv/factorio`.
* Restart Samba so the share appears on your LAN.
* Pull the latest images and bring the Factorio + Tailscale stack up.

From another machine on your LAN, you’ll see a Samba share named `factorio`, containing:

* `/srv/factorio/saves` – your save games.
* `/srv/factorio/mods` – your mods.
* `/srv/factorio/config` and related files.

You can now manage saves/mods directly over the network while still keeping the server itself reachable only via Tailscale.
