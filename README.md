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

## Deploy

```bash
ssh ore-dock-110-1
cd /data/compose/notifiarr-l-swarm
set -a && source .env && docker stack deploy -c docker-compose.yml --with-registry-auth notifiarr-l-swarm && set +a
```
