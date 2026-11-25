---
name: scaffold-clean-fullstack
description: >-
  Inicializa el esqueleto completo de código de una aplicación Fullstack (.NET 10
  Minimal API en Clean Architecture de 4 capas + React TypeScript en Node 26 LTS
  + Docker + Tests de Arquitectura e Integración) sobre el repositorio plantilla.
---

# Skill: Inicializar Esqueleto Fullstack en Clean Architecture

Usa este skill cuando el usuario solicite iniciar el desarrollo de una aplicación sobre este template y aún no existan las carpetas `backend/` y `frontend/`.

## Pre-requisitos Obligatorios
1. Lee `MEMORY.md` y verifica que el plan de implementación haya sido aprobado por el usuario y esté registrado en `MEMORY.md`.
2. Verifica que **no estás en la rama `main`**. Si estás en `main`, crea primero una rama de trabajo (`git checkout -b feat/initial-scaffold`).
3. Si tienes alguna duda técnica sobre los campos de la entidad principal o reglas de negocio, **pregunta al usuario antes de generar código (no asumas ni crees fallbacks silenciosos)**.

---

## Procedimiento Paso a Paso

### Paso 1: Crear la Solución Backend en Clean Architecture (`backend/`)
Crea la solución `backend/Books.sln` (o el nombre del dominio indicado en `MEMORY.md`) con **5 proyectos sobre `.NET 10` (`net10.0`)**:

1. **`Domain` (`*.Domain.csproj`):**
   - Cero referencias a otros proyectos o paquetes NuGet.
   - Contiene `Entities/` (entidades de dominio) e `Interfaces/` (contratos de repositorio).
2. **`Application` (`*.Application.csproj`):**
   - Referencia **únicamente** a `Domain`.
   - Contiene `DTOs/` y `Services/` (casos de uso y reglas de negocio).
3. **`Infrastructure` (`*.Infrastructure.csproj`):**
   - Referencia a `Domain` y `Application`.
   - Paquetes: **`Pomelo.EntityFrameworkCore.MySql`** (`9.0.0`, usando `UseMySql` + `MySqlConnector`, compatible de forma nativa con los Unix Sockets `/cloudsql/` de Cloud Run), `Microsoft.EntityFrameworkCore.InMemory` (`9.0.*`) y `Microsoft.EntityFrameworkCore.Design` (`9.0.*`) manteniendo `<TargetFramework>net10.0</TargetFramework>`. *(No usar `MySql.EntityFrameworkCore`/`MySql.Data` porque falla en Unix Sockets de Cloud Run con `Socket.Available`).*
   - Contiene `Data/AppDbContext.cs`, `Repositories/`, `Migrations/` y `DependencyInjection.cs`.
   - **Base de Datos MySQL por Área + Prefijo por App:**
     - Toma la cadena base de `ConnectionStrings:DefaultConnection` y asigna dinámicamente `Database=<entorno>_<area>` según `ASPNETCORE_ENVIRONMENT` (`local_<area>` en Development, `qa_<area>` en QA, `prod_<area>` en Production).
     - En `AppDbContext.OnModelCreating`, mapea **todas las tablas** con prefijo `<nombre>_` (ej. `ToTable("cajachica_vales")`) para evitar colisiones dentro de la misma BD de área.
     - En `DependencyInjection.cs`, configura el historial propio por app: `mysql.MigrationsHistoryTable("__EFMigrationsHistory_<nombre>")`.
   - Genera la migración inicial (`dotnet ef migrations add InitialCreate ...`). **Prohibido usar `EnsureCreated()`.**
4. **`Api` (`*.Api.csproj`):**
   - Referencia a `Application` e `Infrastructure`.
   - `appsettings.json` con `"DefaultConnection": ""` (vacío por seguridad; nunca hardcodear secretos).
   - `Program.cs` con Minimal APIs, `/health`, **CORS restringido** (prohibido `AllowAnyOrigin()`; leer orígenes de `Cors:AllowedOrigins` o permitir explícitamente el origen del frontend en `localhost` / Cloud Run del servicio), **filtro de IP opcional (`AllowedClientIps`)** (cuando `AllowedClientIps` está configurado en QA/Producción, validar la primera IP de `X-Forwarded-For` o `RemoteIpAddress` excepto en `/health` y devolver `403 Forbidden` si no coincide) y arranque seguro en MySQL: cuando `db.Database.IsRelational()`, ejecuta `CREATE DATABASE IF NOT EXISTS \`<entorno>_<area>\`` sobre la conexión base y luego `db.Database.Migrate()`.
5. **`Tests` (`*.Tests.csproj`):**
   - Paquetes: `xunit`, `xunit.runner.visualstudio`, `Microsoft.NET.Test.Sdk`, `Microsoft.AspNetCore.Mvc.Testing` (`10.0.*`).
   - Incluye obligatoriamente `ArchitectureTests.cs` validando por reflexión que `Domain` no dependa de nada y `Application` solo dependa de `Domain`.
   - En pruebas de integración con `WebApplicationFactory<Program>`, configura `UseEnvironment("Development")` y reemplaza `DbContextOptions<AppDbContext>` por `UseInMemoryDatabase(...)` (o inyecta un `ConnectionStrings:DefaultConnection` dummy vía `UseSetting`) para que `Filtro TI` en GitHub Actions pase sin requerir MySQL levantado.

