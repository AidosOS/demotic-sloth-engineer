# Sloth Engineer — Agent Guide

## Role

Sloth Engineer 🦥 est l'ingénieur autonome de la Demotic Suite. Fork d'OpenHands, il exécute des tâches longues en sandbox Docker isolée: lecture complète d'un repo, plan `PLAN.md`, boucle code → tests → auto-correction jusqu'à PR parfaite. Le développeur reste "paresseux" pendant que le paresseux travaille.

Base: `OpenHands-main/` (OpenHands v1.13.0 MIT, `@openhands/agent-canvas`). Ne pas modifier le vendor sans décision produit — le rebrand est branding + glue uniquement en beta.

## Stack

- OpenHands agent loop (Docker sandbox, terminal + FS + browser interne)
- Node >=22.12.0, `@openhands/agent-canvas` (`bin/agent-canvas.mjs`)
- Docker / docker-compose (runtime `docker/runtime`)
- Ports: `8789` par défaut (`DEMOTIC_SLOTH_PORT`), overridable

## Workspace Integration

Partage `~/.demotic/workspaces/<project>/` via `bridge/workspace.mjs`:

```
.sloth/
├── tasks/      # tâches entrantes: <id>.json { id, instruction, repo, status: queued|running|done }
├── sandbox/    # clones éphémères par tâche
└── pr-queue/   # PR candidates prêtes à valider depuis mobile
```

Protocole (cf. `bridge/PROTOCOL.md` §6):

```
clisis-coder --delegate-to sloth "Fix auth bug"
  → writes .sloth/tasks/<uuid>.json
  → Sloth picks, docker sandbox, PLAN.md, loop, writes .sloth/pr-queue/<uuid>.json + .demotic/notifications/<uuid>.json
  → Squid Hub: "🦥 PR ready" | Owl: next task
```

## Commands

| Action | Commande |
|---|---|
| Dev (canvas) | `cd OpenHands-main && npm run dev` (ou `bin/agent-canvas.mjs --port 8789`) |
| Via launcher | `node ../bridge/launch.mjs --only sloth --project my-app` |
| Docker | `docker compose -f docker-compose.yml up` |
| Hand-off depuis Clisis | `clisis-coder --delegate-to sloth "<instruction>"` (écrit `.sloth/tasks/`) |

Package manager: **npm** (OpenHands). Ne pas introduire pnpm/bun ici.

## Rules

- Sandbox obligatoire: jamais d'exécution directe sur le host hors Docker.
- Ne pas toucher au host FS hors `~/.demotic/workspaces/<project>/.sloth/` et `.demotic/notifications/`.
- Toute tâche doit produire `PLAN.md` interne + boucle tests avant PR.
- Notifications via `bridge/workspace.mjs:emitNotification()` (polled) + `readNotifications()`. Le watcher SSE ignore `.demotic/` — ne pas attendre de SSE.
- Coût tokens: proxy via `demotic-cloud` (`lib/cost.ts` marge 30%). BYOK local = 0$ pour l'utilisateur.
- Licence: OpenHands MIT conservée dans `OpenHands-main/` ; wrapper Demotic = BSL 1.1 (2031-07-18 → AGPL). Ne pas re-licencier le vendor.

## Known constraints

- `OpenHands-main/` est un vendor snapshot — `git submodule` non utilisé volontairement en beta (évite 1.1k fichiers en diff).
- Le port 8789 peut confliter avec Motion 8788 / Owl 8787 — vérifier `bridge/PROTOCOL.md` Port Allocation.
- `DEMO_MODE` n'existe pas ici — l'auth passe par `demotic-cloud` (DeviceLink + UserApiKey).
