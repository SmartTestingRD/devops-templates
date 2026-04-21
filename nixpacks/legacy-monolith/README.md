# legacy-monolith

Template Nixpacks para aplicaciones "monolito legacy" donde un backend Node
(Express u otro) sirve un frontend SPA ya construido desde su propio
`frontend/dist/`, exponiendo todo bajo un único dominio.

## Cuándo usar este template

- El repositorio tiene el backend en la raíz y un frontend en `frontend/`.
- El backend sirve los estáticos de `frontend/dist/` (SPA) además del API.
- Un solo dominio responde tanto a las rutas del API (p.ej. `/api/*`) como
  a las rutas del SPA (`/*`), eventualmente con rutas API duplicadas en `/*`
  como workaround de un proxy (Traefik u otro).
- No quieres/puedes refactorizar el repo para separar frontend y backend.

Si tu proyecto ya está separado en dos servicios, prefiere
`vite-ts` + `express` (o `react` + `express`) como despliegues independientes.

## Convenciones que asume (estructura mínima del repo)

```
repo/
├── package.json            # backend (dependencias + scripts del API)
├── package-lock.json
├── src/
│   └── server.js           # entry point del backend
└── frontend/
    ├── package.json        # frontend (Vite/CRA/etc.)
    ├── package-lock.json
    └── dist/               # generado en el build; servido estáticamente por el backend
```

- Node 20, alineado con los Dockerfiles y `engines` del repo piloto.
- `npm ci` con fallback a `npm install` para tolerar lockfiles desactualizados
  sin romper el build (tanto en la raíz como en `frontend/`).
- El frontend se construye con `npm run build --prefix frontend`.
- Entry point del backend fijado por convención en `src/server.js`.

## Variables de entorno requeridas en Coolify

- `PORT` — puerto en el que el backend escucha. Coolify lo inyecta; el código
  del backend debe leerlo (`process.env.PORT`).
- `VITE_API_URL` — URL base del API consumida por el frontend. **No** se
  hardcodea en el build: se inyecta como variable de entorno de Coolify
  marcada como **Build Variable: Yes** para que Vite la incruste al compilar.
- Las demás variables del backend (DB, JWT, etc.) se configuran como runtime
  env vars normales en Coolify.

## Configuración en Coolify

- **Build pack:** Nixpacks
- **Base Directory:** `/` (raíz del repo)
- **Puerto expuesto:** el que defina la env var `PORT` del backend
- **Healthcheck path:** `/api` (debe devolver `{status:'ok'}` o similar)
- **Build Variables (marcadas como Yes):** `VITE_API_URL`

## Limitaciones conocidas

- El backend debe exponer el frontend desde `frontend/dist`. Si sirve desde
  otra ruta, ajusta el template o el código del backend.
- Un único dominio sirve todo (SPA + API en la misma URL). No hay separación
  de subdominios ni de servicios.
- No apto para escalar backend y frontend por separado: comparten proceso,
  imagen y ciclo de despliegue.
- Cualquier rewrite raro de rutas (p.ej. API duplicado en `/*` por un proxy
  que no respeta prefijos) vive en el código del backend, no en el template.

## Caso de referencia

- [`SmartTestingRD/partido360`](https://github.com/SmartTestingRD/partido360):
  backend Express en la raíz sirviendo `frontend/dist` como SPA, con rutas
  del API respondiendo tanto en `/api/*` como en `/*` por un workaround de
  Traefik. Este template está diseñado para tolerar esa estructura tal cual,
  sin refactor.
