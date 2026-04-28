# Guía de Deploy — SmartTestingRD

## Requisitos previos
- Antigravity instalado
- MCP de Coolify configurado (ver onboarding/SETUP.md)
- Acceso al proyecto en GitHub
- Credenciales de base de datos del entorno a deployar

## Cómo deployar un proyecto desde cero

### 1. Abre Antigravity

### 2. Pega este prompt y sustituye los 4 valores:

```
Lee el archivo playbooks/DEPLOY.md del repo
SmartTestingRD/devops-templates rama main y síguelo
al pie de la letra para este deploy:

- Repo: SmartTestingRD/<NOMBRE-DEL-REPO>
- Rama: <RAMA>
- Entorno: <dev | pre-prod | prod>
- Nombre del proyecto en Coolify: <NOMBRE-DEL-PROYECTO>

No improvises nada fuera del playbook. Si algo no está
cubierto, PARA y pregúntame.
```

### 3. El agente hace automáticamente:
- Lee el playbook
- Verifica el repo en GitHub
- Crea el proyecto en Coolify con 3 ambientes (dev, pre-prod, prod)
- Detecta el framework y elige el template correcto
- Crea las 3 apps con sus dominios
- Configura las variables de entorno públicas

### 4. Cuando el agente pare y pida secretos:
Ve a Coolify UI → proyecto → ambiente → app → Environment Variables
y agrega los valores reales de:
- `JWT_SECRET`
- `DATABASE_URL`
- `PGHOST`, `PGDATABASE`, `PGUSER`, `PGPASSWORD`, `PGPORT`

Luego dile al agente: **"listo, continúa"**

### 5. El agente:
- Dispara el deploy
- Monitorea hasta que termine
- Verifica que la app responde en /health y /

### 6. Verifica en el browser
Abre la URL del entorno y confirma que la app carga y el
login funciona.

## Dominios por entorno
| Entorno | Dominio |
|---|---|
| dev | <repo>-dev.smarttesting.com.do |
| pre-prod | <repo>-pre-prod.smarttesting.com.do |
| prod | <repo>.smarttesting.com.do |

## Regla de ramas → ambientes
| Rama | Ambiente |
|---|---|
| main | prod |
| pre-prod | pre-prod |
| cualquier otra | dev |

## Cómo hacer rollback
Si el deploy falla o la app se rompe, dile al agente:

```
Haz rollback de <nombre-app> en <entorno>
```

El agente lista los últimos 5 deploys, tú eliges a cuál volver
y confirmas con el ID específico.

## Templates disponibles
| Template | Cuándo se usa automáticamente |
|---|---|
| legacy-monolith | Backend Node + frontend React en mismo repo |
| express | Solo API Node/Express sin frontend |
| react | Solo frontend React sin backend |
| vite-ts | Solo frontend Vite+TypeScript sin backend |

## Limitaciones conocidas
- Docker Compose requiere primer deploy manual desde UI de Coolify
- Webhooks automáticos solo funcionan con GitHub Apps (no repos públicos)
- Aislamiento de servidor por team no disponible en Coolify beta.470

## Soporte
Si el agente para y no sabes cómo continuar, contacta al
equipo de infraestructura.
