# AI agent instructions

When a user asks to map, diagram, discover, or document the current machine or
homelab, read `AI_AUTOMAP.md` and `CLAUDE.md` completely, then follow the
autonomous workflow in `AI_AUTOMAP.md`.

The default task is discovery and documentation, not infrastructure changes.
Do not start, stop, restart, install, remove, or reconfigure services; do not
change Docker, networking, firewall, storage, or system settings. Repository
files and project dependencies may be changed as described in `AI_AUTOMAP.md`.

If the user only asks to develop or review this repository, use the relevant
engineering instructions in `CLAUDE.md` without scanning the host.
