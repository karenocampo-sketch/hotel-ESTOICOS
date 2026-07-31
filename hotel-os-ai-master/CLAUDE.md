<!-- GSD:project-start source:PROJECT.md -->
## Proyecto

**Hotel Estoicos**

Hotel Estoicos es un **sistema operativo hotelero single-tenant** para un hotel especifico (no SaaS multi-hotel). Cubre las dos audiencias clasicas:

- **Staff del hotel** (PMS - Property Management System): recepcion, housekeeping, reservas internas, reportes y administracion.
- **Huespedes** (Booking Engine publico): busqueda de disponibilidad y reserva online.

El diferenciador es un **asistente conversacional con IA** (Kimi/Moonshot AI API - modelo `kimi-latest`) accesible para el staff con acceso a datos del PMS - consulta de disponibilidad, generacion de reportes en lenguaje natural, asistencia operativa.

**Valor Central:** > **Un staff de hotel debe poder gestionar reservas, check-in/out, habitaciones y operacion diaria desde una sola interfaz web, complementado por un asistente IA que responde preguntas del PMS y ayuda a tareas operativas.**

Si ese flujo no funciona, no hay producto.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Stack Tecnologico

## Decisiones Confirmadas
| Capa | Tecnologia | Estado |
|------|-----------|--------|
| Frontend | React 18 + TypeScript + Vite + Tailwind CSS | Confirmado |
| Backend | NestJS 11.x (Node.js) | Confirmado |
| ORM | Prisma 7.x | Confirmado |
| Base de datos | PostgreSQL 16 | Confirmado |
| Auth | JWT + refresh tokens (NestJS Passport + @nestjs/jwt) | Confirmado |
| Realtime | Socket.io 4.x (@nestjs/platform-socket.io) | Confirmado |
| IA | OpenAI SDK - `kimi-latest` (Kimi/Moonshot AI) | Confirmado (revisado 2026-06-12 - cambiado de OpenAI a Kimi) |
| Deploy | Railway | Confirmado |

## Stack Recomendado (completo)
### Backend Core
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `@nestjs/core` | 11.1.x | Kernel de aplicacion NestJS | Estable actual. v12 (migracion ESM) es Q3 2026 - empezar en v11 para evitar inestabilidad. |
| `@nestjs/platform-express` | 11.1.x | Adaptador HTTP (Express bajo NestJS) | Por defecto, probado. Fastify ofrece ganancias marginales no justificadas para un PMS single-tenant. |
| `@nestjs/platform-socket.io` | 11.1.x | Gateway WebSocket | Integracion nativa NestJS para Socket.io. |
| `@nestjs/jwt` | 11.x | Firma/verificacion JWT | Paquete oficial NestJS, funciona con Passport. |
| `@nestjs/passport` | 11.x | Andamiaje de estrategia auth | Patron estandar de auth NestJS. |
| `passport-jwt` | 4.x | Estrategia JWT para Passport | Estandar de la industria. |
| `@nestjs/config` | 4.x | Gestion de variables de entorno | Nativo NestJS, soporta validacion Joi de env vars. |
| `@nestjs/swagger` | 11.4.x | Documentacion OpenAPI | Genera docs de API desde decoradores. Critico para DX del equipo y alineacion de contratos con frontend. |
| `@nestjs/schedule` | latest | Trabajos cron | Para reportes nocturnos, recordatorios de reserva. No necesita Redis para programaciones simples. |
| `prisma` | 7.x | CLI ORM + migraciones | v7 es estable actual. Motor WASM sin Rust ahora listo para produccion en PostgreSQL. |
| `@prisma/client` | 7.x | Cliente de base de datos con tipos | Generado desde schema - la dependencia principal en runtime. |
| `zod` | 4.4.x | Validacion de schemas (compartido, backend + frontend) | v4 es estable y actual. Usar para validacion de request body en pipes NestJS y para tipos compartidos con frontend. Reemplaza `class-validator` para validacion pura. |
| `bcrypt` / `@types/bcrypt` | 5.x | Hash de contrasenas | Estandar. Usar bcrypt, no bcryptjs - los bindings nativos de Node.js son mas rapidos. |

