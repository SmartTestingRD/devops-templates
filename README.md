# DevOps Templates

Este repositorio contiene las plantillas (`nixpacks.toml`) predefinidas para despliegues en Coolify mediante Nixpacks. 
El objetivo principal es ser consumido por un agente/MCP, de manera que cuando se indique "Despliega en Coolify", el agente pueda conectarse, descargar la plantilla adecuada para el proyecto, incluirla en él y facilitar el despliegue automático.

## Estructura

- `/nixpacks`
  - [`/vite-ts`](./nixpacks/vite-ts/nixpacks.toml): Plantilla para aplicaciones web con Vite + TypeScript (sirviendo archivos estáticos desde `dist`).
  - [`/express`](./nixpacks/express/nixpacks.toml): Plantilla base para APIs/backend Node.js + Express.
  - [`/react`](./nixpacks/react/nixpacks.toml): Plantilla para React (Create React App, sirviendo archivos estáticos desde `build`).
  - [`/python`](./nixpacks/python/nixpacks.toml): Plantilla base para Python (con ejemplos orientados a FastAPI/Uvicorn y Flask/Gunicorn).
  - [`/go`](./nixpacks/go/nixpacks.toml): Plantilla para aplicaciones en Go.
  - [`/monorepo-workspaces`](./nixpacks/monorepo-workspaces/nixpacks.toml): Plantilla genérica para arquitecturas de Workspaces (NPM, Yarn, pnpm).
  - [`/monorepo-turborepo`](./nixpacks/monorepo-turborepo/nixpacks.toml): Plantilla diseñada para Turborepo manipulando las construcciones con filtros.
  - [`/legacy-monolith`](./nixpacks/legacy-monolith/nixpacks.toml): Plantilla para monolitos legacy donde un backend Node sirve un frontend SPA desde `frontend/dist/` bajo un único dominio.

## Templates Nixpacks disponibles

| Template | Descripción |
| --- | --- |
| [`legacy-monolith`](./nixpacks/legacy-monolith/README.md) | Backend Node (Express u otro) que sirve un frontend SPA ya construido desde `frontend/dist/`. Un solo dominio expone API + SPA; pensado para repos que no se pueden/quieren refactorizar. |

## ¿Qué es Nixpacks?

Nixpacks toma tu código fuente y crea una imagen OCI (Docker) lista para producción. Al incluir un archivo `nixpacks.toml` en la raíz de la aplicación que se está construyendo, Coolify lo detectará y aplicará las instrucciones de instalación, construcción y ejecución según se defina en él.

## Cómo el Agente debe usar este repositorio

1. Identificar el framework o lenguaje que está usando la aplicación actual a desplegar.
2. Hacer un `fetch` o lectura del archivo `nixpacks.toml` correspondiente dentro del directorio `/nixpacks/<framework>/`.
3. Crear el archivo `nixpacks.toml` en la raíz del repositorio de la aplicación destino.
4. Proceder con el comando de despliegue correspondiente de Coolify a través del MCP.
