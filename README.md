# 🦥 Sloth Engineer — Autonomous DevOps Agent

Fork d'OpenHands pour la Demotic Suite. Agent autonome en sandbox Docker qui prend une tâche complexe (bug, migration, upgrade deps) et boucle jusqu'à PR parfaite. Le dev reste paresseux pendant que le paresseux bosse.

- **Accent:** `#A16207` (Écorce) — force tranquille, travail de fond
- **Port:** `8789` (`DEMOTIC_SLOTH_PORT`)
- **Runtime:** `OpenHands-main/` (`@openhands/agent-canvas` v1.13.0 MIT)
- **Workspace:** `~/.demotic/workspaces/<project>/.sloth/{tasks,sandbox,pr-queue}`
- **Licence:** BSL 1.1 (wrapper) + MIT (vendor)

## Quick start

```bash
# Standalone
node OpenHands-main/bin/agent-canvas.mjs --port 8789

# Via Demotic launcher
node ../bridge/launch.mjs --only sloth --project demo
node ../bridge/launch.mjs --only sloth,watcher

# Docker
docker compose up
```

## Hand-off

```bash
clisis-coder --delegate-to sloth "Optimise toute la couche sécurité"
# → écrit .sloth/tasks/<id>.json → Sloth pick → Docker → PR → notification → Squid "swipe to merge"
```

Voir `AGENTS.md` et `bridge/PROTOCOL.md` §6.