### Frontend Core
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `react` | 18.x | Libreria de UI | Confirmado. NO actualizar a React 19 aun - la estabilidad del ecosistema para 18 es mayor. |
| `react-dom` | 18.x | Renderizador DOM | Pareado con React 18. |
| `vite` | 6.x | Herramienta de build | Major actual. Vite 6 soporta React 18 y TypeScript nativamente. |
| `@vitejs/plugin-react` | 4.x | Plugin React para Vite | Basado en SWC - mas rapido que Babel. Usar variante `plugin-react-swc`. |
| `tailwindcss` | 4.x | CSS utilitario | Tailwind v4 es actual. Nota: el formato de config cambio de `tailwind.config.js` a nativo CSS - usar v4 desde el dia uno. |
| `typescript` | 5.7.x | Seguridad de tipos | Estable actual para 2026. |

### Libreria de Componentes UI
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `shadcn/ui` | CLI-based (sin version de paquete) | Sistema de componentes | Enfoque copy-paste - los componentes viven en tu codigo fuente, no en una caja negra de node_modules. Basado en Radix UI. Nativo Tailwind. Perfecto para una UI hotelera personalizada. |
| `@radix-ui/*` | latest (manejado por shadcn CLI) | Primitivas headless | Primitivas con accesibilidad correcta para modales, dropdowns, tooltips, selects. |
| `class-variance-authority` | 0.7.x | Gestion de variantes para componentes | Peer dep estandar de shadcn/ui. |
| `clsx` + `tailwind-merge` | latest | Fusion condicional de classNames | Utilidades estandar de shadcn/ui. |
| `lucide-react` | 0.x latest | Libreria de iconos | Set de iconos por defecto de shadcn/ui. Consistente, tree-shakeable. |

### Calendario / Seleccion de Fechas (CRITICO para UX hotelera)
#### Calendario de Ocupacion (vista de grilla de habitaciones - el calendario del PMS)
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `@schedule-x/react` | latest | Calendario visual de timeline por habitacion | Alternativa moderna y framework-aware a react-big-calendar. Inyeccion nativa de componentes React, diseno responsivo, mejor DX en 2025+. Usar para la vista de timeline horizontal (habitaciones en eje Y, fechas en eje X - diseno clasico de PMS). |

#### Selector de Rango de Fechas para Reservas (booking engine publico + filtros de busqueda)
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `react-day-picker` | 10.x | Seleccion de rango de fechas | v10 es el estable actual (recien lanzado). Impulsa el propio componente Calendar de shadcn/ui. Usar directamente para el selector de rango check-in/check-out del motor de reservas. El `mode="range"` integrado es exactamente lo que necesita una reserva de hotel. |

### Gestion de Formularios
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `react-hook-form` | 7.75.x | Gestion de estado de formularios | 7.75.x es actual (mayo 2026). Componentes no controlados = minimos re-renders. Critico para formularios con muchos campos (CRUD de habitaciones, registro de huespedes). |
| `@hookform/resolvers` | 3.x | Conecta RHF con zod | Usar `zodResolver` - schema de validacion unico compartido entre backend (Zod pipes) y frontend. |
| `zod` | 4.4.x | Validacion de schemas | Ya listado en backend. Importar el mismo paquete de schema en ambos extremos para paridad de contratos. |

### Gestion de Estado
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `zustand` | 5.0.x | Estado global de UI | v5 es actual (mayo 2026). Usar para: usuario logueado actual, preferencias de UI (sidebar abierto/cerrado), filtros activos, habitacion seleccionada. NO usar para datos del servidor - eso pertenece a TanStack Query. |
| `@tanstack/react-query` | 5.100.x | Estado servidor/asincrono | v5 es actual. Maneja fetching, caching, refetch en segundo plano, actualizaciones optimistas. Esencial para los datos del PMS en tiempo real (lista de reservas, estado de habitaciones). |
| `@tanstack/react-query-devtools` | 5.x | Herramientas de desarrollo | Incluir solo en desarrollo. |

### Graficos / Visualizacion de Datos
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `recharts` | 2.x (v3 si esta estable) | Graficos de ocupacion, ingresos, KPIs del dashboard | 2.4M descargas semanales, API declarativa React, basado en SVG. El dashboard del hotel (%, ADR, tendencias de RevPAR) se mapea perfectamente a sus componentes LineChart, BarChart y PieChart. Los componentes de graficos de shadcn/ui estan construidos sobre Recharts - usarlos para mantener consistencia con el sistema de diseno. |

