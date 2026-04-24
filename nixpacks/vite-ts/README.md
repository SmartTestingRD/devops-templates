# vite-ts

## Señales de detección
El agente usa este template cuando el repo cumple TODAS estas señales:
- package.json en la raíz con vite en devDependencies
- TypeScript: tsconfig.json presente
- Script build que corre vite build
- NO tiene backend Node en la raíz

Señales de exclusión:
- Tiene backend acoplado en la misma raíz
- No usa TypeScript
