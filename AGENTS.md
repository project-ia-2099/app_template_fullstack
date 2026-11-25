# Reglas de Ingeniería, Arquitectura y Flujo de Trabajo (AGENTS.md)

Este repositorio se rige por las reglas obligatorias del **Golden Path Corporativo**. Todo agente de IA (**Codex**, **Antigravity**, **GitHub Copilot**) debe obedecer estas directrices sin excepción:

---

## 1. Fuente de Verdad y Documentación (`MEMORY.md` y `README.md`)
1. **`MEMORY.md` es tu Fuente de Verdad:** Antes de iniciar cualquier tarea, lee `MEMORY.md`. Una vez que un plan sea aprobado por el usuario, **debes registrarlo inmediatamente en `MEMORY.md`** antes de escribir código.
2. **Actualización Continua:** Al completar cada tarea y antes de abrir un Pull Request, actualiza obligatoriamente tanto `MEMORY.md` (estado de tareas, decisiones técnicas y progreso) como `README.md` (si hubo cambios en uso, estructura o endpoints).
3. **Secuencialidad Estricta:** **No se avanza a la siguiente tarea si no se ha terminado y validado por completo la tarea anterior.**

---

## 2. Flujo de Git, Ramas y Pull Requests (`gh` CLI)
1. **Prohibido trabajar directo sobre `main`:** Nunca implementes ni hagas commits directamente en `main`. Crea siempre una rama siguiendo la convención:
   - `feat/<descripcion-corta>` (nuevas funcionalidades)
   - `fix/<descripcion-corta>` (corrección de errores)
   - `docs/<descripcion-corta>` (documentación)
   - `refactor/<descripcion-corta>` (refactorización de código)
   - `chore/<descripcion-corta>` (configuración, mantenimiento, CI/CD)
2. **Conventional Commits v1.0.0 Obligatorios (`https://www.conventionalcommits.org/es/v1.0.0/`):**
   - Formato obligatorio: `<tipo>[alcance opcional]: <descripción en minúsculas>`
   - Tipos permitidos: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
   - Ejemplo: `feat(books): add isbn uniqueness validation` o `fix(api): return 404 when book does not exist`.
3. **Subida al Remoto y Creación de PR con `gh`:**
   - Todo cambio se sube al repositorio remoto y **siempre se crea un Pull Request (`PR`)** usando GitHub CLI (`gh pr create`).
4. **Prohibido Fusionar a `main` sin Orden Explícita:**
   - **Nunca ejecutes `git merge` ni `gh pr merge` hacia `main` por iniciativa propia.** Solo fusiona cuando el usuario lo ordene explícitamente.

---

## 3. Higiene del Repositorio y Seguridad (Cero Exposición)
1. **Cero Archivos Basura:** No subas archivos temporales, logs, artefactos de compilación, scripts de prueba sueltos ni archivos que no pertenezcan al core del proyecto. **Si tienes duda sobre si un archivo debe subirse, pregunta al usuario antes de hacer `git add`.**
2. **Cero Datos Sensibles:** Jamás subas contraseñas, cadenas de conexión, tokens, IPs privadas, claves `.json` ni archivos `.env`.
   - En producción/QA (GCP Cloud Run), las credenciales se inyectan exclusivamente desde **GCP Secret Manager**.
   - En local, usa `dotnet user-secrets` o variables de entorno no versionadas.

---

## 4. Calidad de Código y Arquitectura Limpia
1. **Stack Tecnológico Fijo:**
   - **Backend:** C# sobre `.NET 10` (`net10.0`) con Minimal APIs y Entity Framework Core (**MySQL**).
   - **Frontend:** React con TypeScript (Vite) sobre **Node.js 26 LTS** (`v26.x`).
   - **Tests:** `xUnit` en Backend y `Vitest` + `@testing-library/react` en Frontend.
2. **Clean Architecture Estricta (`backend/`):**
   - `Domain`: Entidades e interfaces de repositorio. Cero dependencias a otros proyectos o paquetes de persistencia.
   - `Application`: Casos de uso, DTOs y servicios de negocio. Solo depende de `Domain`.
   - `Infrastructure`: EF Core (`DbContext`), MySQL e implementación de repositorios. Depende de `Domain` y `Application`.
   - `Api`: Minimal APIs, inyección de dependencias y CORS.
3. **DRY (Don't Repeat Yourself):** Reúsa el código existente. Extrae lógica repetida en funciones, servicios o componentes reutilizables.
4. **Prohibido Crear Fallbacks Silenciosos:** No ocultes errores con `catch {}` vacíos, valores mágicos inventados ni caminos alternativos silenciosos cuando falle una configuración o dependencia. **Si existe cualquier duda técnica o de negocio, detente y pregunta al usuario; nunca asumas.**
5. **Convención de Nombres, Lista Cerrada de Áreas y Aislamiento MySQL:**
   - **Áreas Permitidas (Lista Cerrada):** Únicamente `contabilidad`, `rrhh`, `sistemas`, `comercial`, `almacen`. Si el usuario solicita un área fuera de esta lista, **detente e indícale que se comunique con TI para solicitar su alta**.
   - **Repositorios:** Formato obligatorio `app-<area>-<nombre>` (`^app-(contabilidad|rrhh|sistemas|comercial|almacen)-[a-z0-9-]+$`, máximo 35 caracteres).
   - **Base de Datos MySQL por Área y Prefijo por App:** Cada área comparte una única BD por entorno (`local_<area>`, `qa_<area>`, `prod_<area>`), creada automáticamente al arrancar (`CREATE DATABASE IF NOT EXISTS`) si es relacional. Dentro de esa BD, cada app aísla sus tablas con prefijo `<nombre>_<tabla>` y su historial propio `MigrationsHistoryTable("__EFMigrationsHistory_<nombre>")` usando **Migraciones EF Core** (`Database.Migrate()`). **Prohibido usar `EnsureCreated()`.**
6. **Lógica de Negocio 100% en Backend (Preparado para Microservicios):**
   - Toda regla de negocio, cálculo, validación crítica, estado o auditoría debe vivir exclusivamente en `backend/*.Domain` y `backend/*.Application`. El Frontend (`React`) solo renderiza vistas y envía peticiones a la API.
7. **Estándar UI/UX Corporativo (Combinación Pastel + Formato Fijo + Font Awesome Free):**
   - **Prohibido usar emoticones/emojis Unicode** en la interfaz (`frontend/src/`). Usa únicamente **Font Awesome Free** (`@fortawesome/fontawesome-free`).
   - Todas las aplicaciones deben respetar la **combinación armónica de colores pastel** (evitando diseños monocromáticos) y el **formato fijo de barra de títulos y secciones** (`h1: 22px`, `h2: 16px`, `max-width: 1120px`) definidos en `scaffold-clean-fullstack`.
8. **Comunicación con el Usuario en Lenguaje Coloquial (Cero Tecnicismos):**
   - En tus respuestas y preguntas al usuario **nunca menciones detalles técnicos** (prohibido decir: *MySQL, migraciones, backend, frontend, API, endpoints, Clean Architecture, ramas Git, Pull Request/PR, Docker, Cloud Run, WIF, CORS, Vitest, xUnit, JSON*).
   - Traduce siempre a lenguaje cotidiano de negocio (ej. *"preparando el espacio para tus datos"*, *"construyendo tu aplicación y verificando que todo funcione bien"*, *"enviando tu aplicación al ambiente de pruebas en línea"*, *"¿este campo es obligatorio u opcional?"*).