### Paso 2: Crear el Frontend en React + TypeScript (`frontend/`)
1. Configura `package.json` (incluyendo `@fortawesome/fontawesome-free`), `tsconfig.json` (con `"types": ["vite/client", "vitest/globals", "@testing-library/jest-dom"]`) y `vite.config.ts`.
2. Lee la URL del backend desde `window.__ENV__?.API_URL ?? import.meta.env.VITE_API_URL` (sin crear fallbacks silenciosos ni `catch {}` vacíos que oculten errores; tampoco envíes valores mágicos inventados como `-1` para campos vacíos — envía `null` o deja validar al Backend). En `index.html`, incluye `<script src="/env-config.js"></script>` antes de `main.tsx` para soportar inyección en tiempo de ejecución en Cloud Run.
3. **Cero Lógica de Negocio en Frontend:** React únicamente renderiza datos y envía formularios hacia la API; toda validación de negocio o cálculo vive en el Backend (`Application` / `Domain`).
4. **Identidad Visual Corporativa (Combinación Pastel Armónica + Formato Fijo de Secciones + Font Awesome Free):**
   - **Cero emoticones/emojis Unicode** en toda la interfaz; usa únicamente iconos de **Font Awesome Free** (`import '@fortawesome/fontawesome-free/css/all.min.css'` y `<i className="fa-solid fa-..." />`).
   - **Combinación de colores pastel (prohibido diseño monocromático de un solo color):** Usa una tríada pastel equilibrada:
     - Fondo general `#F7F9FC`, tarjetas `#FFFFFF` con borde `#E2E8F0` (`box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04)`).
     - **Barra de títulos (Header):** Fondo pastel suave combinado (`linear-gradient(90deg, #EEF2FF 0%, #ECFDF5 100%)`) con borde inferior `#CBD5E1`, caja de icono en `#E0E7FF` (icono `#4F46E5`) y badge de área en menta pastel (`#D1FAE5`, texto `#065F46`).
     - **Botón principal:** Azul pizarra pastel `#5B7C99` (`hover: #4A6984`); **acento secundario/bordes activos:** lavanda `#C7D2FE`.
     - **Estados:** Éxito salvia `#D1FAE5`/`#065F46`, alerta ámbar pastel `#FEF3C7`/`#92400E`, peligro rosa pastel `#FFE4E6`/`#BE123C`.
   - **Formato y tipografía estrictamente uniformes entre aplicaciones:**
     - Contenedor principal centrado `max-width: 1120px`, fuente `Inter, system-ui, sans-serif`, texto `#1E293B` / `#64748B`.
     - **Barra de títulos (Header):** `padding: 16px 24px`, caja de icono `40px × 40px` (`border-radius: 10px`), título **`h1` siempre `font-size: 1.375rem` (`22px`, `font-weight: 600`, `line-height: 1.25`)**, subtítulo **`0.875rem` (`14px`)**.
     - **Tarjetas y secciones (`form-card`, `table-card`):** `border-radius: 12px`, `padding: 20px 24px`, títulos de sección **`h2` siempre `1rem` (`16px`, `font-weight: 600`)**, etiquetas de formulario `0.8125rem` (`13px`), inputs/botones `height: 38px`, `border-radius: 8px`, `font-size: 0.875rem`.
5. Agrega pruebas unitarias en `src/App.test.tsx` con `Vitest` y `@testing-library/react`.

### Paso 3: Contenedores Docker (`Dockerfile` y `docker-compose.yml`)
1. `backend/Dockerfile`: Multi-stage con `mcr.microsoft.com/dotnet/sdk:10.0` y `mcr.microsoft.com/dotnet/aspnet:10.0` en puerto `8080`.
2. `frontend/Dockerfile` + `frontend/nginx.conf`: Multi-stage con `node:26-alpine` y `nginx:alpine` en puerto `8080`. Incluye:
   - `/docker-entrypoint.d/98-env-config.sh`: genera `/usr/share/nginx/html/env-config.js` con `window.__ENV__ = { API_URL: "${API_URL:-}" };` al arrancar el contenedor, permitiendo promover **la misma imagen inmutable** de QA a Producción sin recompilar.
   - `/docker-entrypoint.d/99-ip-allowlist.sh`: si `AllowedClientIps` está definido en entorno, genera `/etc/nginx/conf.d/allowlist.inc` con `allow <IP>;` por cada IP y `deny all;` (usando `set_real_ip_from 0.0.0.0/0; real_ip_header X-Forwarded-For; real_ip_recursive on;`) para devolver `403` directo desde Nginx a IPs no autorizadas.
3. `docker-compose.yml`: Servicios `mysql` (`mysql:8.0`), `books-api` y `books-web`.

### Paso 4: Verificación y Documentación
1. Ejecuta `dotnet test backend/*.sln` y verifica que todos los tests pasen.
2. Ejecuta `cd frontend && npm install && npm test && npm run build` usando **Node 26 LTS**.
3. Actualiza `MEMORY.md` y `README.md` y utiliza el skill `git-pr-workflow` para subir la rama y crear el Pull Request con `gh`.
