# Playbook: Deploy de un repo SmartTestingRD

## Entrada esperada del usuario
Frase en lenguaje natural que contiene:
- Repo (obligatorio): <org>/<repo> o URL de GitHub
- Entorno (opcional): dev | pre-prod | prod
  Si no se especifica, se deriva de la rama activa según la regla
  de ramas de abajo.

Ejemplos válidos:
- "deploya partido360 en dev"
- "subi api-finanzas a pre-prod"
- "deploya SmartTestingRD/nuevo-repo"

Si falta el repo, pide al usuario que lo aclare antes de continuar.

## Regla de ramas → ambientes

| Rama | Ambiente destino |
|---|---|
| main | prod |
| pre-prod | pre-prod |
| cualquier otra | dev |

Si el usuario no especifica rama ni entorno, pregunta en qué rama
está trabajando antes de continuar.

## Paso 0 — Verificaciones previas y setup del proyecto

### 0.1 — Verificar repo
Confirma que el repo existe y es accesible en GitHub.

### 0.2 — Verificar o crear proyecto en Coolify
Busca en Coolify un proyecto con el mismo nombre que el repo.

Si NO existe:
1. Crea el proyecto con el nombre del repo.
2. Crea exactamente 3 ambientes: dev, pre-prod, prod.
3. Reporta los UUIDs del proyecto y cada ambiente.

Si YA existe:
1. Verifica que tenga los 3 ambientes (dev, pre-prod, prod).
2. Si falta alguno, créalo.
3. Reporta UUIDs.

### 0.3 — Verificar o crear las 3 apps
Para cada ambiente, crea una app si no existe:

| Ambiente | Rama git | Nombre app | Dominio |
|---|---|---|---|
| dev | cualquier rama activa | <repo>-dev | <repo>-dev.smarttesting.com.do |
| pre-prod | pre-prod | <repo>-pre-prod | <repo>-pre-prod.smarttesting.com.do |
| prod | main | <repo> | <repo>.smarttesting.com.do |

Parámetros comunes para las 3 apps:
- Build pack: Nixpacks
- Base Directory: /
- Puerto: detectado del código, default 3001
- Healthcheck: /health

### 0.4 — Configurar webhooks automáticos
Para cada app creada:
1. Obtén el webhook secret via get_application
   (campo manual_webhook_secret_github)
2. Construye la URL del webhook:
   https://coolify-mcp.smarttesting.com.do/api/v1/webhooks/source/github/events?api_token=<secret>
3. Registra el webhook en GitHub usando el GITHUB_TOKEN del entorno:

   POST https://api.github.com/repos/<org>/<repo>/hooks
   Headers:
     Authorization: Bearer <GITHUB_TOKEN>
     Accept: application/vnd.github+json
   Body:
   {
     "name": "web",
     "active": true,
     "events": ["push"],
     "config": {
       "url": "<webhook-url>",
       "content_type": "json",
       "secret": "<webhook-secret>"
     }
   }

4. Reporta éxito o error por cada webhook.

Regla de auto-deploy por rama (que Coolify aplicará al recibir
el webhook):
- Push a main → redeploya app de prod
- Push a pre-prod → redeploya app de pre-prod
- Push a cualquier otra rama → redeploya app de dev

### Paso 0.3b — Crear base de datos (si el proyecto la necesita)

Detecta si el repo necesita DB buscando en .env.example las
variables: DATABASE_URL, PGHOST, PGDATABASE, PGUSER, PGPASSWORD.

Si existen → crea automáticamente una DB Postgres en Coolify
dentro del mismo proyecto y ambiente:

Parámetros:
- Nombre: <repo>-<entorno>-db
- Database: <repo>_<entorno> (guiones reemplazados por guiones bajos)
- User: <repo>_<entorno>_user
- Password: autogenerada por Coolify (NO mostrar en chat)

Después de crear la DB:
1. Obtén el internal hostname de la DB creada
2. Configura automáticamente estas env vars en la app:
   - PGHOST = <internal hostname>
   - PGDATABASE = <database name>
   - PGUSER = <user>
   - PGPORT = 5432
   - DATABASE_URL = postgresql://<user>:<password>@<hostname>:5432/<database>
3. Avisa al usuario: "DB creada. Solo falta PGPASSWORD —
   cópiala de la UI de Coolify y agrégala como env var secreta."

Si NO existen esas variables → omite este paso.

## Paso 1 — Detectar tipo de proyecto
Lee el README.md de cada template en
SmartTestingRD/devops-templates/nixpacks/ y compara sus
"Señales de detección" con el repo a deployar.

Proceso:
1. Lee del repo: package.json raíz, presencia de frontend/,
   presencia de src/server.js, presencia de docker-compose.yml.
2. Para cada template, verifica si el repo cumple TODAS sus señales
   y ninguna de exclusión.