### Asistente IA (Streaming)
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `openai` | 4.x latest | Cliente de API compatible con OpenAI | Usa SDK de OpenAI con URL base de Kimi (Moonshot AI). Decision del equipo (revisado 2026-06-12 - cambiado a Kimi). Modelo: `kimi-latest`. Soporta streaming + function calling (tools) + salidas estructuradas. |
| Modelo: `kimi-latest` | - | LLM | Elegido para el asistente IA del staff. Kimi ofrece precios competitivos, fuerte soporte de espanol y ventana de contexto de 128k+. Soporte nativo para function calling. Usa API compatible con OpenAI via `https://api.moonshot.cn/v1`. |
| NestJS SSE (decorador `@Sse()`) | integrado | Stream de tokens al frontend | NestJS tiene soporte SSE nativo via respuesta compatible con `EventSource`. Pasar la respuesta streaming de OpenAI (`AsyncIterable<ChatCompletionChunk>`) a traves de un `Observable` de RxJS y retornar desde un endpoint `@Sse()`. La API EventSource del navegador NO soporta headers de Autorizacion - usar `fetch + ReadableStream` en el frontend con JWT en headers. |
| `eventsource-parser` | 2.x | Parsear stream SSE crudo en el cliente (si se necesita) | Solo se necesita para el frontend si hacemos parsing SSE manual. El flujo server-side de NestJS usa el AsyncIterable nativo del SDK directamente. |

### Almacenamiento de Archivos (Fotos de Habitaciones)
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `@aws-sdk/client-s3` | 3.726.x | Cliente S3-compatible para R2 | Cloudflare R2 es completamente S3-compatible. Fijar a 3.726.1 - v3.729.0 introdujo un comportamiento de checksum incompatible con R2. |
| `@aws-sdk/s3-request-presigner` | misma | URLs pre-firmadas para carga directa | Generar URLs de carga pre-firmadas en el backend; el frontend carga directamente a R2. Nunca proxear bytes de archivos a traves de NestJS. |

### Generacion de PDF (Reportes)
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `@react-pdf/renderer` | 4.5.x | Generar reportes PDF (ocupacion, ingresos, historial de huespedes) | v4.5.1 es actual (abril 2026). Corre en Node.js (no solo navegador). Usar enfoque de componentes React en el backend NestJS - `ReactPDF.renderToStream()` o `renderToBuffer()`. Genera PDFs bien estructurados y con estilo de forma declarativa. |

### Email (Transaccional)
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `resend` | 4.x | Confirmaciones de reserva, notificaciones de check-in | API-first, sin SMTP que administrar. Tier gratis: 3,000 emails/mes. Para un solo hotel, esto cubre todo el email transaccional indefinidamente. Configuracion en 5 minutos. Excelente SDK de TypeScript. |

### Tracking de Errores / Observabilidad
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `@sentry/nestjs` | 8.x | Tracking de errores backend + rendimiento | SDK oficial NestJS. Captura excepciones no manejadas con contexto completo de request, stack traces, breadcrumbs. |
| `@sentry/react` | 8.x | Tracking de errores frontend + rendimiento | Captura errores JS con stack traces de componentes. |

### Testing
| Tecnologia | Version | Proposito | Por que |
|------------|---------|-----------|---------|
| `vitest` | 4.1.x | Tests unitarios + integracion (frontend + backend) | v4.1.6 es actual (mayo 2026). Nativo de Vite, 4x mas rapido que Jest, cero config para TypeScript. Usar para tests de servicios NestJS (via `vitest` con `@nestjs/testing`) y tests de componentes React. |
| `@testing-library/react` | 16.x | Utilidades de testing de componentes | Testing estandar React. Consultar DOM por rol/label (no por detalles de implementacion). |
| `@testing-library/user-event` | 14.x | Interacciones de usuario simuladas | `userEvent` > `fireEvent` - mas cercano al comportamiento real del navegador. |
| `@testing-library/jest-dom` | 6.x | Matchers de asercion DOM | `toBeInTheDocument()`, `toHaveValue()`, etc. Funciona con Vitest via `@testing-library/jest-dom/vitest`. |
| `jsdom` | 26.x | Entorno de navegador para Vitest | Requerido para tests de componentes React en Node. |
| `@playwright/test` | 1.57.x | Tests E2E | Estable actual. Testear flujos criticos: creacion de reserva, check-in, disponibilidad del calendario. Solo para happy paths en v1 - no invertir de mas en cobertura E2E en etapa MVP. |
| `@nestjs/testing` | 11.x | Fabrica de modulo de prueba NestJS | Usar `Test.createTestingModule()` para testing unitario de controladores y servicios con DI. |
| `supertest` | 7.x | Aserciones HTTP para e2e de NestJS | Testear ciclos completos de request/response en NestJS sin iniciar un servidor. |

