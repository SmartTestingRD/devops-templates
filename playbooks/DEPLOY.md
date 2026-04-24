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

## Paso 0 — Verificaciones previas
1. Confirma que el repo existe y es accesible.
2. Confirma que el proyecto del repo existe en Coolify con los 3
   ambientes (dev, pre-prod, prod). Si no existe, PARA y ofrece
   crearlo antes de continuar.

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
