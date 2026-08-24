# Traefik — webservices VM (301, 10.0.50.31)

Fresh rebuild 2026-08-24 (pre-INC-002 salvage used as reference only).

- **DNS-01 ACME via Cloudflare** — wildcard `*.ghostlab.space`, no inbound exposure.
- `exposedByDefault: false` — services opt in explicitly via labels.
- Dashboard chain: `lan-only` → `rate-limit` → `dashboard-auth` → `security-headers`.
  IP allowlist runs FIRST, so off-LAN requests 403 before reaching a password prompt.

## Deploy — files NOT in git, create on the VM
| File | Purpose |
|---|---|
| `.env` | `DOMAIN` + `CF_DNS_API_TOKEN` (Bitwarden) |
| `dynamic/local.yml` | real LAN CIDRs — copy from `local.yml.example` |
| `dynamic/.htpasswd` | `htpasswd -Bc dynamic/.htpasswd luke` |
| `acme/acme.json` | `touch` + `chmod 600`; certs land here |

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
