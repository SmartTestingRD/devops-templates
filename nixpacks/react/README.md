# react

## Señales de detección
El agente usa este template cuando el repo cumple TODAS estas señales:
- package.json en la raíz con react en dependencies
- Script build presente
- NO tiene backend Node en la raíz
- NO tiene carpeta api/ o src/server.js

Señales de exclusión:
- Tiene backend acoplado en la misma raíz
- Es un monorepo con backend y frontend juntos
