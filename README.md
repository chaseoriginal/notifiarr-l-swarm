# notifiarr-l-swarm

Portainer-managed Docker Swarm stack. **Git is the source of truth**; Portainer
owns the deployment and polls this repository for changes.

| | |
|---|---|
| Portainer stack id | 14 |
| Endpoint | local (id 3) |
| Swarm | iymtakl6ibno |
| Host | ore-dock-110-1 |
| Git repo | https://github.com/chaseoriginal/notifiarr-l-swarm |
| Compose file | `docker-compose.yml` |
| Auto-update poll | every 900s |

## Files

- `docker-compose.yml` - the deployed compose. This is byte-identical to what Portainer
  deploys; it is read back from Portainer's own stack-file API so the two can
  never drift.
- `.env.example` - the variable names Portainer substitutes, with values
  redacted. The real values live in Portainer's stack environment and are
  deliberately NOT committed.

## How a change is deployed

1. Edit `docker-compose.yml` in this repo and push to `main`.
2. Portainer polls the repo and redeploys automatically (no manual step).

To force an immediate pull, use the stack's **Refresh** action in the Portainer
UI, or redeploy it from the stack page.

## Verifying a deployment

```bash
docker service ls --filter name=notifiarr-l-swarm
docker service inspect notifiarr-l-swarm_<service> --format '{{.Spec.TaskTemplate.Networks}}'
```

A healthy stack shows `1/1` and a task with no error state.
