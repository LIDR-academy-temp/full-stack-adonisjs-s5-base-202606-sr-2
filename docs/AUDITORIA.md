# Auditoría de documentación

> Repositorio: `full-stack-adonisjs-s5-base-202606-sr-2` · Fecha: 2026-10-31 · Commit auditado: `a3f40ff` (rama `main`)
> Método: lectura de todos los ficheros versionados (READMEs, `CLAUDE.md`, `docs/`, `openspec/`, backend y frontend) y contraste de cada requirement de las specs contra controllers, rutas, validators y modelo. Análisis estático; no se ejecutó la app ni hay tests que ejecutar.

**Leyenda de estado:** ✅ Completa · 🟡 Parcial o pobre · ❌ Inexistente · ➖ No aplica

## 1. Matriz resumen

| # | Tipo de documentación | Estado | Observación | Ubicación |
|---|---|---|---|---|
| 1 | README de proyecto | ✅ | Estructura, requisitos, arranque en 2 pasos y tabla de endpoints. Pequeñas incoherencias de nombre | `README.md`, `backend/README.md` |
| 2 | Arquitectura general | 🟡 | Árbol de carpetas y stack, pero sin diagramas ni flujo de auth | `README.md`, `backend/README.md`, `docs/PRD.md` §5 |
| 3 | Documentación de la API | 🟡 | Tabla de rutas y specs con códigos de error; sin OpenAPI ni ejemplos de payload | `README.md`, `openspec/specs/` |
| 4 | Docstrings / comentarios en código | 🟡 | JSDoc en métodos de controllers; nada en modelo, validators de reglas ni frontend | `backend/app/**` |
| 5 | Decisiones técnicas (ADRs) | ❌ | `docs/README.md` los promete "en sesiones posteriores"; no existen | — |
| 6 | Guía operacional | 🟡 | Solo arranque en desarrollo; sin despliegue, producción, tests ni troubleshooting | `README.md` |
| 7 | Convenciones de código | ✅ | Reglas explícitas y repetidas de forma coherente en 3 sitios | `CLAUDE.md`, `backend/README.md`, `openspec/config.yaml` |
| 8 | Spec OpenSpec ↔ código | 🟡 | `authentication` coincide al 100 %; `users` promete un endpoint que no existe | `openspec/specs/` |
| 9 | Tests y frontend (adicionales) | ❌ | `backend/tests/functional/` vacío; sin README del frontend | `backend/tests/`, `frontend/` |

**Recuento:** obligatorios → ✅ 2 · 🟡 5 · ❌ 1 · adicionales → ❌ 2

## 2. Detalle por tipo

### 1. README de proyecto — ✅
- Cubre qué es, estructura, requisitos (Node 24), arranque de backend y frontend, endpoints y workflow OpenSpec.
- Incoherencia: el producto es "FlowSync" (PRD y specs) pero README, `package.json` y `GET /` usan `full-stack-adonisjs-master`.
- La sección "Para el formador" y la nota sobre "Sesión 3" son contenido de curso mezclado con el README del producto.

### 2. Arquitectura general — 🟡
- Hay árbol de directorios y stack en `README.md`/`backend/README.md`, y el PRD §5 lista el stack.
- Falta el diagrama C4 y los de arquitectura que `docs/README.md` anuncia; no hay descripción del flujo token → guard → middleware → controller → transformer.
- Frontend sin descripción de capas (`services/`, `components/`, `pages/`, `types/`).

### 3. Documentación de la API — 🟡
- Las 7 rutas reales están en la tabla del README y coinciden con `start/routes.ts`.
- Las specs añaden códigos de estado y errores (`401`, `404`, `422`, `E_INVALID_CREDENTIALS`).
- Faltan esquemas de request/response y ejemplos; no hay OpenAPI/Swagger ni colección de cliente HTTP.

### 4. Docstrings y comentarios — 🟡
- Todos los métodos de controller tienen JSDoc con ruta y propósito; `UserTransformer`, `AuthMiddleware` y los validators también.
- Sin comentarios en `models/user.ts` (p. ej. por qué scrypt, `expiresIn: '30 days'`, `tokenSecretLength`) ni en `config/*` más allá del boilerplate.
- Frontend sin comentarios; solo un `eslint-disable` sin justificación en `DashboardPage.tsx`.
- `users_controller.ts` conserva un bloque "NOTA PARA EL FORMADOR" que no es documentación del código.

### 5. Decisiones técnicas — ❌
- No hay ADRs ni sección de decisiones. Ejemplos sin justificar: SQLite como BD, access tokens opacos con expiración de 30 días, token en `localStorage`, CORS con `origin: true` + `credentials: true`.

