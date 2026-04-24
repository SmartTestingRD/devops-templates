# express

## Señales de detección
El agente usa este template cuando el repo cumple TODAS estas señales:
- package.json en la raíz con script start
- Entry point en src/server.js, src/index.js, app.js o similar
- express en dependencies
- NO tiene carpeta frontend/ con package.json propio
- NO tiene docker-compose.yml válido

Señales de exclusión:
- Tiene carpeta frontend/ con su propio package.json
- Es un monorepo con workspaces
