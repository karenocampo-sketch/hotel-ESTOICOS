Área,Confianza,Notas
Versión de NestJS,ALTA,"Verificado: v11.1.19 actual, v12 Q3 2026."
Versión de Prisma,ALTA,"Verificado: v7.x actual (v7.3+ a enero de 2026), motor WASM sin Rust listo para producción."
react-hook-form,ALTA,Verificado: v7.75.x actual (mayo de 2026).
Zustand,ALTA,Verificado: v5.0.13 actual.
TanStack Query,ALTA,Verificado: v5.100.10 actual.
Vitest,ALTA,Verificado: v4.1.6 actual.
Zod,ALTA,Verificado: v4.4.3 actual.
@nestjs/swagger,ALTA,Verificado: v11.4.2 actual.
react-day-picker,ALTA,Verificado: v10.x recién lanzado; usar v10.
@react-pdf/renderer,ALTA,Verificado: v4.5.1 actual (abril de 2026).
@schedule-x,MEDIA,Actual según búsqueda; verificar la versión exacta al instalar.
Cloudflare R2 + aws-sdk pin,ALTA,Verificado a través de la documentación oficial de Cloudflare R2 — fijar en 3.726.1.
Resend,MEDIA,Actual según búsquedas de 2025; límites del nivel gratuito a verificar antes de comprometerse.
Playwright,ALTA,Verificado: v1.57.x actual.
Recharts,MEDIA,v2.x estable; v3 en curso — comenzar con v2.
SDK de OpenAI (openai),MEDIA,v4.x actual. Verificar disponibilidad de kimi-latest + precios al instalar.

Referencia de Instalación
Backend — NestJS
Bash
# Crear proyecto o inicializar dependencias principales
npm i @nestjs/core@11.1.x @nestjs/platform-express@11.1.x @nestjs/platform-socket.io@11.1.x @nestjs/jwt@11.x @nestjs/passport@11.x passport-jwt@4.x @nestjs/config@4.x @nestjs/swagger@11.4.x @nestjs/schedule@latest reflect-metadata rxjs

# ORM y Base de Datos
npm i prisma@7.x @prisma/client@7.x

# Validación y Utilidades
npm i zod@4.4.x bcrypt@5.x
npm i -D @types/bcrypt

# IA, PDF y Almacenamiento
npm i openai@latest @aws-sdk/client-s3@3.726.1 @aws-sdk/s3-request-presigner@3.726.1 @react-pdf/renderer@4.5.x resend@4.x

# Observabilidad y Pruebas
npm i @sentry/nestjs@8.x
npm i -D vitest@4.1.x @nestjs/testing@11.x supertest@7.x
Frontend — React (Vite + TypeScript)
Bash
# Dependencias principales de UI y Estado
npm i react@18.x react-dom@18.x zustand@5.0.x @tanstack/react-query@5.100.x

# Formularios y Validación
npm i react-hook-form@7.75.x @hookform/resolvers@3.x zod@4.4.x

# Enrutamiento, Calendario e Iconos
npm i @schedule-x/react@latest react-day-picker@10.x lucide-react@latest clsx tailwind-merge class-variance-authority@0.7.x @radix-ui/react-slot

# Visualización de Datos
npm i recharts@2.x

# Observabilidad y Pruebas
npm i @sentry/react@8.x
npm i -D vite@6.x @vitejs/plugin-react-swc@4.x tailwindcss@4.x typescript@5.7.x vitest@4.1.x @testing-library/react@16.x @testing-library/user-event@14.x @testing-library/jest-dom@6.x jsdom@26.x @playwright/test@1.57.x
shadcn/ui (Instalación mediante CLI)
Bash
npx shadcn@latest init
# Añadir componentes esenciales según se necesiten:
npx shadcn@latest add button calendar card dialog dropdown-menu input label popover select table tabs textarea toast
Convenciones
Las convenciones de código y estructura se irán formalizando y poblando a medida que emerjan patrones estables durante el desarrollo inicial del módulo core.

Naming: Convención camelCase para variables y métodos, PascalCase para clases, interfaces y componentes de React, y kebab-case para nombres de archivos y rutas.

Commits: Uso de Conventional Commits (feat:, fix:, refactor:, docs:, etc.).

Arquitectura
Patrón Modular: Organizado por contextos delimitados (bounded contexts), manteniendo separación estricta entre Dominio, Aplicación e Infraestructura.

Comunicación en Tiempo Real: Eventos mediante Socket.io para sincronización instantánea de cambios de estado en habitaciones y reservas entre el personal de recepción y limpieza.

Flujo de IA: Las consultas del asistente del staff se procesan en el backend exponiendo un endpoint SSE que conecta de forma segura con la API de Kimi (kimi-latest), inyectando contexto dinámico del PMS mediante llamadas a funciones (tool calling).

Cumplimiento del Flujo de Trabajo GSD
Antes de utilizar comandos de edición, escritura o cualquier herramienta que modifique archivos en el repositorio, el trabajo debe iniciarse a través de un comando GSD para mantener sincronizados los artefactos de planificación y el contexto de ejecución:

/gsd-quick: Para correcciones menores, actualizaciones de documentación y tareas puntuales.

/gsd-debug: Para investigación y resolución de errores (bug fixing).

/gsd-execute-phase: Para la ejecución planificada de fases de trabajo.

Restricción: No realizar ediciones directas en el repositorio fuera de un flujo GSD a menos que el usuario solicite explícitamente omitirlo.

Perfil del Desarrollador
Perfil no configurado. Ejecute /gsd-profile-user para generar su perfil de desarrollador personalizado.

¿Con qué fase o módulo específico de Hotel Estoicos te gustaría que comencemos a trabajar o planificar ahora (por ejemplo, el esquema de la base de datos con Prisma, la estructura de carpetas del backend con NestJS, o el motor de reservas del
