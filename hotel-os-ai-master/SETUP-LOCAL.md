# Hotel Estoicos — Plan de Configuración Local

**Versión:** 1.0
**Fecha:** 2026-07-30
**Stack:** NestJS + React (Monorepo pnpm/Turborepo) + PostgreSQL + Prisma

---

## 1. Requisitos del Sistema

| Requisito | Versión Mínima | Estado Actual |
|-----------|---------------|---------------|
| Node.js   | >= 20         | v25.6.1 |
| pnpm      | >= 9          | v11.18.0 |
| Docker    | Cualquiera    | Disponible |
| Git       | Cualquiera    | Disponible |
## 2. Estructura del Proyecto

```
hotel-estoicos-master/
+-- apps/
|   +-- api/          # NestJS backend (puerto 3001)
|   |   +-- src/
|   |   +-- prisma/   # Schema + 27 migraciones
|   |   +-- .env.example
|   +-- web/          # React + Vite frontend (puerto 5173)
+-- packages/
|   +-- shared/       # Codigo compartido (types, utils)
+-- docker-compose.yml
+-- package.json      # Root workspace
+-- pnpm-workspace.yaml
+-- turbo.json
```

---

## 3. Configuracion Paso a Paso

### Paso 1: Variables de Entorno

Crear apps/api/.env desde apps/api/.env.example:

```powershell
Copy-Item apps/api/.env.example apps/api/.env
```

Editar apps/api/.env con valores reales:

| Variable | Valor Local | Notas |
|----------|------------|-------|
| DATABASE_URL | postgresql://user:password@localhost:5432/hoteldb | Usuario/contra de docker-compose |
| JWT_ACCESS_SECRET | 32+ chars aleatorios | Usar node -e para generar |
| JWT_REFRESH_SECRET | 32+ chars aleatorios | Idem |
| RESEND_API_KEY | re_xxxx | Opcional para dev |
| OPENAI_API_KEY | Tu API key | Opcional sin IA |
| CONCIERGE_IP_HASH_SALT | 32+ chars aleatorios | Opcional dev |
| REVIEW_TOKEN_SECRET | 32+ chars aleatorios | Opcional dev |

### Paso 2: Base de Datos (PostgreSQL via Docker)

Opcion A - Usar docker-compose.yml existente:
```powershell
docker compose up -d db
```

Opcion B - Contenedor manual:
```powershell
docker run -d --name Hotel Estoicos-db -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -e POSTGRES_DB=hoteldb -p 5432:5432 postgres:16-alpine
```

Verificar conexion:
```powershell
docker ps | findstr postgres
```
### Paso 3: Migraciones de Prisma

```powershell
# Aplicar migraciones a la DB local
pnpm --filter @hotel/api exec prisma migrate deploy

# (Opcional) Sembrar datos de prueba
pnpm --filter @hotel/api exec prisma db seed
```

Solucion de problemas:
- Error de conexion: Verificar que Docker este corriendo y DATABASE_URL sea correcta
- Error de migracion: prisma migrate reset borra datos y re-ejecuta

### Paso 4: Iniciar Servidores de Desarrollo

Terminal 1 - API (NestJS):
```powershell
pnpm --filter @hotel/api start:dev
# Escucha en: http://localhost:3001
# Health check: http://localhost:3001/health
```

Terminal 2 - Web (React + Vite):
```powershell
pnpm --filter @hotel/web dev
# Escucha en: http://localhost:5173
```

Alternativa - Todo en uno con Turborepo:
```powershell
pnpm dev
```

### Paso 5: Verificar que Todo Funciona

```powershell
# 1. API responde
curl http://localhost:3001/health

# 2. Web carga
curl http://localhost:5173

# 3. Prisma conecta
pnpm --filter @hotel/api exec prisma db push --dry-run

# 4. Tests unitarios pasan
pnpm test
```
---

## 4. Comandos Utiles

| Comando | Descripcion |
|---------|-------------|
| pnpm dev | Inicia API + Web en paralelo (Turborepo) |
| pnpm build | Build de todos los paquetes |
| pnpm test | Tests de todo el monorepo |
| pnpm lint | Lint de todos los paquetes |
| pnpm --filter @hotel/api start:dev | Solo API con hot-reload |
| pnpm --filter @hotel/web dev | Solo Web con hot-reload |
| pnpm --filter @hotel/api exec prisma studio | Prisma Studio UI |
| pnpm --filter @hotel/api exec prisma migrate dev | Nueva migracion |

---

## 5. Troubleshooting Comun

### "pnpm no se reconoce"
```powershell
npm install -g pnpm
```

### "Puerto 3001 en uso"
```powershell
netstat -ano | findstr :3001
# Luego taskkill /PID <PID> /F
```

### "ECONNREFUSED PostgreSQL"
```powershell
docker ps
docker logs Hotel Estoicos-db
```

### "Migraciones fallan - relacion ya existe"
```powershell
pnpm --filter @hotel/api exec prisma migrate reset
```

---

## 6. Stack Tecnico (Referencia)

| Capa | Tecnologia | Proposito |
|------|-----------|-----------|
| Backend | NestJS + Express | API REST |
| Frontend | React + Vite + Tailwind | SPA |
| Database | PostgreSQL + Prisma | Persistencia + ORM |
| Auth | JWT (access + refresh) | Autenticacion |
| Cache/Uploads | Cloudflare R2 | Archivos de huespedes |
| Email | Resend | Notificaciones |
| AI | OpenAI GPT-4o-mini | Asistente staff + conserje |
| Monorepo | Turborepo + pnpm | Orquestacion |
| Tiempo Real | Socket.io | Housekeeping |