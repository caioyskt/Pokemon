# Pokémon RPG — Mesa Principal

Plataforma completa de gerenciamento de campanhas de RPG de Pokémon (baseado nos livros "Biblioteca Élfica"), com autenticação, fichas de treinador, equipes Pokémon e gerenciamento de campanhas.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — API server (port 8080)
- `pnpm --filter @workspace/pokemon-rpg run dev` — frontend (port 22733)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run typecheck:libs` — typecheck composite libs only (faster)
- `pnpm --filter @workspace/db run push` — push DB schema to Postgres
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, Tailwind CSS (dark terminal theme, amber/crimson palette)
- API: Express 5, cookie-parser, openid-client (Replit Auth OIDC)
- Database: PostgreSQL via Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (ESM bundle)

## Where things live

- API spec: `lib/api-spec/openapi.yaml`
- API routes: `artifacts/api-server/src/routes/`
  - `auth.ts` — Replit OIDC login/callback/logout
  - `usuarios.ts` — user role (mestre/jogador)
  - `campanhas.ts` — campaign CRUD
  - `fichas.ts` — character sheets + Pokémon team
  - `pokemon.ts` — in-memory Pokédex
  - `regras.ts` — game rules engine
- Auth lib: `artifacts/api-server/src/lib/auth.ts`
- Auth middleware: `artifacts/api-server/src/middlewares/authMiddleware.ts`
- DB schemas: `lib/db/src/schema/auth.ts`, `lib/db/src/schema/game.ts`
- Auth hook: `lib/replit-auth-web/src/use-auth.ts`
- Frontend pages: `artifacts/pokemon-rpg/src/pages/`

## DB Schema Tables

- `sessions` — Replit Auth session store (mandatory)
- `users` — Replit Auth user records (mandatory)
- `user_profiles` — role: mestre | jogador
- `campaigns` — GM campaigns with invite codes
- `campaign_members` — player ↔ campaign membership
- `character_sheets` — full treinador sheet (6 attrs, level, classes, Pokédolares, insígnias)
- `pokemon_team` — up to 6 Pokémon per character sheet

## Game Rules

- Progression table: 66 attrs / 2 talents at level 0 → 106 attrs / 32 talents at level 50
- Trava do dobro: no attribute can exceed 2× the lowest attribute
- PV = Saúde × 4 + Nível × 4
- MOD = floor((attr - 10) / 2)
- Classes: 1ª (lv 0), 2ª (lv 5+), 3ª (lv 12+), 4ª (lv 24+)
- 6 attributes: Saúde, Ataque, Defesa, Atq. Especial, Def. Especial, Velocidade

## Auth Flow

1. User hits `/api/login` → redirected to Replit OIDC
2. Callback at `/api/callback` → session created, user upserted in DB
3. Frontend `useAuth()` hook fetches `/api/auth/user` on load
4. If no role set → `RoleSelection` page shown
5. Role → Mestre dashboard or Jogador dashboard

## Roles

- **Mestre**: Create/manage campaigns, generate invite codes, view player sheets
- **Jogador**: Create character sheets, join campaigns via code, manage Pokémon team

## Gotchas

- After running codegen, manually fix `lib/api-zod/src/index.ts` to only contain `export * from "./generated/api";`
- The API server must be restarted after any route changes (it builds with esbuild)
- Port 22733 for frontend, 8080 for API (proxied through Replit at localhost:80)
- `lib/replit-auth-web` is composite — needs `tsc --build` before frontend can import it cleanly
- DB schema changes require `pnpm --filter @workspace/db run push`

## Product Pages

- **/** — Role-gated home (Mestre → campaign dashboard; Jogador → character/campaign overview)
- **/campanha/:id** — Campaign detail with invite code and player list
- **/ficha/:id** — Full character sheet with attribute editor, class selector, Pokémon team management
- **/pokédex** — In-memory Pokédex with stats and TN
- **/auditoria** — Attribute audit tool (rule validator)
- **/combate** — Combat damage calculator
- **/captura** — Capture rate calculator
