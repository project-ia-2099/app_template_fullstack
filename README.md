# App Template Fullstack (Golden Path - Vicoders)

Repositorio plantilla corporativo gobernado por TI para la creación de aplicaciones Fullstack asistidas por IA (**Codex / Antigravity**).

> **Nota:** Este repositorio plantilla contiene únicamente la estructura de gobernanza, reglas (`Rules`), procedimientos (`Skills`), memoria del proyecto (`MEMORY.md`), pipelines CI/CD y documentación de uso. El código fuente de cada aplicación (`backend/` en .NET 10 Clean Architecture y `frontend/` en React + TypeScript con Node 26 LTS) es generado automáticamente por el agente mediante las **Skills y Rules** incluidas en `.agents/`.

---

## 📋 Contenido del Template

| Archivo / Ruta | Propósito |
| :--- | :--- |
| [`MEMORY.md`](./MEMORY.md) | **Fuente de Verdad del proyecto.** Registra los planes aprobados por el usuario, decisiones de arquitectura y el estado secuencial de las tareas. |
| [`AGENTS.md`](./AGENTS.md) | Reglas globales leídas automáticamente por **Codex**, **Antigravity** y **GitHub Copilot**. |
| [`.agents/rules/golden-path-rules.md`](./.agents/rules/golden-path-rules.md) | Reglas inquebrantables (`always_on`): Conventional Commits v1.0.0, ramas obligatorias, PR siempre con `gh`, DRY, cero fallbacks silenciosos y cero datos sensibles. |
| [`.agents/skills/scaffold-clean-fullstack/SKILL.md`](./.agents/skills/scaffold-clean-fullstack/SKILL.md) | Skill para inicializar la solución `.NET 10` en 4 capas de Clean Architecture + `React TS` (`Node 26 LTS`) + `Docker`. |
| [`.agents/skills/add-crud-feature/SKILL.md`](./.agents/skills/add-crud-feature/SKILL.md) | Skill para implementar módulos de negocio capa por capa con pruebas obligatorias. |
| [`.agents/skills/git-pr-workflow/SKILL.md`](./.agents/skills/git-pr-workflow/SKILL.md) | Skill para gestionar ramas, auditoría de secretos/archivos basura, Conventional Commits y creación de Pull Requests con `gh`. |
| [`plan.md`](./plan.md) | Definición operativa de las 4 fases del **Golden Path** (Local, Filtro TI, QA en GCP y Pase a Producción). |
| [`.github/copilot-instructions.md`](./.github/copilot-instructions.md) | Criterios de auditoría automática para **GitHub Copilot Code Review** en cada Pull Request. |
| [`.github/pull_request_template.md`](./.github/pull_request_template.md) | Checklist obligatorio cargado automáticamente en cada Pull Request para validar arquitectura, UI, MySQL y seguridad. |
| [`.github/workflows/ci-cd.yml`](./.github/workflows/ci-cd.yml) | Pipeline automatizado: Filtro de TI (Tests + Clean Architecture + Cero Secretos), Despliegue a Cloud Run (QA) y Pase a Producción con aprobación de 1 clic. |

---

## 🛡️ Las 11 Reglas de Oro del Flujo de Trabajo

1. **Conventional Commits v1.0.0:** Todo commit sigue `<tipo>(<alcance>): <descripción>` (`feat`, `fix`, `docs`, `refactor`, `test`, `chore`, etc.).
2. **Prohibido trabajar sobre `main`:** Todo cambio se implementa en una rama (`feat/*`, `fix/*`, `docs/*`, `chore/*`).
3. **No Auto-Merge y Secuencialidad:** No se fusiona a `main` sin orden explícita del usuario y no se avanza a una tarea nueva sin terminar la anterior.
4. **Uso obligatorio de `gh` CLI:** Todo cambio se sube al remoto y se gestiona mediante `gh`.
5. **Pull Request (`PR`) siempre:** Cada entrega genera un PR hacia `main` (`gh pr create`).
6. **Cero archivos basura:** No se suben archivos ajenos al core; ante cualquier duda, el agente pregunta al usuario.
7. **Cero exposición de datos sensibles (3 Entornos Preconfigurados bajo `ConnectionStrings__DefaultConnection`):** Ninguna credencial o secreto en el repositorio:
   * **Local:** Variable de entorno de sistema `ConnectionStrings__DefaultConnection` preconfigurada hacia SQL Server Local con autenticación integrada de Windows.
   * **Autenticación Zero-Trust (`WIF OIDC` por Entorno):** GitHub Actions se autentica mediante **Workload Identity Federation** (tokens efímeros sin claves JSON) usando identidades separadas **`deployer-qa`** y **`deployer-prod`**. Cloud Run ejecuta cada servicio con **`cloudrun-runtime-qa`** o **`cloudrun-runtime-prod`** con acceso acotado exclusivamente a los secretos de su entorno.
   * **QA (Cloud Run):** Secreto `db-connection-string-qa` en **GCP Secret Manager** inyectado en `ConnectionStrings__DefaultConnection` apuntando a `qa_<area>` vía túnel interno `--add-cloudsql-instances`.
   * **Producción (Cloud Run):** Secreto `db-connection-string-prod` en **GCP Secret Manager** inyectado en `ConnectionStrings__DefaultConnection` apuntando a `prod_<area>` vía túnel interno `--add-cloudsql-instances` (requiere 1 clic del Administrador de TI `888Leonidas888` en *Run workflow*).
   * **Lista blanca de IPs (`AllowedClientIps`):** Secreto `allowed-client-ips` en **GCP Secret Manager** inyectado en QA y Producción tanto en Backend como en Frontend (`nginx`) para restringir el acceso completo a las IPs corporativas autorizadas.
   * **Enlaces Oficiales de QA y Producción:** Se publican automáticamente en el **Resumen de GitHub Actions (`$GITHUB_STEP_SUMMARY`)**, en la sección **Deployments** del repositorio y son entregados en el chat por el agente tras cada despliegue.
