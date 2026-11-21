# podman-mac

An agent skill for running [Podman](https://podman.io) on a Mac as a lighter replacement for Docker Desktop. It tells the agent how to start, stop and check `podman machine`, keep the `docker` CLI pointed at it, fix "Cannot connect to the Docker daemon", and back up and restore databases without losing data.

Works with any agent that supports [skills](https://skills.sh).

## Install

```bash
npx skills add fahrizkyputra/podman-mac-skill -g
```

Or copy `skills/podman-mac/` into `~/.claude/skills/` (or your agent's skills folder).

## Use

Ask in plain language:

- "Start podman" / "Stop podman" / "Is podman running?"
- "docker: Cannot connect to the Docker daemon"
- "Run this compose project on podman"
- "Back up the MySQL container, then restore it into a new volume"
- "I'm moving from Docker Desktop to Podman, what do I need to keep?"

## What it covers

| Area | Content |
|---|---|
| Control | start, stop, restart, resize the VM; what survives a stop |
| Connection | Docker context for the Podman socket, stale socket after reboot, `/var/run/docker.sock` helper |
| Compose | bind mounts hiding image files, service names vs `localhost`, rootless port limits |
| Data safety | commands that destroy data, SQL dumps for MySQL and Postgres, volume tarballs, restore and verify |
| Migration | Docker Desktop to Podman checklist, keeping the `docker` CLI working |
| RAM | one engine at a time, stop the VM when idle |

## Assumptions

Podman from Homebrew, and `docker` / `compose` / `buildx` installed as standalone Homebrew CLIs. The skill checks state first and adapts if yours differs. It never runs `sudo` for you and asks before any destructive command.

## License

MIT
