Beginner‑friendly guide. It’s written for miners and community members who may not have developer experience, so it avoids jargon and walks through everything step by step.

---

# 🌐 Friendly Guide: Setting Up Eiquidus Explorer

This guide is for **newcomers, miners, and community members** who want to host and run an Eiquidus blockchain explorer. No developer background required — we’ll go step by step using a global Node.js + npm setup.

---

## 1. Prepare Your Server

Update your system and install basic tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git curl
```

---

## 2. Install Node.js and npm Globally

We’ll use Node.js LTS (stable version):

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

Check versions:

```bash
node -v
npm -v
```

---

## 3. Install MongoDB

Eiquidus uses MongoDB to store blockchain data.

```bash
sudo apt install -y mongodb
```

Enable and start MongoDB:

```bash
sudo systemctl enable mongodb
sudo systemctl start mongodb
```

---

## 4. Clone the Eiquidus Repository

Pick a folder and clone:

```bash
git clone https://github.com/your-repo/eiquidus.git
cd eiquidus
```

---

## 5. Install Dependencies

Use npm globally:

```bash
npm install -g forever
npm install
```

- `forever` keeps the explorer running in the background.

---

## 6. Configure Settings

Open `settings.json`:

```bash
nano settings.json
```

Set:
- **page_title** → Your explorer name.
- **database** → Database name (e.g., `explorerdb`).
- **coin** → Coin name, symbol, and RPC details.
- **public/logo** → Paths to your branding images.
- **public/img/logo** → Paths to your branding images.

Save and exit.

---

## 7. Initialize Database

Drop any old database (optional):

```bash
mongosh explorerdb --eval "db.dropDatabase()"
```

Run sync scripts:

```bash
node scripts/sync.js
node scripts/index.js
```

---

## 8. Start the Explorer

Run with forever:

```bash
forever start bin/instance
```

Check logs:

```bash
forever list
```

Stop explorer:

```bash
forever stopall
```

---

## 9. Access Your Explorer

Open your browser and go to:

```
http://your-server-ip:3001
```

You should see your blockchain explorer live!

---

## 10. Create the sync service unit

File: /etc/systemd/system/eiquidus-sync.service

---
[Unit]
Description=Eiquidus Sync Job
After=network.target

[Service]
Type=oneshot
WorkingDirectory=/root/eiquidus
ExecStart=/usr/bin/node /root/eiquidus/scripts/sync.js index update
StandardOutput=append:/var/log/eiquidus-sync.log // optional
StandardError=append:/var/log/eiquidus-sync-error.log // optional

Type=oneshot → runs once and exits (perfect for jobs).

ExecStart → points to your sync script.

WorkingDirectory → explorer root.

Logs go to /var/log/eiquidus-sync.log.

---

## 11. Create the timer unit

File: /etc/systemd/system/eiquidus-sync.timer:

```
[Unit]
Description=Run Eiquidus Sync Job every minute

[Timer]
OnBootSec=60s
OnUnitActiveSec=120s
Unit=eiquidus-sync.service

[Install]
WantedBy=timers.target
```

OnBootSec=60s → first run 60 seconds after boot.

OnUnitActiveSec=120s → repeat every minute.

Links to the service above.

---

#### Enable and start

```
sudo systemctl daemon-reload
sudo systemctl enable eiquidus-sync.timer
sudo systemctl start eiquidus-sync.timer
```
#### Check status:
```
systemctl status eiquidus-sync.timer
systemctl list-timers | grep eiquidus-sync
```
Logs:

journalctl -u eiquidus-sync.service -f

```
sync.js runs every 2 minutes under a systemd timer.

No need for cron — everything is managed consistently by systemd.

```

## 🔧 Tips for Newcomers

```
- Always keep Node.js and npm updated.
- Use `forever` or `pm2` to keep the explorer running.
- If images don’t update, use ssh not password.
- Back up your `settings.json` and branding assets before pulling new repo updates.

```

## 🎉 Conclusion
```
You’ve set up Eiquidus Explorer from scratch using a friendly, step‑by‑step approach. Now miners and community members can host their own explorer without needing deep developer knowledge.


---
