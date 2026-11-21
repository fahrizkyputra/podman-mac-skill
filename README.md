# podman-mac

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![skills.sh](https://img.shields.io/badge/skills.sh-podman--mac-black)](https://skills.sh/fahrizkyputra/podman-mac-skill/podman-mac)

An [agent skill](https://skills.sh) for running [Podman](https://podman.io) on macOS as a lighter replacement for Docker Desktop.

It teaches your coding agent how to control `podman machine`, keep the `docker` CLI pointed at it, recover from connection errors, and back up and restore databases without losing data.

## Why

On a Mac, Podman runs containers inside a Linux VM. That VM keeps running until you stop it, and the `docker` CLI needs to be routed to it. On a machine with limited RAM, an agent that forgets either of these either leaves the VM idling or fails with `Cannot connect to the Docker daemon`. This skill gives the agent the procedure, and a set of guardrails for commands that destroy data.

## Install

```bash
npx skills add fahrizkyputra/podman-mac-skill -g
```

Or copy `skills/podman-mac/` into your agent's skills directory.

To update an installed copy:

```bash
npx skills update -g
```

## Usage

Ask in plain language. The skill activates on requests such as:

| You say | The agent |
|---|---|
| "Start podman" / "Stop podman" | Runs `podman machine start` or `stop` and confirms the VM state |
| "Is podman running?" | Checks the VM, the Docker context and the running containers |
| "docker: Cannot connect to the Docker daemon" | Diagnoses a stopped VM or a stale socket and repairs the context |
| "Run this compose project on podman" | Runs `docker compose up` against Podman and checks the app responds |
| "Back up the MySQL container" | Dumps to SQL and verifies the result |
| "Move my databases from Docker Desktop" | Follows the migration checklist and keeps the old data until verified |

## What it covers

| Area | Details |
|---|---|
| VM control | Start, stop, restart and resize the machine; what survives a stop |
| Connectivity | Docker context for the Podman socket, stale socket after a reboot, the `/var/run/docker.sock` helper |
| Compose | Bind mounts that hide image contents, service names versus `localhost`, rootless port limits |
| Data safety | Commands that destroy data, SQL dumps for MySQL and Postgres, volume archives, restore and verification |
| Migration | Docker Desktop to Podman checklist, including keeping the standalone `docker` CLI working |
| Resources | One engine at a time, stopping the VM when idle |

## Safety

- The agent asks for confirmation and a fresh backup before any destructive command, such as `podman machine rm`, `podman system reset`, volume prunes or `docker compose down -v`.
- It never runs `sudo` for you. Steps that need elevated rights are given to you as commands to run yourself.
- Backup guidance warns that dumps contain real data and credentials and must stay out of version control.

## Requirements

- macOS with Podman installed (for example `brew install podman`)
- Standalone `docker` CLI, with `docker compose` and `docker buildx` if you use them
- An agent that supports the [Agent Skills](https://skills.sh) format

The commands were verified on Apple silicon with Podman 6.2 and the libkrun provider. The skill checks the current state first and adapts when your setup differs.

## Repository layout

```
.
├── skills/
│   └── podman-mac/
│       └── SKILL.md      # the skill
├── LICENSE
└── README.md
```

## Contributing

Issues and pull requests are welcome. Please keep the skill free of personal data: no hostnames, project names, credentials or local paths.

## License

[MIT](LICENSE)