### 6. Guía operacional — 🟡
- Cubre instalación, `.env`, `generate:key`, migraciones y `npm run dev`; `.env.example` de ambos lados.
- No cubre: build y arranque en producción (`npm run build`/`start`), variables de producción, ubicación y backup de la SQLite, rotación de `APP_KEY`, ejecución de tests ni resolución de problemas.
- `backend/.env.example` no documenta variables de base de datos (se asume el default de `config/database.ts`).

### 7. Convenciones de código — ✅
- `CLAUDE.md` recoge convenciones (lógica en controllers, VineJS, transformer, `/api/v1`, subpath imports) y lista de "No hacer".
- Se repiten sin contradicción en `backend/README.md` y `openspec/config.yaml`; el código las cumple (rutas sin lógica, transformer en todas las salidas).
- Menor: `CLAUDE.md` anuncia un `AGENTS.md` con symlink que no existe en el repo; no hay convenciones del frontend.

### 8. Spec OpenSpec y trazabilidad — 🟡

| Elemento de la spec | Código | Resultado |
|---|---|---|
| `authentication` · Registro (`POST /account/register`, 201, `{user, token}`, email único, pass 8–180, fullName 2–100) | `new_accounts_controller.ts`, `validators/auth.ts` | ✅ Coincide |
| `authentication` · Login (`last_seen_at`, `{user, token}`, 422 por payload inválido) | `access_tokens_controller.ts`, `loginValidator` | ✅ Coincide |
| `authentication` · Logout (revoca token, `{revoked: true}`, 401 sin token) | `access_tokens_controller.destroy`, `middleware.auth()` | ✅ Coincide |
| `authentication` · Perfil (`{user}`, 401 sin token) | `profiles_controller.ts` | ✅ Coincide |
| `users` · Listado (`{users}`, `created_at` desc, 401) | `users_controller.index` | ✅ Coincide |
| `users` · Usuario por id (`{user}`, 404 `E_ROW_NOT_FOUND`) | `users_controller.show` (`findOrFail`) | ✅ Coincide |
| `users` · **Listado de usuarios activos** (`GET /users/active`, 24 h, orden `last_seen_at` desc) | No hay método `active` ni ruta en `start/routes.ts` (solo un comentario). El campo `last_seen_at` sí existe y el login lo actualiza | ❌ **Divergencia: la spec dice "ya implementado" y no lo está** |
| `openspec/changes/` | No existe ningún cambio activo ni archivado; `/health` (S2) no tiene spec | 🟡 Sin trazabilidad del flujo OpenSpec |

- Además la spec `users` afirma en su *Purpose* que documenta comportamiento "ya implementado", lo que agrava el error.
- `README.md` sí avisa de que `/users/active` llega en la Sesión 3, pero la spec no refleja que sea pendiente.

### 9+. Tipos adicionales
- **Tests — ❌**: `backend/tests/functional/` está vacío y solo existe `bootstrap.ts`, aunque `backend/README.md` anuncia "Suite de tests (Japa)". Ningún scenario de las specs tiene test que lo verifique.
- **Frontend — ❌**: sin README ni documentación propia.
- **PRD ↔ implementación — 🟡**: `docs/PRD.md` describe tareas, CSV y Google Calendar, aún sin código (esperable en MVP). Dentro de §3.1, el onboarding tras registro y el aviso "email ya registrado → ir a login" no están implementados en el frontend.

## 3. Top 3 de carencias

1. **Divergencia spec ↔ código en `GET /api/v1/users/active`:** la spec lo da por implementado y no existe, así que la documentación viva no es fiable.
2. **Sin tests ni trazabilidad:** `tests/functional/` vacío y sin `openspec/changes/`; nada verifica que el código cumpla las specs.
3. **Sin ADRs, diagramas ni OpenAPI:** las decisiones (SQLite, tokens en `localStorage`, CORS abierto) y la arquitectura solo se pueden deducir leyendo el código.

## 4. Top 3 de fortalezas

1. **Specs de autenticación precisas:** seis de siete requirements de las specs coinciden exactamente con el código, con scenarios y códigos de error verificables.
2. **Convenciones claras y coherentes:** `CLAUDE.md`, READMEs y `openspec/config.yaml` dicen lo mismo y el código las respeta.
3. **README operativo y rutas documentadas:** un nuevo desarrollador puede arrancar backend y frontend en minutos y la tabla de endpoints coincide con las rutas reales.
