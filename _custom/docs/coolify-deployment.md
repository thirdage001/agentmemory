# Coolify-Deployment — agentmemory Produktionsinstanz

Diese Datei dokumentiert die **reale** Coolify-Konfiguration der agentmemory-
Produktionsinstanz, ausgelesen via Coolify MCP am 2026-09-01. Sie dient der
Reproduzierbarkeit beim Setup auf einem anderen Computer bzw. einer neuen
Coolify-Instanz.

> **Secrets:** Alle echten Secrets (`OPENAI_API_KEY`, Webhook-Secrets, HMAC-
> Secret) sind hier **nicht** enthalten — sie stehen als `<...>` Platzhalter.
> Echte Werte liegen nur in der Coolify-UI (Environment Variables) bzw. werden
> zur Laufzeit generiert. Siehe [Secrets-Verwaltung](#secrets-verwaltung).

---

## Übersicht

| Feld | Wert |
|---|---|
| **Öffentliche URL** | `https://agentmemory.kopps.net` |
| **Coolify-Version** | `4.3.14` |
| **Status** | `running:healthy` |
| **Build Pack** | `dockercompose` (Compose Parsing v5) |
| **Git-Repo** | `thirdage001/agentmemory` (GitHub), Branch `main` |
| **Erstellt** | 2026-08-27 |
| **Zuletzt aktualisiert** | 2026-09-01 |

---

## Server (Coolify-Host)

Der agentmemory-Container läuft auf dem Coolify-Host selbst (kein separater
Server).

| Feld | Wert |
|---|---|
| Name | `localhost` |
| UUID | `kkkokik9jdtypqfq84rp8gun` |
| IP (intern) | `host.docker.internal` |
| Öffentliche IPv4 | `185.207.250.150` |
| Öffentliche IPv6 | `2a02:c207:3018:9083::1` |
| SSH | Port `22`, User `root` |
| Docker | `29.4.0` |
| Timezone | `UTC` |
| `is_coolify_host` | `true` |
| `concurrent_builds` | `2` |
| `dynamic_timeout` | `3600` |
| `deployment_queue_limit` | `25` |
| `force_docker_cleanup` | `true` (Threshold 80 %, täglich 0:00 UTC) |

### Reverse Proxy (Traefik)

| Feld | Wert |
|---|---|
| Typ | `TRAEFIK` |
| Image | `traefik:v3.6` (erkannt `3.6.13`; latest `3.6.23`) |
| Status | `running` |
| HTTP-Redirect → HTTPS | `true` |
| TLS | Let's Encrypt, ACME HTTP-01 Challenge |
| Entrypoints | `:80` (http), `:443` (https, HTTP/3), `:8080` (api) |
| Netzwerk | `coolify` (external) |
| Config-Verzeichnis | `/data/coolify/proxy/` → `/traefik` im Container |

### Docker-Network (Destination)

| Feld | Wert |
|---|---|
| Name | `coolify` |
| UUID | `yh1cpw1li2z6jpf6gjoya1go` |
| Server | `localhost` (id 0) |

---

## Projekt & Environment

| Feld | Wert |
|---|---|
| Team | `Root Team` (id 0, personal team) |
| Projekt | `My first project` (UUID `d1m4uutouqobxauz31f8us87`, id 1) |
| Environment | `production` (UUID `adxitipi421zbxotmg9qx8hu`, id 1) |

---

## Application

| Feld | Wert |
|---|---|
| Name | `agentmemory` |
| UUID | `nbazoikzdq35bggev4ep9qu1` (id 4) |
| Beschreibung | `agentmemory deployed via Coolify Docker Compose template` |
| Build Pack | `dockercompose` |
| Compose Parsing Version | `5` |
| Base Directory | `/` |
| Compose Path | `/docker-compose.yml` (Repo-Root) |
| Git Repository | `thirdage001/agentmemory` |
| Git Branch | `main` |
| Git Commit SHA | `HEAD` (folgt immer main) |
| Source Type | `GithubApp` |
| Container Name | `agentmemory-nbazoikzdq35bggev4ep9qu1-120431331599` |
| Config Hash | `7206acf029deda59a1b34755e92068ddcb95e121eba752a4e9876abaf8dfdc8d` |

### Domain & Ports

| Feld | Wert |
|---|---|
| FQDN (Compose-Domain) | `https://agentmemory.kopps.net:3111` |
| `SERVICE_FQDN_AGENTMEMORY` | `agentmemory.kopps.net` |
| `ports_exposes` | `3111` (nur Proxy-Netzwerk, nicht auf Host gebunden) |
| `ports_mappings` | `3113:3113` → gebunden auf `127.0.0.1:3113:3113` (Viewer, nur lokal) |
| Redirect | `both` (HTTP→HTTPS + www↔non-www) |
| Force HTTPS | `true` |
| Gzip | `true` |
| StripPrefix | `true` |
| HTTP Basic Auth | deaktiviert |

### Application-Settings

| Setting | Wert |
|---|---|
| Auto-Deploy | `true` (Push auf `main` triggert Deploy) |
| Preview Deployments | `false` |
| Log Drain | `false` |
| GPU | `false` |
| Shallow Clone | `true` |
| Build Cache | aktiv (`disable_build_cache: false`) |
| Docker Images to Keep | `2` |
| Inject Build Args to Dockerfile | `true` |
| Raw Compose Deployment | `false` |
| Consistent Container Name | `false` |
| Swarm | nicht aktiv (worker-only flag `true`, aber kein Swarm) |

### Healthcheck

Coolify's eigener Healthcheck ist **deaktiviert** (`health_check_enabled:
false`). Der Healthcheck läuft über die `HEALTHCHECK`-Direktive im Compose-File:

```yaml
healthcheck:
  test: ["CMD-SHELL", "curl -fsS http://127.0.0.1:3111/agentmemory/livez || exit 1"]
  interval: 30s
  timeout: 5s
  start_period: 30s
  retries: 3
```

### Resource-Limits

| Feld | Wert |
|---|---|
| CPU Shares | `1024` |
| CPUs | `0` (unbegrenzt) |
| Memory | `0` (unbegrenzt) |
| Memory Swap | `0` (unbegrenzt) |
| Swappiness | `60` |
| Max Restart Count | `10` |

---

## Docker Compose (effektiv deployed)

Die von Coolify generierte Compose-Datei (mit aufgelösten Variablen):

```yaml
services:
  agentmemory:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        AGENTMEMORY_VERSION: "0.9.29"
        III_VERSION: "0.11.2"
        III_SDK_VERSION: "0.11.2"
    restart: unless-stopped
    environment:
      AGENTMEMORY_VIEWER_HOST: "0.0.0.0"
      VIEWER_ALLOWED_HOSTS: "localhost:3113,127.0.0.1:3113,[::1]:3113"
      AGENTMEMORY_VERSION: "0.9.29"
      III_VERSION: "0.11.2"
      III_SDK_VERSION: "0.11.2"
      SERVICE_URL_AGENTMEMORY: "https://agentmemory.kopps.net"
      SERVICE_FQDN_AGENTMEMORY: "agentmemory.kopps.net"
      COOLIFY_URL: "https://agentmemory.kopps.net"
      COOLIFY_FQDN: "agentmemory.kopps.net"
      SERVICE_NAME_AGENTMEMORY: "agentmemory"
    expose:
      - "3111"
    ports:
      - "127.0.0.1:3113:3113"
    volumes:
      - "nbazoikzdq35bggev4ep9qu1_agentmemory-data:/data"
    healthcheck:
      test: ["CMD-SHELL", "curl -fsS http://127.0.0.1:3111/agentmemory/livez || exit 1"]
      interval: 30s
      timeout: 5s
      start_period: 30s
      retries: 3
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

> Die Quelldatei im Repo ist `deploy/coolify/docker-compose.yml` (bzw. Root
> `docker-compose.yml`). Coolify injiziert automatisch `SERVICE_FQDN_*`,
> `COOLIFY_*`, `container_name`, `labels` und das Volume-Prefix.

### Build Args

| Arg | Wert | Zweck |
|---|---|---|
| `AGENTMEMORY_VERSION` | `0.9.29` | agentmemory npm-Package-Version |
| `III_VERSION` | `0.11.2` | iii-engine Binary-Version |
| `III_SDK_VERSION` | `0.11.2` | iii-sdk npm-Package-Version |

---

## Persistenter Storage

| Feld | Wert |
|---|---|
| Volume-Name | `nbazoikzdq35bggev4ep9qu1_agentmemory-data` |
| Mount Path | `/data` |
| Storage UUID | `ysdn5fjwjafprobrojjfkcix` |
| Host Path | (Docker named volume, kein bind mount) |
| Preview Suffix | aktiviert (`is_preview_suffix_enabled: true`) |

Das Volume enthält: SQLite State Store, BM25-Index, Stream-Backlog und die
HMAC-Secret-Datei (`/data/.hmac`). **Nicht löschen** — sonst gehen alle
Memories und das HMAC-Secret verloren.

Host-Pfad (für Backups):
```
/var/lib/docker/volumes/nbazoikzdq35bggev4ep9qu1_agentmemory-data/_data
```

---

## Environment Variables

### Selbst konfiguriert (Production, `is_coolify: false`)

Diese Variablen wurden manuell in der Coolify-UI gesetzt und aktivieren die
erweiterten agentmemory-Features:

| Key | Wert | Zweck |
|---|---|---|
| `AGENTMEMORY_SLOTS` | `true` | `memory_slot_*` Tools — persistente Context-Slots |
| `AGENTMEMORY_REFLECT` | `true` | `memory_reflect` — LLM-synthetisierte Insights (erfordert `SLOTS=true`) |
| `CONSOLIDATION_ENABLED` | `true` | `memory_consolidate` — 4-Tier-Pipeline (working→episodic→semantic→procedural) |
| `OPENAI_MODEL` | `gpt-4o-mini` | LLM-Modell für Consolidation (max_tokens-kompatibel, vermeidet 400-Fehler bei gpt-5.x/o1/o3) |
| `OPENAI_API_KEY` | `<OPENAI_API_KEY>` | OpenAI API-Key für Consolidation/Reflect — **Secret, siehe unten** |
| `AGENTMEMORY_INJECT_CONTEXT` | `true` | Automatische Context-Injection in Session-Start |

### Coolify-verwaltet (Production, `is_coolify: true`, auto-generiert)

Diese Variablen werden von Coolify aus der Domain-Konfiguration abgeleitet und
sollten **nicht** manuell editiert werden:

| Key | Wert |
|---|---|
| `SERVICE_FQDN_AGENTMEMORY` | `agentmemory.kopps.net` |
| `SERVICE_URL_AGENTMEMORY` | `https://agentmemory.kopps.net` |
| `SERVICE_FQDN_AGENTMEMORY_3111` | `agentmemory.kopps.net:3111` |
| `SERVICE_URL_AGENTMEMORY_3111` | `https://agentmemory.kopps.net:3111` |

### Compose-injiziert (aus docker-compose.yml `environment:`)

| Key | Wert |
|---|---|
| `AGENTMEMORY_VIEWER_HOST` | `0.0.0.0` |
| `VIEWER_ALLOWED_HOSTS` | `localhost:3113,127.0.0.1:3113,[::1]:3113` |

### Preview-Env-Variablen (auto-generiert, Preview deaktiviert)

Preview-Deployments sind deaktiviert (`is_preview_deployments_enabled: false`),
aber Coolify hat Preview-Env-Variablen mit der sslip.io-Fallback-Domain
angelegt. Diese sind inaktiv und können ignoriert werden:

- `SERVICE_FQDN_AGENTMEMORY` (preview) = `nbazoikzdq35bggev4ep9qu1.185.207.250.150.sslip.io`
- weitere Preview-Duplikate der obigen Variablen

---

## Secrets-Verwaltung

Folgende Secrets werden **nicht** in dieser Datei gespeichert. Sie müssen auf
der neuen Instanz neu gesetzt bzw. generiert werden:

| Secret | Wo gesetzt | Hinweis |
|---|---|---|
| `OPENAI_API_KEY` | Coolify UI → Application → Environment Variables | Production-Key (`sk-proj-...`). Wird für Consolidation + Reflect gebraucht. |
| `AGENTMEMORY_SECRET` (HMAC) | **nicht in Env-Vars** — wird beim ersten Start generiert und in `/data/.hmac` gespeichert | Einmalig in den Container-Logs beim ersten Boot abgreifen (`grep -i secret`). Wird nie wieder ausgegeben. Rotation: `/data/.hmac` löschen + Redeploy. |
| `manual_webhook_secret_github` | Coolify auto-generiert | Für Git-Push-Webhook. Steht in den Application-Details. |
| `manual_webhook_secret_gitea` / `_gitlab` / `_bitbucket` | Coolify auto-generiert | Analoge Webhook-Secrets für andere Git-Provider. |
| Coolify API-Token | Coolify UI → API Tokens | Für MCP-Zugriff auf Coolify. In `mcp-configs/devin-mcp-config.json` als `<COOLIFY_TOKEN>` Platzhalter. |

---

## Reproduktion auf einer neuen Coolify-Instanz

1. **Coolify-Instanz bereitstellen** (VPS mit Docker, Coolify installiert).
2. **Neue Application erstellen** — Build Pack `dockercompose`, Git-Repo
   `thirdage001/agentmemory`, Branch `main`, Base Directory `/`, Compose Path
   `/docker-compose.yml`.
3. **Domain setzen**: `https://<eigene-domain>:3111` (Port-Suffix wichtig für
   Traefik-Routing).
4. **Environment Variables setzen** (siehe Tabelle oben, selbst konfiguriert):
   - `AGENTMEMORY_SLOTS=true`
   - `AGENTMEMORY_REFLECT=true`
   - `CONSOLIDATION_ENABLED=true`
   - `OPENAI_MODEL=gpt-4o-mini`
   - `OPENAI_API_KEY=<echter-key>`
   - `AGENTMEMORY_INJECT_CONTEXT=true`
5. **Deploy** — erster Build dauert ~2 min.
6. **HMAC-Secret aus Logs holen**: `docker logs <container> 2>&1 | grep -i
   secret` → als `AGENTMEMORY_SECRET` in alle Client-MCP-Configs eintragen.
7. **DNS**: A/AAAA-Record für `agentmemory.<domain>` → Server-IP
   (`185.207.250.150` / `2a02:c207:3018:9083::1` bei dieser Instanz).
8. **Backup-Strategie** für das Docker-Volume einrichten (Coolify Backups
   Feature oder host-level Snapshots).

### Client-seitige Konfiguration

Nach dem Deploy müssen auf jedem Client-Rechner gesetzt werden:

- `AGENTMEMORY_URL` = `https://agentmemory.kopps.net` (oder neue Domain)
- `AGENTMEMORY_SECRET` = `<hmac-secret aus Logs>`
- MCP-Server-Block in allen Agent-Configs (siehe `mcp-configs/` und
  `docs/setup-guide.md` Step 5)

---

## Verifikation

```bash
# Healthcheck
curl https://agentmemory.kopps.net/agentmemory/livez
# → {"status":"ok"}

# Authentifizierter Call
curl -H "Authorization: Bearer <AGENTMEMORY_SECRET>" \
     https://agentmemory.kopps.net/agentmemory/sessions
```

---

## Bekannte Besonderheiten

- **`OPENAI_MODEL=gpt-4o-mini`**: Neuere OpenAI-Modelle (`gpt-5.x`, `o1`, `o3`,
  `o4-mini`) lehnen den legacy `max_tokens`-Parameter ab und verursachen
  `Summarize failed` 400-Fehler bei der Consolidation. `gpt-4o-mini` ist
  kompatibel, bis der `max_completion_tokens`-Fix in einer veröffentlichten
  npm-Version shipped.
- **Viewer-Port 3113** ist nur auf `127.0.0.1` gebunden — erreichbar via
  SSH-Tunnel (`ssh -L 3113:127.0.0.1:3113 root@<host>`), nicht öffentlich.
- **Base Directory `/`**: Die App nutzt das Root-Compose-File, nicht
  `deploy/coolify/` (obwohl der deploy-Ordner ein eigenes Compose-File
  enthält). Beide Dateien sind inhaltlich identisch.
- **Traefik-Version**: `3.6.13` installiert, `3.6.23` verfügbar (Patch-Update).
  Ein v3.7-Branch (`3.7.8`) existiert ebenfalls.
- **Auto-Deploy aktiv**: Jeder Push auf `main` triggert automatisch einen
  Rebuild. Zum Sperren: `is_auto_deploy_enabled` in den App-Settings
  deaktivieren.
