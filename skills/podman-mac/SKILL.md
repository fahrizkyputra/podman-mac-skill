---
name: podman-mac
description: >-
  Use when the user runs Podman on a Mac in place of Docker Desktop and wants to start, stop, check or fix it — "start podman", "stop podman", "is podman running", "docker: Cannot connect to the Docker daemon", "podman machine fails", "docker compose up on podman", "back up or restore a database in podman", or any docker/podman container task on macOS where the engine is `podman machine`.
---

# Podman on macOS

Podman on a Mac runs containers inside one Linux VM, `podman machine`. There is no daemon on the host and no GUI app unless the user installed Podman Desktop. This skill covers day-to-day control of that VM, keeping the `docker` CLI pointed at it, and not losing data.

Assumed setup: Podman from Homebrew, `docker` / `docker compose` / `docker buildx` as separate Homebrew CLIs, Docker context named `podman`. If the user's setup differs, check first and adapt.

## State check (always first)

```bash
podman machine list          # is the VM "Currently running"?
docker context show          # should print: podman
docker ps -a --format '{{.Names}} {{.Status}} {{.Ports}}'
```

## Start / stop

| Intent | Command | Done when |
|---|---|---|
| Start | `podman machine start` | `podman machine list` shows `Currently running`, `docker ps` works |
| Stop | `podman machine stop` | list shows stopped; all containers stopped, RAM freed, volumes kept |
| Restart | `podman machine stop && podman machine start` | as above |
| Stop containers, keep VM | `docker stop $(docker ps -q)` | `docker ps` empty |

- The VM does not sleep on its own and does not start after a reboot. Start it again after a restart.
- Containers with `restart: always` or `unless-stopped` come back on `podman machine start`.
- Before stopping the VM, tell the user which containers are running (`docker ps`). Stopping does not delete volumes.
- Resize: `podman machine stop && podman machine set --cpus 2 --memory 2048`.

## Connection problems

`Cannot connect to the Docker daemon`: the VM is stopped, so run `podman machine start`. If it still fails, the context socket path may be stale (it lives under `$TMPDIR` and can change after a reboot):

```bash
docker context ls
docker context rm podman 2>/dev/null
docker context create podman --docker "host=unix://$(podman machine inspect --format '{{.ConnectionInfo.PodmanSocket.Path}}')"
docker context use podman
```

To make `/var/run/docker.sock` point at Podman, the user must run `sudo podman-mac-helper install` themselves, then `podman machine stop && podman machine start`. Do not run sudo for them.

## Compose and rootless notes

- `docker compose -p <project> up -d --build` from the project folder works against Podman.
- A bind mount such as `.:/var/www/html` hides files the image built into that path (for example PHP `vendor/` or Node `node_modules/`). If they are missing on the host, install them inside the container.
- Inside compose, a service reaches another by service name, not `localhost`. Override env per command (`docker exec -e DB_HOST=db ...`) instead of editing the user's `.env`.
- Rootless Podman cannot publish host ports below 1024. Use 8080, 3306 and similar.
- Podman Desktop and Docker Desktop both claim `/var/run/docker.sock`. Run only one.

## Data safety

Database and volume data live on the VM disk. Never run these without explicit user confirmation and a fresh backup, because they destroy data irreversibly:

`podman machine rm`, `podman system reset`, `podman volume prune`, `docker volume prune`, `docker system prune --volumes`, `docker compose down -v`.

Back up databases as SQL dumps; they are portable across versions:

```bash
# MySQL / MariaDB
docker exec <ctr> sh -c 'mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" --all-databases --routines --triggers --events --single-transaction' > backup.sql
# Postgres
docker exec <ctr> sh -c 'PGPASSWORD="$POSTGRES_PASSWORD" pg_dumpall -U "$POSTGRES_USER"' > backup.sql
# Any volume, raw
docker run --rm -v <volume>:/data:ro -v "$PWD":/backup alpine tar czf /backup/<volume>.tgz -C /data .
```

Backups contain real data and credentials. Keep them out of git and out of public places.

Restore: start an empty container with the target volume, **wait for `ready for connections` in `docker logs`** (an immediate ping gives a false "Access denied"), then pipe the `.sql` in. Verify with table counts against the dump and `COUNT(*)` row counts. MySQL's `information_schema.table_rows` is cached and often shows 0, so do not trust it.

Migrating from Docker Desktop: Podman cannot read Docker Desktop's `Docker.raw`. Dump and restore as above, and keep Docker Desktop installed until the restored data is verified. Standalone `docker`, `compose` and `buildx` CLIs must be installed (for example `brew install docker docker-compose docker-buildx`) before removing `Docker.app`, because the app bundles them. Also fix `~/.docker/config.json` (`currentContext`, `credsStore`) and any PATH line pointing into `Docker.app`.

## RAM discipline

- Run only the containers the task needs.
- Do not run two container engines at once on a small Mac; two VMs make it swap.
- Stop the VM when the task ends and say so. Check pressure with `sysctl vm.swapusage`.

## Verification

Report from output, not assumption: `podman machine list` for VM state, `docker ps` for containers, `curl -s -o /dev/null -w '%{http_code}' http://localhost:<port>` for apps. Say "not verified live" for anything not checked.

## Optional: auto-start at login

Off by default to save RAM. Upstream describes a launchd agent in `docs/tutorials/macos_autostart.md` of the Podman repository; it needs `AbandonProcessGroup` set to `true` or launchd kills the VM when `podman machine start` exits.
