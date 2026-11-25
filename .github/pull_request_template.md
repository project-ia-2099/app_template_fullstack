## Resumen del Cambio
<!-- Describe brevemente qué funcionalidad o corrección introduce este Pull Request -->

---

## Checklist de Cumplimiento (Golden Path Corporativo)

### 1. Gobernanza Git y Documentación
- [ ] Rama creada desde `main` (`feat/*`, `fix/*`, `docs/*`, `refactor/*`, `chore/*`) con commits bajo **Conventional Commits v1.0.0**.
- [ ] `MEMORY.md` actualizado con el plan aprobado y estado de tareas.
- [ ] `README.md` actualizado (si hubo cambios en uso, estructura o endpoints).
- [ ] Cero archivos basura, caches (`*.tsbuildinfo`) o configuraciones locales (`appsettings.Development.json`).

### 2. Arquitectura, Código Limpio y Base de Datos MySQL
- [ ] Nombres de clases, métodos, DTOs y variables en **inglés técnico**.
- [ ] Clean Architecture estricta en 4 capas (`Domain` sin dependencias, `Application` solo depende de `Domain`, `Infrastructure`, `Api`).
- [ ] Lógica de negocio, validaciones y cálculos 100% en Backend (`Domain` / `Application`); cero lógica de negocio en Frontend.
- [ ] Cero fallbacks silenciosos (`catch {}` vacíos, valores mágicos inventados como `-1`).
- [ ] MySQL por área (`local_<area>`, `qa_<area>`, `prod_<area>`) con `CREATE DATABASE IF NOT EXISTS` (solo si `IsRelational()`), prefijo `<nombre>_` en tablas y `MigrationsHistoryTable("__EFMigrationsHistory_<nombre>")` (cero `EnsureCreated()`).

### 3. Estándar Visual UI/UX (Frontend)
- [ ] Iconografía exclusivamente con **Font Awesome Free** (`@fortawesome/fontawesome-free`) y **cero emoticones/emojis Unicode** en `frontend/src/`.
- [ ] Combinación armónica de colores pastel (no monocromática) y formato fijo de títulos/secciones (`h1: 22px`, `h2: 16px`, `max-width: 1120px`).
- [ ] URL de API leída en runtime vía `window.__ENV__?.API_URL ?? import.meta.env.VITE_API_URL` (`98-env-config.sh`).

### 4. Seguridad, Pruebas y CI/CD
- [ ] Cero secretos o credenciales hardcodeadas (`appsettings.json` vacío) y **Gitleaks** en verde.
- [ ] CORS restringido (`AllowAnyOrigin()` prohibido; `Cors__AllowedOrigins__0` inyectado en Cloud Run).
- [ ] Filtro `AllowedClientIps` activo en Backend y Nginx (`99-ip-allowlist.sh`).
- [ ] `dotnet test` (`ArchitectureTests` + integración), `npm test` (Vitest) y `npm run build` ejecutados y en verde.