### Patrones de Arquitectura NestJS (Hexagonal)
| Patron | Implementacion | Por que |
|--------|---------------|---------|
| Abstraccion de repositorio | Interfase + clase de implementacion Prisma por bounded context | El cliente Prisma permanece en la capa `infrastructure/`. Los servicios de dominio dependen de la interfase, no de Prisma directamente. |
| Modulo por bounded context | `ReservationsModule`, `InventoryModule`, `GuestsModule`, etc. | Cada modulo encapsula las capas de dominio, aplicacion e infraestructura. Sin importaciones de Prisma entre modulos. |
| Validacion de DTOs | `class-transformer` + pipes Zod (o `nestjs-zod`) | Usar schemas Zod como fuente unica de verdad para la forma de los DTOs. |
| Guards para RBAC | `@nestjs/passport` + `RolesGuard` personalizado | Decorar controladores con `@Roles('admin', 'reception')`. |
| Validacion de config | `@nestjs/config` con schema Zod para env vars | Fallo rapido al inicio si faltan variables de entorno requeridas. |

## Alternativas Consideradas
| Categoria | Recomendada | Alternativa | Por que no |
|-----------|-------------|-------------|------------|
| SDK Asistente IA | OpenAI SDK + Kimi (`kimi-latest`) | OpenAI (gpt-4o-mini), Anthropic (Claude) | Decision del equipo revisada 2026-06-12: Cambiado a Kimi (Moonshot AI) por mejor soporte de espanol, precios competitivos y API compatible con OpenAI. Usa URL base `https://api.moonshot.cn/v1`. La config anterior con OpenAI gpt-4o-mini funcionaba pero Kimi ofrece mejor valor para operaciones hoteleras en espanol. |
| Componentes UI | shadcn/ui | MUI, Ant Design | Imponen sistemas de diseno incompatibles con el branding hotelero personalizado |
| Estado (global) | Zustand 5 | Redux Toolkit | Sobrecarga de boilerplate enorme para una app single-tenant |
| Estado (servidor) | TanStack Query 5 | SWR | TanStack Query tiene control de cache mas rico, mutaciones y devtools |
| Formularios | react-hook-form | Formik | RHF es consistentemente mas rapido (modelo no controlado) y ha superado a Formik |
| Graficos | Recharts | Tremor, Nivo | Tremor envuelve Recharts; los graficos de shadcn/ui ya envuelven Recharts - no necesitamos otra capa |
| PDF | @react-pdf/renderer | Puppeteer | Puppeteer requiere Chrome headless - demasiado pesado para contenedores de Railway |
| Email | Resend | Nodemailer | Sobrecarga de administracion SMTP no justificada para un solo hotel |
| Almacenamiento | Cloudflare R2 | AWS S3, Railway Volumes | R2 no tiene cargos de egress; Railway Volumes no son CDN-capables |
| ORM | Prisma 7 | Drizzle, TypeORM | TypeORM tiene problemas conocidos con relaciones complejas; Drizzle es prometedor pero ecosistema menos maduro; la seguridad de tipos de Prisma no tiene igual |
| Calendario (grilla) | @schedule-x | react-big-calendar | La vista de recurso/timeline de react-big-calendar requiere mas workarounds; schedule-x tiene mejor integracion con React |
| Testing (unitario) | Vitest | Jest | Vitest es 4x mas rapido, cero config para proyectos Vite y API compatible con Jest |

## Fijaciones de Versiones Clave
| Paquete | Razon de la Fijacion |
|---------|---------------------|
| `@aws-sdk/client-s3` | `3.726.1` - v3.729.0+ tiene bug de checksum incompatible con Cloudflare R2 |
| `react` | `18.x` (no 19) - La estabilidad del ecosistema React 19 aun no alcanza el nivel necesario para produccion |
| `@nestjs/*` | `11.x` (no 12) - v12 (migracion ESM completa) es Q3 2026 - empezar en el estable v11 |
| `openai` | `4.x latest` - SDK Node de OpenAI v4 (actual a 2026-05). Modelo: `kimi-latest` via URL base de Kimi/Moonshot AI. |

