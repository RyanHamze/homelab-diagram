# Autonomous environment mapping

This is the reusable runbook for an AI coding agent with terminal, file-editing,
and image-viewing capabilities. Its goal is to turn the environment visible
from the current Ubuntu host into a new, sanitized homelab diagram with minimal
user input.

## Invocation

The user can start the workflow with this single prompt:

> Read AGENTS.md, AI_AUTOMAP.md, and CLAUDE.md completely. Map the current
> environment and produce the final sanitized diagram. Follow the autonomous
> discovery workflow, use read-only system inspection, do not change running
> infrastructure, and continue through render and visual validation without
> waiting for routine confirmation.

Treat that prompt as authorization to run the read-only commands listed below
on the current host and to edit files inside this repository. It is not
authorization to elevate privileges, scan arbitrary remote networks, access
unrelated private directories, or change the system. Ask only when credentials,
privilege, a remote scan, or a decision that materially changes scope is needed.

## Required results

Produce all of the following:

- `out/homelab.svg` and `out/homelab.png`: sanitized outputs safe for review.
- `out/discovery.json`: structured facts and provenance, with secrets removed.
- `out/discovery-report.md`: concise verified facts, inferences, and limitations.
- `out/unknowns.md`: information that could not be established safely.

Keep unredacted outputs local and generate them only when the user explicitly
requests them. Never commit generated inventories or outputs without an
explicit request after a privacy review.

## Safety boundary

Discovery is read-only. Never:

- start, stop, restart, create, remove, or reconfigure containers or services;
- modify interfaces, bridges, routes, DNS, firewall rules, VLANs, mounts,
  storage, permissions, users, groups, scheduled jobs, or boot configuration;
- use `sudo`, the Docker socket, SSH, SNMP, an API, or a network scan unless it
  is already accessible to the current user and within the user's stated scope;
- read or reproduce passwords, tokens, cookies, private keys, complete container
  environment values, or secret files;
- probe public addresses or subnets that the user has not explicitly approved.

Docker read access is powerful and may expose secrets. Prefer metadata-specific
formats and never dump complete `docker inspect` output or container environment
variables into reports. Record secret-bearing fields as `[redacted]` or omit them.

Installing repository-only Python dependencies, fetching the repository's icon
set, editing repository files, and rendering images are allowed. If a required
system package is missing, report the exact package and continue as far as
possible; do not install it without approval.

## Autonomous workflow

### 1. Establish scope and preserve the example

Work from the repository root. Read `README.md`, `CLAUDE.md`, this file, the
inventory schema, and the renderer before editing. Preserve the shipped example
on the first run:

```bash
cp -n inventory.yaml inventory.example.yaml
```

Do not overwrite an existing user backup. Inspect the Git working tree and
preserve unrelated changes.

### 2. Collect local facts

Run the useful commands that exist and are permitted; a missing tool or denied
permission is an unknown, not a reason to weaken the safety boundary.

Host and operating system:

```bash
hostnamectl
cat /etc/os-release
uname -a
lscpu
free -h
lsblk -J -o NAME,MODEL,SERIAL,SIZE,TYPE,FSTYPE,MOUNTPOINTS
findmnt --json
lspci
```

Networking and local services:

```bash
ip -json address show
ip -json route show table all
ip -details -json link show
bridge -json link show
bridge -json vlan show
ss -H -lntup
resolvectl status
```

Docker, when available to the current user:

```bash
docker version
docker info --format '{{json .}}'
docker ps -a --no-trunc --format '{{json .}}'
docker network ls --format '{{json .}}'
docker volume ls --format '{{json .}}'
docker compose ls --all --format json
```

For each discovered container or network, query only the fields needed for the
map: name, image, state, health, labels used for Compose grouping, network/IP,
published ports, and mount source type/destination. Do not collect `Config.Env`
or secret/config payloads. Read Compose files only when their paths are exposed
by Compose labels and the current user can read them; redact environment values.

Optionally collect locally available virtualization and storage facts with
read-only commands such as `virsh list --all`, `pct list`, `qm list`,
`zpool list -v`, and `zfs list`. Do not treat their absence as an error.

### 3. Normalize and classify evidence

Write `out/discovery.json` with a top-level schema version, timestamp, host,
interfaces, routes, networks, listeners, storage, virtualization, containers,
services, relationships, and sources. Every relationship must be classified as:

- `verified`: directly supported by local metadata;
- `inferred`: a defensible conclusion from names, ports, mounts, labels, or
  shared networks; or
- `unknown`: unavailable from this observation point.

Never invent physical cabling, upstream switch ports, VLAN membership, router
rules, external DNS, or application dependencies. A Docker bridge is not a
physical VLAN. A published port does not prove internet exposure.

### 4. Build a new inventory and layout

Replace the example content in `inventory.yaml` with the discovered environment.
Use neutral placeholders for unknown values and keep public redactions enabled.
Represent the Ubuntu host, its physical interfaces, Docker/virtual bridges,
Compose projects, containers, published ports, persistent storage, and verified
connections. Clearly distinguish inferred connections in the supporting report.

Adapt `build.py` to this environment. The existing coordinates describe the
example lab and are expected to change. Remove panels that do not apply rather
than filling them with invented data. Follow the visual rules and renderer notes
in `CLAUDE.md`.

### 5. Render, inspect, and iterate

Use the pinned Python environment when available:

```bash
uv sync
uv run fetch_icons.py
uv run build.py
```

Otherwise use the documented Python commands if dependencies already exist.
Open `out/homelab.png` with the agent's image-viewing capability. Inspect the
whole image and crops of dense areas. Fix overlaps, clipping, unreadable text,
crossing cables, missing labels, broken icons, and excessive empty space. Render
and inspect again until the image is readable. Do not declare completion based
only on a successful process exit.

### 6. Privacy and correctness gate

Before completion, search the SVG, reports, and inventory for values that should
not leave the machine: real domains, SSIDs, public IPs, usernames, home paths,
serial numbers, tokens, keys, and environment values. Redact them in public
outputs. Confirm that every rendered connection is verified or clearly explained
as inferred. Put missing router, switch, cabling, VLAN, and remote-host facts in
`out/unknowns.md`.

Finish by reporting the output paths, the discovery coverage, major inferences,
unknowns, privacy redactions, and any dependency that prevented a complete PNG.

## Extending beyond the current host

The current host cannot reveal an entire routed or physical network by itself.
If the user wants deeper discovery, present a separate read-only expansion plan
for explicitly selected devices or subnets. State the exact protocol and scope
before requesting access—for example, SNMPv3/LLDP for managed switches, a
read-only controller API for UniFi, or read-only Proxmox/TrueNAS credentials.
Never reuse credentials found incidentally on disk.
