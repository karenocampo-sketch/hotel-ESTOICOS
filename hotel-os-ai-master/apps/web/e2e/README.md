# Tests E2E (Playwright)

Suite de tests end-to-end para la aplicacion web de Hotel Estoicos.

## Prerequisitos

1. **Node.js 20+** y **pnpm 10.29+**
2. **PostgreSQL** ejecutando con datos de seed
3. **Navegador Chromium** instalado para Playwright

### Configuracion inicial

```bash
# Desde la raiz del repositorio
pnpm install

# Instalar navegadores de Playwright (solo chromium)
cd apps/web
npx playwright install --with-deps chromium

# Poblar la base de datos (requiere env vars del API configuradas)
cd ../api
pnpm seed:admin
```

### Variables de entorno

| Variable | Valor por defecto | Descripcion |
|---|---|---|
| `E2E_ADMIN_EMAIL` | `admin@hotelsumapaz.co` | Email del usuario admin para tests de login |
| `E2E_ADMIN_PASSWORD` | `Admin123!` | Contrasena del usuario admin |
| `E2E_BASE_URL` | `http://localhost:4173` | URL del frontend (Vite preview) |
| `E2E_API_URL` | `http://localhost:3001` | URL del API para llamadas directas en fixtures |

## Ejecucion local

### Opcion A: Build de produccion (recomendado, igual a CI)

```bash
# 1. Construir el frontend
pnpm --filter @hotel/web build

# 2. Iniciar el API en una terminal separada
cd apps/api
pnpm dev

# 3. Ejecutar tests E2E (inicia automaticamente el servidor Vite preview)
cd apps/web
npx playwright test
```

### Opcion B: Contra el servidor de desarrollo

```bash
# 1. Iniciar ambos servidores (API y Web)
pnpm dev

# 2. Ejecutar tests E2E contra el servidor de desarrollo
cd apps/web
E2E_BASE_URL=http://localhost:5180 npx playwright test
```

### Comandos utiles

```bash
# Ejecutar un archivo de test especifico
npx playwright test smoke-responsive

# Ejecutar en modo headed (ver el navegador)
npx playwright test --headed

# Ejecutar con UI de Playwright (depurador interactivo)
npx playwright test --ui

# Listar todos los tests sin ejecutarlos
npx playwright test --list

# Ver el reporte HTML despues de una ejecucion
npx playwright show-report
```

## Archivos de test

| Archivo | QSI | Descripcion |
|---|---|---|
| `smoke-responsive.spec.ts` | QSI-06 | Portal publico en 4 viewports: sin scroll horizontal, hero visible, CTA alcanzable |
| `login-dashboard-logout.spec.ts` | QSI-07 | Ciclo completo de auth: formulario login -> dashboard -> logout del sidebar -> portal |
| `reservation-wizard.spec.ts` | QSI-08 | Wizard de reserva de 4 pasos: abrir, navegar pasos, cerrar |
| `calendar-drag-to-move.spec.ts` | QSI-09 | Room rack DnD: chips arrastrables, celdas de grilla, interaccion drag-and-drop |
| `error-boundaries.spec.ts` | QSI-10 | Rutas desconocidas, fallos del API: sin pantallas blancas |

## Comportamiento en CI (QSI-11)

El job de E2E corre en `.github/workflows/ci.yml` como un job separado (`e2e`) que
depende del job principal `ci`. Solo se activa en pull requests para ahorrar
minutos de CI en pushes directos a master.

Pasos:
1. Checkout + instalar dependencias
2. Construir la app web (`pnpm --filter @hotel/web build`)
3. Instalar chromium de Playwright
4. Iniciar un contenedor de servicio PostgreSQL
5. Ejecutar migraciones de Prisma + seed del usuario admin
6. Iniciar API + Vite preview en segundo plano
7. Ejecutar `npx playwright test`
8. Subir reporte HTML como artifact en caso de fallo

Reintentos: 1 reintento en CI para absorber flakes. 0 reintentos localmente.

## Atributos data-testid

Los siguientes atributos `data-testid` fueron agregados a los componentes de produccion
para seleccion E2E confiable. Cada uno esta documentado y es minimal:

| Componente | Atributo | Proposito |
|---|---|---|
| `RoomRackTable.tsx` | `data-testid="rack-grid"` | Contenedor grilla externo - verifica que el calendario se renderizo |
| `RoomRackTable.tsx` | `data-testid="rack-cell"` | Cada celda de dia - destino de drop para DnD |
| `RoomRackTable.tsx` | `data-testid="rack-event-chip"` | Chip de reserva - fuente de arrastre para DnD |

## Solucion de problemas

**Los tests fallan con "Login failed"**: Asegurarse de que el API esta ejecutando y que el usuario admin esta configurado con las credenciales que coinciden con `E2E_ADMIN_EMAIL` / `E2E_ADMIN_PASSWORD`.

**Vite preview muestra pagina en blanco**: Ejecutar `pnpm --filter @hotel/web build` primero. El servidor preview sirve el directorio `dist/` que debe existir.

**Test de drag-and-drop saltado**: El test de DnD del calendario requiere al menos una reserva en la ventana de 30 dias actual. Configurar reservas via `seed-phase12` o crear una manualmente.