## Evaluacion de Confianza
| Area | Confianza | Notas |
|------|-----------|-------|
| Version NestJS | ALTA | Verificado: v11.1.19 actual, v12 Q3 2026 |
| Version Prisma | ALTA | Verificado: v7.x actual (v7.3+ a ene 2026), motor WASM sin Rust listo para produccion |
| react-hook-form | ALTA | Verificado: v7.75.x actual (mayo 2026) |
| Zustand | ALTA | Verificado: v5.0.13 actual |
| TanStack Query | ALTA | Verificado: v5.100.10 actual |
| Vitest | ALTA | Verificado: v4.1.6 actual |
| Zod | ALTA | Verificado: v4.4.3 actual |
| @nestjs/swagger | ALTA | Verificado: v11.4.2 actual |
| react-day-picker | ALTA | Verificado: v10.x recien lanzado; usar v10 |
| @react-pdf/renderer | ALTA | Verificado: v4.5.1 actual (abril 2026) |
| @schedule-x | MEDIA | Actual segun busqueda; verificar version exacta al instalar |
| Cloudflare R2 + aws-sdk pin | ALTA | Verificado via docs oficiales de Cloudflare R2 - fijar 3.726.1 |
| Resend | MEDIA | Actual segun busquedas 2025; verificar limites de tier gratis antes de comprometerse |
| Playwright | ALTA | Verificado: v1.57.x actual |
| Recharts | MEDIA | v2.x estable; v3 en progreso - empezar con v2 |
| OpenAI SDK (`openai`) | MEDIA | v4.x actual. Verificar disponibilidad + precios de gpt-4o-mini al instalar. |

## Fuentes
- Versiones NestJS: https://github.com/nestjs/nest/releases
- Changelog Prisma: https://www.prisma.io/changelog
- react-hook-form npm: https://www.npmjs.com/package/react-hook-form
- Zustand npm: https://www.npmjs.com/package/zustand
- TanStack Query npm: https://www.npmjs.com/package/@tanstack/react-query
- Vitest: https://vitest.dev/
- Zod v4: https://zod.dev/v4
- @nestjs/swagger npm: https://www.npmjs.com/package/@nestjs/swagger
- react-day-picker v10: https://daypicker.dev/changelog
- @react-pdf/renderer npm: https://www.npmjs.com/package/@react-pdf/renderer
- Cloudflare R2 + aws-sdk-js-v3: https://developers.cloudflare.com/r2/examples/aws/aws-sdk-js-v3/
- Playwright releases: https://github.com/microsoft/playwright/releases
- Sentry para NestJS: https://docs.sentry.io/platforms/javascript/guides/nestjs/
- shadcn/ui date picker: https://ui.shadcn.com/docs/components/radix/date-picker
- Schedule-X: https://github.com/schedule-x/schedule-x
- Streaming + function calling OpenAI: https://platform.openai.com/docs/api-reference/streaming
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Convenciones

Convenciones aun no establecidas. Se completaran a medida que se establezcan patrones durante el desarrollo.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Arquitectura

Arquitectura aun no mapeada. Seguir los patrones existentes encontrados en el codigo fuente.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Habilidades del Proyecto

No se encontraron habilidades del proyecto. Agregar habilidades en cualquiera de: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/` o `.codex/skills/` con un archivo indice `SKILL.md`.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## Ejecucion del Flujo de Trabajo GSD

Antes de usar Edit, Write u otras herramientas que cambien archivos, iniciar el trabajo a traves de un comando GSD para que los artefactos de planificacion y el contexto de ejecucion se mantengan sincronizados.

Usar estos puntos de entrada:
- `/gsd-quick` para arreglos pequenos, actualizaciones de docs y tareas ad-hoc
- `/gsd-debug` para investigacion y correccion de bugs
- `/gsd-execute-phase` para trabajo planificado por fases

No hacer ediciones directas al repositorio fuera de un flujo de trabajo GSD a menos que el usuario solicite explcitamente omitirlo.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Perfil del Desarrollador

> Perfil aun no configurado. Ejecutar `/gsd-profile-user` para generar tu perfil de desarrollador.
> Esta seccion es gestionada por `generate-claude-profile` -- no editar manualmente.
<!-- GSD:profile-end -->