# Uptime Kuma

Runs on the **PBS Beelink (10.0.0.62)** at `/opt/stacks/uptime-kuma`.
Deliberately on PBS: it must survive the PVE host dying (INC-005 black-box watcher).

- UI: http://10.0.0.62:3001  (admin credentials in Bitwarden)
- `network_mode: host` — needed for ICMP ping monitors and to reach the host's
  Postfix at 127.0.0.1:25 for email notifications.
- **NOT in git:** `data/` (SQLite DB: monitors, history, notification config).
  Rebuildable from scratch; no secrets are committed here.