8. **Código DRY (Don't Repeat Yourself):** Reutilización obligatoria de lógica y componentes.
9. **Cero fallbacks silenciosos:** Prohibido ocultar errores o asumir comportamientos; ante cualquier duda técnica se consulta al usuario.
10. **Sincronización de `README.md` y `MEMORY.md`:** Ambos documentos se mantienen actualizados en cada entrega.
11. **`MEMORY.md` como Fuente de Verdad:** Una vez aprobado el plan por el usuario, se registra en `MEMORY.md` antes de ejecutar.
12. **Convención de Nombres y Áreas Permitidas (`app-<area>-<nombre>`):** Lista cerrada de áreas (`contabilidad`, `rrhh`, `sistemas`, `comercial`, `almacen`); cualquier otra área requiere solicitud a TI. Formato en minúsculas (`kebab-case`, máx. 35 caracteres) para compatibilidad directa con Cloud Run.
13. **Base de Datos MySQL por Área + Prefijo por Aplicación:** Cada área comparte una BD por entorno (`local_<area>`, `qa_<area>`, `prod_<area>`) creada al arrancar si no existe. Cada app aísla sus tablas con prefijo `<nombre>_` y su propio historial `MigrationsHistoryTable("__EFMigrationsHistory_<nombre>")` vía Migraciones EF Core (`Database.Migrate()`), prohibiendo `EnsureCreated()`.
14. **Lógica de Negocio 100% en Backend:** Toda regla, cálculo o validación crítica reside exclusivamente en `.NET 10` (`Domain` / `Application`) para permitir futura unificación bajo arquitectura de microservicios.
15. **Estándar UI Minimalista Pastel y Font Awesome Free:** Prohibido usar emoticones/emojis Unicode en la interfaz; todas las apps comparten combinación armónica de colores pastel, escala fija de títulos/header (`h1: 22px`, `h2: 16px`, `max-width: 1120px`) e iconografía de **Font Awesome Free**.
16. **Código en Inglés Técnico y CORS Restringido:** Los símbolos de código se escriben en inglés técnico y está prohibido `AllowAnyOrigin()` en Producción.

---

## 🔄 Secuencia Oficial del Flujo (Vicoder — Agente — TI)

1. **Petición en lenguaje natural:** El Vicoder describe su necesidad en Codex / Antigravity y aprueba el plan propuesto.
2. **Construcción y prueba local:** El agente implementa la aplicación en una rama (`feat/*`), ejecuta pruebas automatizadas y se la muestra al usuario en local.
3. **Pull Request (PR):** El agente sube la rama y entrega el enlace del PR para revisión.
4. **Revisión y Pase a QA:** El Ingeniero TI aprueba y fusiona el PR hacia `main`, lo que dispara el **Filtro de TI (`Gitleaks` + tests)** y despliega automáticamente el enlace de **QA** en Cloud Run.
5. **Validación en QA:** El Vicoder prueba la aplicación en el enlace de QA y confirma que cumple con lo requerido.
6. **Pase a Producción:** El Ingeniero TI ejecuta el despliegue a **Producción** (`Run workflow` / aprobación en GitHub Actions) y entrega el enlace oficial productivo.

> [!WARNING]
> **Nota sobre GitHub Free en repositorios privados:** Mientras la organización permanezca en el plan gratuito (`GitHub Free`) con repositorios privados sin *Branch Protection*, GitHub no bloquea técnicamente la autoaprobación de Pull Requests y el propio agente/usuario podría ejecutar el merge si se le solicita. Al migrar al plan **GitHub Team**, se activa la protección obligatoria en `main` para impedir que el autor apruebe o fusione su propio PR.
