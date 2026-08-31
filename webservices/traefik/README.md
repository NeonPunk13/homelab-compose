# Traefik — webservices VM (301, 10.0.50.31)

Fresh rebuild 2026-08-24 (pre-INC-002 salvage used as reference only).

Docker provider removed 2026-08-31 — **file provider only**, no `docker.sock`.

- **DNS-01 ACME via Cloudflare** — wildcard `*.ghostlab.space`, no inbound exposure.
- **The wildcard is declared once, on the `websecure` entryPoint** (`traefik.yml`).
  Routers in `dynamic/services.yml` carry **no `tls:` block** — a router-level block
  overrides the entryPoint default and reverts that host to a per-hostname cert.
- **Routing is explicit, not discovered.** Add a service by adding a router + service
  to `dynamic/services.yml`. Labels on other containers do nothing; a container can no
  longer publish itself (the INC-006 mechanism is gone, not disabled).
- A router referencing a **missing middleware fails to register** — it does not skip it.
  `lan-only` is VM-only (`dynamic/local.yml`), so forgetting that file on redeploy takes
  down every route, not just the dashboard.
- Dashboard chain: `lan-only` → `rate-limit` → `dashboard-auth` → `security-headers`.
  IP allowlist runs FIRST, so off-LAN requests 403 before reaching a password prompt.

## Deploy — files NOT in git, create on the VM
| File | Purpose |
|---|---|
| `.env` | `DOMAIN` + `CF_DNS_API_TOKEN` (Bitwarden) |
| `dynamic/local.yml` | real LAN CIDRs — copy from `local.yml.example` |
| `dynamic/.htpasswd` | `htpasswd -Bc dynamic/.htpasswd luke` |
| `acme/acme.json` | `touch` + `chmod 600`; certs land here |

> **Deploy from THIS directory.** The repo root also has `traefik/` — the pre-INC-002
> salvage (v3.0, DEBUG logging, `docker.sock`, kasm routes). It is one tab-completion
> away and scp'ing from it will quietly downgrade the VM.

## Gotchas (all cost time on 2026-08-24)
- **Traefik must be ≥ 3.6.1** with Docker Engine 29+. Earlier versions hardcode Docker
  API 1.24; Docker 29 requires ≥1.40. Symptom: `client version 1.24 is too old`.
  `DOCKER_API_VERSION` does **nothing** — Traefik ignores it.
- **Do NOT set `resolvers:` under `dnsChallenge`.** Pointing it at 1.1.1.1 forces
  validation through a caching public resolver and DNS-01 times out. Default
  behaviour (query the zone's authoritative NS) works.
- Cloudflare token needs **Zone:DNS:Edit AND Zone:Zone:Read**. DNS:Edit alone → 403.
- `acme.json` is pretty-printed: grep `'"main": *"'`, not `'"main":"'`.
- Debug against the LE **staging** CA — production allows only 5 failed
  authorizations per hostname per hour.

## Exposure
No WAN port forward. PFW001 was deleted 2026-08-24 after it accidentally published
this dashboard when VM 301 reoccupied 10.0.50.31. External access will be via
**Cloudflare Tunnel + Access**, never an inbound port.
