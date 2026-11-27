# Plan de Implementación: Práctica 4 - CI/CD con Angular y GitHub Actions

## Contexto y Punto de Partida

* El proyecto Angular **ya está creado** en este directorio.
* Ya existe un borrador inicial de `.github/workflows/cicd.yml`.
* **Objetivo:** Inspeccionar el proyecto actual, implementar la aplicación con componentes y servicios reales, configurar los tests unitarios con umbrales de cobertura (+0,5 pts), ajustar `.github/workflows/cicd.yml` a la estructura real del proyecto y dejar una guía clara para las pruebas y capturas en GitHub.

## Fase 1: Inspección y Ajuste del Proyecto Existente

1. **Auditar configuración actual:**
   * Revisar `package.json` y `angular.json` para detectar la versión de Angular, el nombre exacto del proyecto (`projects.<nombre>`), el test runner configurado (Karma/Jasmine o Vitest/Jest) y la ruta de salida del build (`outputPath`, ej. `dist/<proyecto>/browser` o `dist/<proyecto>`).
   * Revisar el archivo `.github/workflows/cicd.yml` existente para conservar su estructura y completarlo en la Fase 3.

2. **Implementar funcionalidad real en Angular:**
   * Mantener o crear una interfaz limpia en el componente principal (`AppComponent` / `App`) con un título visible (`<h1>Práctica 4: CI/CD con Angular y GitHub Pages</h1>`) para poder hacer cambios visibles fácilmente en una Pull Request.
   * Implementar **1 servicio** (`TaskService`) con lógica real (listar, añadir, completar y filtrar tareas).
   * Implementar **2 componentes** (`TaskListComponent` y `StatusBannerComponent`) que interactúen con el servicio o reciban datos y respondan a eventos de usuario (clics, inputs).

## Fase 2: Extensión (+0,5 pts) - Tests Unitarios y Cobertura

1. **Tests unitarios significativos (`*.spec.ts`):**
   * Escribir pruebas reales (no solo `should create`) para:
     * `TaskService` (comprobar retorno de datos, inserción y cambio de estado).
     * `TaskListComponent` (comprobar renderizado en el DOM y reacción a eventos de clic/input).
     * `StatusBannerComponent` (comprobar visualización según el estado recibido).

2. **Modo no interactivo y umbrales mínimos de cobertura:**
   * Configurar el entorno de pruebas para ejecutarse en CI sin interfaz gráfica (`ChromeHeadless`, `--no-watch` / `--watch=false`, `--code-coverage`).
   * Configurar en el bloque de cobertura (`karma.conf.js` o equivalente según el runner detectado) los umbrales mínimos obligatorios que hacen fallar el comando si no se cumplen:
     * **Statements (sentencias):** 80 %
     * **Functions (funciones):** 80 %
     * **Lines (líneas):** 80 %
     * **Branches (ramas):** 70 %
   * Asegurar un script `"test:ci"` en `package.json` listo para ejecutarse en GitHub Actions.

## Fase 3: Completar y Verificar `.github/workflows/cicd.yml`

Revisar el archivo `.github/workflows/cicd.yml` ya existente y actualizarlo asegurando que incluye comentarios breves explicando cada bloque (`on`, `jobs`, `needs`, `if`, `permissions`) para la memoria final:

1. **Disparadores (`on`):**
   * `pull_request` hacia `main`.
   * `push` hacia `main`.

2. **Job `build` (Integración Continua):**
   * `runs-on: ubuntu-latest`
   * `permissions: contents: read`
   * Pasos (`steps`):
     1. `actions/checkout@v4` (Descargar código).
     2. `actions/setup-node@v4` con `node-version: '22'` y `cache: 'npm'`.
     3. `npm ci` (Instalar dependencias exactas).
     4. Ejecutar tests y cobertura (`npm run test:ci`). Debe fallar el job si algún test falla o si no se alcanza el 80%/70%.
     5. Subir informe de cobertura como artefacto con `actions/upload-artifact@v4` (con `if: always()`, apuntando a `coverage/`).
     6. Compilar Angular: `npx ng build --base-href /<NOMBRE_REPO>/` (si no se puede inferir el nombre del repo desde `git remote -v`, usar el nombre del proyecto de `angular.json` y avisar al usuario para que lo verifique).
     7. Subir artefacto de GitHub Pages con `actions/upload-pages-artifact@v3`, **solo** cuando `github.event_name == 'push'`, usando exactamente la ruta verificada en `angular.json` (`dist/<proyecto>/browser` o `dist/<proyecto>`).

3. **Job `deploy` (Despliegue Continuo):**
   * `needs: build`
   * `if: github.event_name == 'push' && github.ref == 'refs/heads/main'`
   * `runs-on: ubuntu-latest`
   * `permissions`: `pages: write` e `id-token: write`.
   * `environment`: `name: github-pages` y `url: ${{ steps.deployment.outputs.page_url }}`.
   * Paso: `actions/deploy-pages@v4` con `id: deployment`.

## Fase 4: Verificación Local y Guía de Entrega

1. **Comprobación local obligatoria:**
   * Ejecutar `npm run test:ci` y verificar que pasan todos los tests y se superan los umbrales de cobertura (80% / 70%).
   * Ejecutar `npx ng build --base-href /<NOMBRE_REPO>/` y verificar que compila limpio y existe la carpeta exacta referenciada en `upload-pages-artifact`.

2. **Generar `GUIA_ENTREGA.md` en la raíz:**
   * Comandos Git exactos para subir los cambios a `main`.
   * Checklist paso a paso con todas las capturas (`📷`) necesarias en GitHub:
     * PR con error de compilación (`✘` y merge bloqueado) -> corrección (`✔` y merge disponible) -> merge a `main` (`build` + `deploy`) -> web en GitHub Pages -> segunda PR con cambio visible.
     * Pruebas de extensión: PR con test fallido (`✘`), PR que baja la cobertura del 80%/70% (`✘`), y log de Actions con el resumen de cobertura en verde.
   * Copia comentada de `.github/workflows/cicd.yml` lista para pegar en el PDF `Practica1_Apellido_Nombre.pdf`.

## Criterios de Finalización para el Agente

* No recrear el proyecto desde cero; trabajar sobre el existente.
* Confirmar explícitamente al finalizar:
  1. Archivos creados o modificados.
  2. Resultado de cobertura obtenido en local (Statements, Branches, Functions, Lines).
  3. Ruta exacta de `dist/` y `--base-href` configurados en `.github/workflows/cicd.yml`.