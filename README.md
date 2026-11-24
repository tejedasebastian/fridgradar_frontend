# FridgeRadar — Frontend

UI en **Next.js 16 (App Router) + React 19 + TypeScript + Tailwind 4**, con TanStack Query para fetching, React Hook Form + Zod para formularios, y Zustand para estado local.

- App: <http://localhost:3000>
- Por defecto consume la API en `http://localhost:8000`.

## Estado del proyecto

Este repositorio es un **fork huérfano**: el proyecto upstream original ya no está disponible (la URL devuelve 404), por lo que no hay un remoto del cual sincronizar cambios ni un lugar canónico donde reportar issues. El desarrollo continúa aquí de forma independiente.

- Alcance actual: MVP funcional para uso local / demo (inventario, caducidades, alertas, recetas, lista de compras, hogares compartidos).
- No hay entorno de producción ni deploy automatizado. El CI solo valida lint, tipos y smoke tests.
- El nombre del repositorio, `fridgradar_frontend`, conserva el typo histórico para no romper URLs; el producto se llama **FridgeRadar**.

## Requisitos

- Node.js 20+ (el CI usa Node 22)
- npm 10+
- Backend de FridgeRadar corriendo en local (ver [`fridgeradar_backend`](https://github.com/tejedasebastian/fridgeradar_backend))

## Setup

```powershell
git clone https://github.com/tejedasebastian/fridgradar_frontend.git
cd fridgradar_frontend
npm ci
Copy-Item .env.example .env.local   # opcional, solo si tu API no está en localhost:8000
npm run dev
```

## Variables de entorno

Por defecto el frontend habla contra `http://localhost:8000`. Si tu backend está en otra URL o puerto, copia `.env.example` a `.env.local` y ajusta:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

> Solo hace falta esa variable. Es pública por diseño (`NEXT_PUBLIC_*` queda embebida en el bundle), así que **nunca pongas secretos ahí**.

### Nota de seguridad: tokens en `localStorage`

Tras el login, el `access_token` y el `refresh_token` se guardan en `localStorage` (ver `src/lib/api.ts` y `src/features/auth/infrastructure/`). Es una decisión consciente para este MVP, pero tiene riesgos conocidos:

- Cualquier script que corra en el origen (por ejemplo, una dependencia comprometida o un XSS) puede leer ambos tokens.
- `localStorage` no tiene expiración ni flags `HttpOnly`/`Secure`/`SameSite`; el control es solo del lado del cliente.
- El refresh token de larga duración amplía la ventana de exposición si se filtra.

Mitigaciones actuales: expiración corta del access token, rotación/revocación de refresh tokens en el backend y logout que limpia las claves. Para un despliegue real se recomienda mover la sesión a cookies `HttpOnly` + `Secure` + `SameSite` gestionadas por el backend (o un BFF), y añadir una CSP estricta. Está documentado como deuda técnica, no como un mecanismo seguro para producción.

## Scripts

| Comando | Descripción |
|---|---|
| `npm run dev` | Servidor de desarrollo en <http://localhost:3000> |
| `npm run build` | Build de producción |
| `npm start` | Sirve el build de producción |
| `npm run lint` | Pasa ESLint |
| `npm run typecheck` | `tsc --noEmit` |
| `npm test` | Smoke tests de `tests/` (catálogo local e iconos de categoría) |

## Tests y CI

El workflow [`.github/workflows/ci.yml`](.github/workflows/ci.yml) corre en cada push y pull request a `main`:

1. `npm ci`
2. `npm run lint`
3. `npm run typecheck`
4. `npm test`

Los smoke tests son scripts de Node que leen el código fuente como texto; no necesitan build ni backend. El build de Next.js no se ejecuta en CI por ahora.

Para correr todo en local:

```powershell
npm ci
npm run lint
npm run typecheck
npm test
```

## Estructura

```
fridgradar_frontend/
├── src/
│   ├── app/                  # App Router de Next.js
│   │   ├── (auth)/           # /login, /register
│   │   └── (main)/           # dashboard, inventory, expiry, alerts, recipes, shopping, ...
│   ├── components/           # layout + ui (shadcn-style)
│   ├── features/             # auth, household, inventory, expiry, alerts, recipes, shopping, ai, activity
│   ├── i18n/                 # cadenas en/es
│   └── lib/                  # api, i18n, nav, theme, utils
├── tests/                    # smoke tests de utilidades
├── .github/workflows/        # CI
├── next.config.ts
├── tailwind (PostCSS)
└── package.json
```

## Datos de demo

El backend incluye un seed con usuarios **ficticios**. Tras correr `python scripts/reset_db.py` en el backend, podés entrar con:

- `demo@example.com` / `demo1234`
- `alice@example.com` / `demo1234`
- `bob@example.com` / `demo1234`

> Todos los correos y contraseñas son placeholders inventados para desarrollo. No corresponden a personas reales y no deben reutilizarse en ningún entorno accesible públicamente. También podés registrar una cuenta nueva en `/register`.

## Notas sobre Next.js 16

Este proyecto usa **Next.js 16.2**, que trae cambios incompatibles con versiones anteriores (App Router, convenciones de archivos, comportamiento de `next dev`/`next build`, etc.). Antes de tocar código del framework, consultá `node_modules/next/dist/docs/`.
