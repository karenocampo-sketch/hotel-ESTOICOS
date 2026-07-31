# Hotel Estoicos

Sistema operativo hotelero **single-tenant** para gestión integral de un hotel. Incluye un PMS (Property Management System) para el staff y un motor de reservas público para huéspedes, complementado por un asistente conversacional con IA.

## Descripción

Hotel Estoicos cubre dos audiencias:

- **Staff del hotel** — Recepción, housekeeping, reservas internas, reportes y administración.
- **Huéspedes** — Búsqueda de disponibilidad y reserva online desde un portal público tipo Airbnb.

El diferenciador es un **asistente con IA** (Kimi/Moonshot AI — modelo `kimi-latest`) accesible para el staff con acceso a datos del PMS: consulta de disponibilidad, generación de reportes en lenguaje natural y asistencia operativa.

> **Core Value:** Un staff de hotel debe poder gestionar reservas, check-in/out, habitaciones y operación diaria desde una sola interfaz web, complementado por un asistente IA que responde preguntas del PMS y ayuda a tareas operativas.

## Funcionalidades

| Módulo | Descripción |
|--------|------------|
| **Auth y RBAC** | JWT con refresh tokens, roles (admin, manager, reception, housekeeping) |
| **Inventario** | CRUD de habitaciones con estados dual (físico + limpieza), fotos, tipos configurables |
| **Pricing** | Rate plans con multiplicadores estacionales y desglose itemizado |
| **Reservas** | Reservas con prevención de overbooking (btree_gist), check-in/out, folio |
| **Booking Engine** | Portal público tipo Airbnb para reserva online |
| **Operación** | Night audit, cargos a habitación, exportación TRA Colombia |
| **Housekeeping** | Máquina de estados de limpieza, asignación de tareas, realtime (Socket.io) |
| **Reportes** | Dashboard KPI, reportes filtrables, exportación CSV/PDF |
| **Asistente IA Staff** | Chat con streaming SSE, herramientas de solo lectura del PMS |
| **Concierge IA** | Chatbot público para huéspedes con catálogo de Bogotá y anti-abuse |

## Stack Tecnológico

| Capa | Tecnología |
|------|-----------|
| Frontend | React 18 + TypeScript + Vite + Tailwind CSS v4 + shadcn/ui |
| Backend | NestJS 11.x |
| ORM | Prisma 7.x |
| Base de datos | PostgreSQL 16 |
| Auth | JWT + refresh tokens (Passport) |
| Realtime | Socket.io 4.x |
| IA | Kimi/Moonshot AI (`kimi-latest`) via SDK compatible OpenAI |
| Deploy | Railway |
| Monorepo | pnpm workspaces + Turborepo |

## Estructura del Proyecto

```
hotel-estoicos/
├── apps/
│   ├── api/              # Backend NestJS (puerto 3001)
│   │   ├── src/
│   │   ├── prisma/       # Schema + migraciones
│   │   └── .env.example
│   └── web/              # Frontend React + Vite (puerto 5173)
├── packages/
│   └── shared/           # Código compartido (types, utils)
├── design/               # Sistema de diseño
├── docker-compose.yml    # PostgreSQL local
├── turbo.json
└── package.json
```

## Requisitos

| Requisito | Versión mínima |
|-----------|---------------|
| Node.js | >= 20 |
| pnpm | >= 9 |
| Docker | Cualquiera |
| Git | Cualquiera |

## Instalación Local

### 1. Clonar e instalar dependencias

```bash
git clone https://github.com/samestediazmedin/hotel-ESTOICOS.git
cd hotel-ESTOICOS
pnpm install
```

### 2. Configurar variables de entorno

```bash
cp apps/api/.env.example apps/api/.env
```

Editar `apps/api/.env`:

| Variable | Valor local | Notas |
|----------|-----------|-------|
| `DATABASE_URL` | `postgresql://user:password@localhost:5432/hoteldb` | Coincidir con docker-compose |
| `JWT_ACCESS_SECRET` | 32+ caracteres aleatorios | Generar con `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"` |
| `JWT_REFRESH_SECRET` | 32+ caracteres aleatorios | Igual que el anterior |
| `OPENAI_API_KEY` | Tu API key de Kimi/Moonshot | Opcional — funcionalidad IA |
| `RESEND_API_KEY` | `re_xxxx` | Opcional — envío de emails |

### 3. Levantar base de datos

```bash
docker compose up -d
```

PostgreSQL 16 en puerto 5432.

### 4. Migraciones y seed

```bash
cd apps/api
pnpm prisma migrate dev
pnpm prisma db seed
```

### 5. Iniciar desarrollo

```bash
pnpm dev
```

Backend (3001) + Frontend (5173) en paralelo via Turborepo.

**Credenciales admin (seed):**
- Email: `admin@hotel.com`
- Contraseña: `admin123`

## Comandos

| Comando | Descripción |
|---------|-----------|
| `pnpm dev` | Desarrollo (api + web) |
| `pnpm build` | Build producción |
| `pnpm test` | Tests unitarios |
| `pnpm typecheck` | Verificación de tipos |

### En `apps/api/`

| Comando | Descripción |
|---------|-----------|
| `pnpm prisma migrate dev` | Crear migración |
| `pnpm prisma generate` | Generar Prisma Client |
| `pnpm prisma db seed` | Poblar BD |
| `pnpm prisma studio` | UI de Prisma |

## Deployment

Proyecto configurado para **Railway**:

```bash
railway up
```

Ver `DEPLOY_ANALYSIS.md` para guía completa de deployment, variables de entorno de producción y configuración de dominio.

## Arquitectura

El backend sigue el patrón **Hexagonal** (puertos y adaptadores):

- **Dominio** — Entidades, value objects, interfaces de repositorio
- **Aplicación** — Casos de uso / servicios de dominio
- **Infraestructura** — Prisma, HTTP, WebSocket, integraciones externas

Cada bounded context (Auth, Inventory, Reservations, Guests, Operations, Housekeeping, Reporting, AI Assistant) es un módulo NestJS independiente.

## Modelo de IA

**Kimi (Moonshot AI)** con modelo `kimi-latest` vía API compatible con OpenAI (`https://api.moonshot.cn/v1`). El asistente tiene herramientas de solo lectura del PMS: consultar disponibilidad, reportes, estado de habitaciones, huéspedes y reservas.

## Documentación

| Archivo | Contenido |
|---------|----------|
| `CLAUDE.md` | Guía completa del proyecto |
| `DEPLOY_ANALYSIS.md` | Análisis de deployment en Railway |
| `SETUP-LOCAL.md` | Configuración local paso a paso |
| `design/DESIGN-SYSTEM.md` | Sistema de diseño y tokens |
| `.planning/` | Roadmap, requisitos y planificación |

## Licencia

Propietario — Hotel Estoicos.
