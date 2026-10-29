# notifiarr-l-swarm

Plex/arr notification aggregator (L = Linux Plex host).

## Stack

- **Host**: ore-dock-110-1
- **Stack**: `notifiarr-l-swarm`
- **Image**: golift/notifiarr
- **Config**: named volume `notifiarr-l-swarm.config`

## Ports

| Port | Published | Protocol |
|------|----------|----------|
| 5454 | 5454 | TCP |

## Client name (hostname)

`hostname=notifiarr-l` in `.env` sets the container's OS hostname. Notifiarr.com
identifies and displays each client by its **hostname**, not by `host_id`
(`host_id` is the hardware identifier). A service created outside a stack
compose gets no hostname, so Docker assigns the container's short ID — which
changes on every task restart, so the site registers a new duplicate client each
time. A static hostname keeps one stable entry.

The value must match the `host_id` the site already has on file for this client
(`notifiarr-l`), otherwise the site treats the new hostname as a different
client. Check it after a change:

```bash
docker logs <container> | grep -E 'new client|Unique Host ID'
# Unique Host ID: notifiarr-l (notifiarr-l)  <- no "new client" line = matched the existing record
```

## Deploy

```bash
ssh ore-dock-110-1
cd /data/compose/notifiarr-l-swarm
set -a && source .env && docker stack deploy -c docker-compose.yml --with-registry-auth notifiarr-l-swarm && set +a
```
