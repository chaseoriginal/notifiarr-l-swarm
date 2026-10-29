# notifiarr-l-swarm

Plex/arr notification aggregator (L = Linux Plex host).

## Stack

- **Host**: ore-dock-110-1 (Swarm manager)
- **Stack name**: `notifiarr-l-swarm` (Portainer stack ID 14, endpointId 3)
- **Service**: `notifiarr-l-swarm_notifiarr`
- **Image**: `golift/notifiarr:latest`
- **Config**: named volume `notifiarr-l-swarm_config` (declared `external: true`)
- **Source of truth**: this repo. Portainer is the deployment target, not the source.

## Ports

| Container | Published | Protocol | Mode |
|-----------|-----------|----------|------|
| 5454 | 5454 | TCP | host |

Web UI: `https://notifiarr-l.echoinggreen.net/`

## Client name (hostname)

`hostname=notifiarr-l` in `.env` sets the container's OS hostname. Notifiarr.com
identifies and displays each client by its **hostname**, not by `host_id`
(`host_id` is the hardware identifier). A service created outside a stack
compose gets no hostname, so Docker assigns the container's short ID — which
changes on every task restart, so the site registers a new duplicate client each
time. A static hostname keeps one stable entry.

The site keys a client on the pair (hostname, `host_id`). The `host_id` is pushed
down by notifiarr.com into `/config/notifiarr.conf` — for this instance it is
`notifiarr-l`. **Both values must stay equal to `notifiarr-l`.** If the hostname
drifts from the site's `host_id`, notifiarr.com registers a *second* client
instead of matching the existing one.

Check it after a change:

```bash
docker service logs notifiarr-l-swarm_notifiarr | grep -E 'new client|Unique Host ID'
# Unique Host ID: notifiarr-l (notifiarr-l)  <- no "new client" line = matched the existing record
```

## Placement

Two constraints, both required:

- `node.labels.deployment.target.notifiarr == true` — fleet convention, the same
  label the `notifiarr-w` instance uses. Set on ore-dock-110-1 only.
- `node.hostname==ore-dock-110-1` — named volumes are **node-local** in Swarm. The
  config volume holds the client identity and credentials; if the task rescheduled
  to another node it would mount an empty volume and the client would look like a
  brand-new one.

## Network: `bridge`, not an overlay

This service attaches to the local `bridge` network, declared `external: true`.

The swarm's default address pool on ore-dock-110-1 was `192.168.100.0/24` at size 24,
which yields exactly **one** subnet. Once the host's `docker_gwbridge` held it, any
service needing an **overlay** network failed to start:

```
failed to set up container networking: error creating external connectivity network:
all predefined address pools have been fully subnetted
```

That failure looks like a broken compose file but is host-level pool exhaustion.
The pool was widened on 2026-10-29 (`172.18.0.0/16` added, 256 more chunks), and
overlay stacks now deploy normally on this host.

The `bridge` attachment is kept anyway: it is the shape the live service has always
run with, and it does not consume an address-pool chunk.

## Deploy

The stack is deployed through the Portainer API so Portainer retains stack control.
Git is the source of truth; Portainer is the deployment target.

```bash
# 1. edit docker-compose.yml / .env here, then:
git add -A && git commit -m "..." && git push origin main

# 2. stage onto the swarm manager
scp docker-compose.yml .env ore-dock-110-1:/data/compose/notifiarr-l-swarm/

# 3. push the compose to Portainer (this redeploys the stack)
ssh ore-dock-110-1 'curl -sk -X PUT \
  "https://localhost:9443/api/stacks/14?endpointId=3" \
  -H "X-API-Key: $(cat /tmp/ptr.key)" -H "Content-Type: application/json" \
  -d @/tmp/notifiarr-l-update.json'
```

The payload is `stackFileContent` (the compose text), `env` (the `.env` pairs as
`[{"name":..,"value":..}]`), and `prune: false`.

A PUT is a **redeploy** — it recreates the task, so expect ~15s of downtime.

Do **not** use `docker stack deploy` against a Portainer-managed stack: the CLI and
Portainer then diverge in state, and a later Portainer redeploy silently reverts
to Portainer's stored copy.

## Operations

```bash
docker service ls | grep notifiarr-l          # 1/1 replicas
docker stack ls | grep notifiarr-l            # stack ownership
docker service inspect notifiarr-l-swarm_notifiarr \
  --format '{{index .Spec.Labels "com.docker.stack.namespace"}}'
```

Rollback material lives in `/home/ladmin/backups/notifiarr-l-reset/` on
ore-dock-110-1: the config volume tarball, the stale bind-path tarball, and the
pre-migration service spec.

## Files

- `docker-compose.yml` — the stack definition
- `.env` — deployment variables (not tracked; `.env.example` is the template)
- `.env.example` — committed template
- `README.md` — this file
