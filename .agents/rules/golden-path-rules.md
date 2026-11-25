---
trigger: always_on
description: Reglas obligatorias de Golden Path para flujo Git, Conventional Commits, Clean Architecture, seguridad y MEMORY.md.
---

# Reglas Inquebrantables del Proyecto (Golden Path)

1. **`MEMORY.md` es la Fuente de Verdad:** Lee `MEMORY.md` al iniciar. Todo plan aprobado por el usuario debe escribirse en `MEMORY.md` antes de ejecutar. Actualiza `MEMORY.md` y `README.md` al concluir cada tarea.
2. **Secuencialidad:** No avances a una tarea nueva sin haber terminado y verificado la tarea actual.
3. **Ramas y Conventional Commits v1.0.0:**
   - Prohibido hacer commits directos sobre `main`. Crea siempre una rama (`feat/...`, `fix/...`, `chore/...`, `docs/...`, `refactor/...`).
   - Usa siempre mensajes de commit bajo el estándar **Conventional Commits v1.0.0** (`<tipo>(<alcance>): <descripción>`).
4. **Pull Requests con `gh` y No Auto-Merge:**
   - Todo cambio se sube al remoto y se abre un Pull Request con `gh pr create`.
   - **Prohibido fusionar (`merge`) a `main`** a menos que el usuario lo indique explícitamente.
5. **Cero Archivos Basura y Cero Datos Sensibles:**
   - No subas archivos ajenos al core del proyecto (ante la duda, pregunta al usuario).
   - Prohibido subir credenciales, cadenas de conexión o datos sensibles. Usa **GCP Secret Manager** en nube y `dotnet user-secrets` en local.
6. **Código DRY y Cero Fallbacks Silenciosos:**
   - Reutiliza código (DRY).
   - **No crees fallbacks silenciosos** (`try/catch` que traguen errores o valores por defecto inventados). Si tienes una duda técnica o ambigüedad, pregunta al usuario; no asumas.
7. **Stack y Clean Architecture:**
   - Backend en `.NET 10` Minimal APIs dividido en `Domain`, `Application`, `Infrastructure`, `Api` y `Tests` (`xUnit`).
   - Frontend en React + TypeScript con **Node.js 26 LTS** y `Vitest`.
8. **Convención de Nombres y Lista Cerrada de Áreas (`app-<area>-<nombre>`):**
   - **Áreas oficiales permitidas:** `contabilidad`, `rrhh`, `sistemas`, `comercial`, `almacen`. Si el usuario solicita otra área, **detente e indícale que se comunique con TI para agregarla**.
   - Todo proyecto nuevo debe nombrarse en minúsculas (`kebab-case`, máximo 35 caracteres, regex `^app-(contabilidad|rrhh|sistemas|comercial|almacen)-[a-z0-9-]+$`) para compatibilidad directa con **Google Cloud Run**.
9. **Base de Datos MySQL por Área + Prefijo por Aplicación (Migraciones EF Core):**
   - Todas las aplicaciones de un área comparten una BD por entorno en MySQL (`local_<area>`, `qa_<area>`, `prod_<area>`), creada al arranque con `CREATE DATABASE IF NOT EXISTS` cuando `Database.IsRelational()`.
   - Cada aplicación debe prefijar todas sus tablas con `<nombre>_` (ej. `cajachica_vales`) y configurar `MigrationsHistoryTable("__EFMigrationsHistory_<nombre>")`, aplicando migraciones con `Database.Migrate()`. **Prohibido usar `EnsureCreated()`.**
10. **Lógica de Negocio Exclusiva del Backend:**
    - Cálculos, reglas de negocio y validaciones críticas residen únicamente en `Domain` y `Application` (listo para futura unificación por microservicios). El Frontend es solo presentación.
11. **UI Minimalista Pastel (Combinación Armónica + Formato Fijo), Font Awesome Free y Cero Emojis:**
    - Prohibido usar emoticones/emojis Unicode en botones, títulos, tablas o alertas. Usa únicamente **Font Awesome Free** (`@fortawesome/fontawesome-free`).
    - Usa la **combinación pastel** (no monocromática) y la **escala fija de barra de títulos/secciones** (`h1: 22px`, `h2: 16px`, `max-width: 1120px`, tarjetas `border-radius: 12px`) definidas en `scaffold-clean-fullstack` para que todas las aplicaciones mantengan exactamente el mismo formato visual.
12. **Inglés Técnico en Código y CORS Restringido:**
    - Nombres de clases, interfaces, métodos, DTOs, variables y archivos de código deben escribirse en **inglés técnico** (los textos visibles de UI e informes pueden estar en español).
    - Prohibido `AllowAnyOrigin()` en `Program.cs`; configura orígenes permitidos explícitos (`Cors:AllowedOrigins` o dominio exacto del frontend).
13. **Comunicación Coloquial con el Usuario (Cero Tecnicismos):**
    - Al hablar o hacer preguntas al usuario final (Vicoder), **nunca menciones términos técnicos** (*MySQL, migraciones, backend, frontend, API, ramas, PR, Docker, Cloud Run, xUnit, Vitest, JSON*). Explica todo en lenguaje simple de negocio.
