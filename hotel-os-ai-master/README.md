# Hotel Estoicos

Sistema operativo hotelero **single-tenant** para gestion integral de un hotel. Incluye un PMS (Property Management System) para el staff y un motor de reservas publico para huespedes, complementado por un asistente conversacional con IA.

## Descripcion

Hotel Estoicos cubre dos audiencias:

- **Staff del hotel** - Recepcion, housekeeping, reservas internas, reportes y administracion.
- **Huespedes** - Busqueda de disponibilidad y reserva online desde un portal publico tipo Airbnb.

El diferenciador es un **asistente con IA** (Kimi/Moonshot AI - modelo `kimi-latest`) accesible para el staff con acceso a datos del PMS: consulta de disponibilidad, generacion de reportes en lenguaje natural y asistencia operativa.

> **Core Value:** Un staff de hotel debe poder gestionar reservas, check-in/out, habitaciones y operacion diaria desde una sola interfaz web, complementado por un asistente IA que responde preguntas del PMS y ayuda a tareas operativas.

## Funcionalidades

| Modulo | Descripcion |
|--------|------------|
| **Auth y RBAC** | JWT con refresh tokens, roles (admin, manager, reception, housekeeping) |
| **Inventario** | CRUD de habitaciones con estados dual (fisico + limpieza), fotos, tipos configurables |
| **Pricing** | Rate plans con multiplicadores estacionales y desglose itemizado |
| **Reservas** | Reservas con prevencion de overbooking (btree_gist), check-in/out, folio |
| **Booking Engine** | Portal publico tipo Airbnb para reserva online |
| **Operacion** | Night audit, cargos a habitacion, exportacion TRA Colombia |
| **Housekeeping** | Maquina de estados de limpieza, asignacion de tareas, realtime (Socket.io) |
| **Reportes** | Dashboard KPI, reportes filtrables, exportacion CSV/PDF |
| **Asistente IA Staff** | Chat con streaming SSE, herramientas de solo lectura del PMS |
| **Concierge IA** | Chatbot publico para huespedes con catalogo de Bogota y anti-abuse |

## Stack Tecnologico

| Capa | Tecnologia |
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
  apps/
    api/              # Backend NestJS (puerto 3001)
      src/
      prisma/         # Schema + migraciones
      .env.example
    web/              # Frontend React + Vite (puerto 5173)
  packages/
    shared/           # Codigo compartido (types, utils)
  design/             # Sistema de diseno
  docker-compose.yml  # PostgreSQL local
  turbo.json
  package.json
```

## Requisitos

| Requisito | Version minima |
|-----------|---------------|
| Node.js | >= 20 |
| pnpm | >= 9 |
| Docker | Cualquiera |
| Git | Cualquiera |

## Instalacion Local

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
| `OPENAI_API_KEY` | Tu API key de Kimi/Moonshot | Opcional - funcionalidad IA |
| `RESEND_API_KEY` | `re_xxxx` | Opcional - envio de emails |

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
- Contrasena: `admin123`

## Comandos

| Comando | Descripcion |
|---------|-----------|
| `pnpm dev` | Desarrollo (api + web) |
| `pnpm build` | Build produccion |
| `pnpm test` | Tests unitarios |
| `pnpm typecheck` | Verificacion de tipos |

### En `apps/api/`

| Comando | Descripcion |
|---------|-----------|
| `pnpm prisma migrate dev` | Crear migracion |
| `pnpm prisma generate` | Generar Prisma Client |
| `pnpm prisma db seed` | Poblar BD |
| `pnpm prisma studio` | UI de Prisma |

## Deployment

Proyecto configurado para **Railway**:

```bash
railway up
```

Ver `DEPLOY_ANALYSIS.md` para guia completa de deployment, variables de entorno de produccion y configuracion de dominio.

## Arquitectura

El backend sigue el patron **Hexagonal** (puertos y adaptadores):

- **Dominio** - Entidades, value objects, interfaces de repositorio
- **Aplicacion** - Casos de uso / servicios de dominio
- **Infraestructura** - Prisma, HTTP, WebSocket, integraciones externas

Cada bounded context (Auth, Inventory, Reservations, Guests, Operations, Housekeeping, Reporting, AI Assistant) es un modulo NestJS independiente.

## Modelo de IA

**Kimi (Moonshot AI)** con modelo `kimi-latest` via API compatible con OpenAI (`https://api.moonshot.cn/v1`). El asistente tiene herramientas de solo lectura del PMS: consultar disponibilidad, reportes, estado de habitaciones, huespedes y reservas.

## Documentacion

| Archivo | Contenido |
|---------|----------|
| `CLAUDE.md` | Guia completa del proyecto |
| `DEPLOY_ANALYSIS.md` | Analisis de deployment en Railway |
| `SETUP-LOCAL.md` | Configuracion local paso a paso |
| `design/DESIGN-SYSTEM.md` | Sistema de diseno y tokens |
| `.planning/` | Roadmap, requisitos y planificacion |

## Licencia

Propietario - Hotel Estoicos.