3. Si exactamente UN template hace match → úsalo.
4. Si NINGUNO hace match → PARA, reporta qué señales no matchearon.
5. Si MÁS DE UNO hace match → PARA, reporta ambigüedad y pide
   que el usuario especifique.

Templates disponibles:
| Template | Señal principal |
|---|---|
| legacy-monolith | backend Node + frontend/dist en mismo repo |
| express | solo backend Node/Express sin frontend |
| react | solo frontend React sin backend |
| vite-ts | solo frontend Vite+TypeScript sin backend |

## Paso 2 — Resolver parámetros
Construye este objeto antes de continuar:
  app_name: <repo>-<entorno>  (prod: solo <repo>)
  project: nombre del proyecto en Coolify
  repo: <org>/<repo>
  branch: según regla de ramas
  base_directory: /
  port: detectado del código, default 3001
  domain: <repo>-<entorno>.smarttesting.com.do
           (prod: <repo>.smarttesting.com.do)
  healthcheck_path: /health
  template: el detectado en Paso 1
  env_vars_build: { VITE_API_URL: https://<domain>/api }
  env_vars_runtime_public: { NODE_ENV: production, PORT: <port> }
  env_vars_runtime_secret: nombres del .env.example del repo

## Paso 3 — Crear app en Coolify
Crea la aplicación Nixpacks con los parámetros del Paso 2.
Reporta UUID.

## Paso 4 — Configurar env vars
Setea las públicas por API. Lista las secretas como PENDIENTES
y pide al usuario que las configure en la UI de Coolify.
ESPERA confirmación antes de continuar.

## Paso 5 — Deploy
Dispara deploy via MCP. Monitorea el log hasta que termine.
Si falla, reporta las últimas 50 líneas y sugiere solución.

## Paso 6 — Verificación
a) GET https://<domain>/health → 200
b) GET https://<domain>/ → 200

Si alguno falla → PARA y reporta.

## Paso 7 — Reporte final

  ✓ Deploy de <repo> en <entorno> completado
  • App UUID: <uuid>
  • URL: https://<domain>
  • Template usado: <template>
  • Duración del build: <mm:ss>
  • Healthcheck: ✓ 200

## Rollback

Cuándo hacer rollback:
- El usuario lo pide explícitamente.
- El healthcheck falla 3 veces consecutivas con 30s entre intentos.

Proceso:
1. Lista los últimos 5 deployments de la app via MCP.
2. Muestra lista con fecha, commit SHA y estado.
3. Pregunta al usuario a cuál hacer rollback.
4. Espera confirmación con el ID específico.
5. Ejecuta rollback via MCP.
6. Verifica healthcheck post-rollback.
7. Reporta resultado.

NUNCA ejecutar rollback sin confirmación explícita con ID específico.

## Reglas de oro
- NUNCA pedir secretos en el chat. Siempre por UI de Coolify.
- Si algo falla, PARA y reporta. No improvises.
- Si el repo no matchea ningún template, PARA y describe qué falta.
- Deploy a prod requiere confirmación explícita del usuario.

## Docker — Cuándo usarlo
El agente usa Docker únicamente cuando el usuario lo pide
explícitamente en su instrucción. Ejemplos:
- "deploya partido360 en dev con Docker"
- "usa Docker para subir api-finanzas a pre-prod"

En cualquier otro caso, el agente usa Nixpacks por defecto.

## Docker — Flujo de deploy

### Paso D1 — Verificar Dockerfile
Busca `Dockerfile` en la raíz del repo (rama del entorno).

- Si existe → úsalo tal cual.
- Si NO existe → crea uno usando el template correspondiente de
  `SmartTestingRD/devops-templates/docker/`:

  | Tipo de proyecto | Template a usar |
  |---|---|
  | Backend Node/Express | docker/node/Dockerfile |
  | Frontend React/Vite | docker/react/Dockerfile |
  | Monorepo Node+React | docker/monorepo/node-react/docker-compose.yml |

  Copia el template al repo, ajusta si es necesario (puerto, entry
  point), haz commit en la rama del entorno antes de continuar.

### Paso D2 — Crear app en Coolify
Mismos parámetros que el flujo Nixpacks (Paso 2) pero con:
- Build pack: Dockerfile (no Nixpacks)
- Dockerfile path: /Dockerfile

### Paso D3 — Env vars, deploy y verificación
Igual que Pasos 4, 5, 6 y 7 del flujo Nixpacks.

### Paso D4 — Rollback Docker
Igual que la sección Rollback del flujo Nixpacks.

### Limitación conocida — Docker Compose
Si el proyecto usa docker-compose.yml, Coolify requiere que el
PRIMER deploy se haga manualmente desde la UI. El agente puede
hacer redeploys posteriores via MCP, pero no la creación inicial.
Si detectas docker-compose.yml, PARA y avisa al usuario que debe
hacer el primer deploy desde la UI de Coolify.
