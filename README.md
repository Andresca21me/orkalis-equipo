# Orkalis — Salones de belleza y barberías

SaaS multi-tenant para salones de belleza y barberías en Colombia: reservas en línea,
agenda de especialistas, recepción, finanzas y liquidaciones, inventario, mensajería
(SMS/WhatsApp/correo) y suscripciones.

## Equipo

| Integrante | Área principal |
|---|---|
| Andrés Camilo Medina | Arquitectura, base de datos, autenticación, base del frontend y CI |
| Juan Felipe Nauffal | Agendamiento, reserva pública, app del especialista, recepción, configuración y pruebas E2E |
| Andrés Vargas | Negocio y equipo, finanzas, operación, inventario, clientes y onboarding |
| Samuel Ceballos | Mensajería, correo, pagos y suscripciones, sitio web, consola de plataforma y despliegue |

## Tecnologías

- **Monorepo** con pnpm workspaces (`apps/api`, `apps/web`, `packages/shared`).
- **Backend:** NestJS + Drizzle ORM + PostgreSQL 16 (aislamiento por negocio con RLS).
- **Frontend:** React 18 + Vite + TypeScript.
- **Pruebas:** Jest (API) y Playwright (E2E).

## Estructura

```
apps/api/          Backend NestJS (módulos: auth, agendamiento, negocio, finanzas,
                   operacion, notificaciones, correo, pagos, plans, config-module)
apps/api/drizzle/  Migraciones SQL versionadas
apps/web/          Frontend React (admin, especialista, recepción, reserva pública,
                   sitio web, plataforma) y pruebas E2E en apps/web/e2e
packages/shared/   Tipos, enums y DTOs compartidos entre API y web
Documentacion/     Definición del problema, historias de usuario, ADRs, SRS y diagramas
Plan-Ejecucion-*/  Planes de ejecución por fase y por módulo
Prototipo Interfaz Orkalis/  Prototipo navegable de la interfaz
```

## Cómo correrlo en local (sin claves de servicios externos)

Esta copia funciona **sin claves de Twilio, SendGrid ni Mercado Pago**: si esas variables
están vacías, la mensajería usa un adaptador simulado (no envía nada) y los pagos operan
en modo inactivo (pagos simulados).

Requisitos: Node 20+, pnpm 11 y Docker.

```bash
# 1. Base de datos PostgreSQL en el puerto 5433
docker compose up -d

# 2. Dependencias
pnpm install
pnpm --filter @orkalis/shared build

# 3. Variables de entorno (archivos .env, no se suben a git)
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.example apps/web/.env
#    En apps/api/.env llenar SOLO estas dos con cualquier texto de 32+ caracteres:
#    JWT_ACCESS_SECRET=un_secreto_local_de_desarrollo_0123456789
#    JWT_REFRESH_SECRET=otro_secreto_local_de_desarrollo_0123456789
#    Todo lo de Twilio, SendGrid y Mercado Pago se deja vacío.

# 4. Migraciones y datos de demostración
pnpm --filter api db:migrate
pnpm --filter api db:seed

# 5. Levantar API (:3000) y web (:5173)
pnpm --filter api dev
pnpm --filter web dev
```

Abrir http://localhost:5173. Usuarios de demostración (contraseña `Orkalis2026!`):
`admin@orkalis.demo` (admin barbería), `recepcion@barberia.orkalis.demo`,
`carlos@barberia.orkalis.demo` (especialista), `operador@orkalis.demo` (plataforma).
El listado completo está en `Plan-Ejecucion-V2/_CREDENCIALES-SEED.md`.

> Sin la clave pública de Mercado Pago, el formulario de tarjeta del alta de negocio no
> carga; el resto del sistema funciona normal.

## Scripts útiles

| Comando | Qué hace |
|---|---|
| `pnpm -r typecheck` | Verifica tipos en todo el monorepo |
| `pnpm --filter api test` | Pruebas del backend (requiere la base de datos arriba) |
| `pnpm --filter web e2e` | Pruebas E2E con Playwright (API y web corriendo) |
