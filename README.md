# BYMovers – Plataforma integral para mudanzas y guardamuebles

## Visión general
BYMovers es una aplicación web full-stack multiusuario que ofrece a una empresa de mudanzas y guardamuebles todas las herramientas necesarias para gestionar operaciones, equipos y clientes en tiempo real. La solución cubre la planificación de servicios, la creación de presupuestos inteligentes, la gestión de personal y vehículos, el control de inventario y finanzas, y la comunicación instantánea entre oficina, comerciales y equipos sobre el terreno.

## Arquitectura tecnológica
- **Backend:** Node.js + Express con TypeScript.
- **Base de datos:** PostgreSQL en producción, SQLite en desarrollo, orquestado con Prisma ORM.
- **Frontend:** React + Vite + Tailwind CSS, preparado como PWA responsive con soporte offline básico para operarios.
- **Autenticación:** JWT (tokens de acceso y refresco) con rotación de refresh tokens y protección por roles.
- **Tiempo real:** Socket.IO para sincronización de cambios entre oficina, comerciales y equipos.
- **Mapas y rutas:** Leaflet sobre OpenStreetMap, con adaptador para integrar OSRM/OpenRouteService y soporte opcional para claves de Google Maps.
- **Almacenamiento de ficheros:** sistema local en desarrollo y servicio S3-compatible en producción.
- **Contenedores:** `docker-compose` con servicios de API, base de datos y frontend.
- **Seguridad y calidad:** bcrypt para hashing, rate limiting en `/auth`, validaciones con Zod, middleware RBAC, logging estructurado (pino/winston) y tests básicos.

## Roles y permisos
- **Admin:** acceso completo.
- **Oficina:** agenda, presupuestos, facturación, RRHH, inventario, guardamuebles.
- **Comercial:** leads, presupuestos, visitas, contratos.
- **Encargado:** asignación de roles en servicios, partes, incidencias, materiales.
- **Operario/Conductor:** jornada, marcaje de horas, checklist vehículo, incidencias, fotos, firma de cliente.

## Módulos funcionales clave
1. **Calendario unificado:** vistas día/semana/mes con trabajos programados, origen/destino, horas, equipo, vehículo, estado y filtros por equipo, vehículo, zona o estado. Permite drag & drop (opcional) para replanificar.
2. **Presupuestos inteligentes:** motor de cálculo con plantilla (ver sección "Cálculo"), estados (borrador, enviado, aceptado, rechazado, caducado), generación de PDF y envío por email. Desencadena visitas comerciales cuando el estimado supera 700 €.
3. **Sincronización Oficina ↔ Comerciales:** cambios de leads, presupuestos y trabajos en tiempo real mediante sockets y notificaciones.
4. **Mapa interactivo:** visualización de origen, destino y paradas, botón "ver ruta" y cálculo de distancia/tiempo. Geocodificación mediante Nominatim/ORS.
5. **Organización de equipos:** asignación de roles (conductor, montador, peón, etc.), vehículos y materiales previstos por servicio.
6. **Mantenimiento de vehículos:** avisos automáticos para ITV, seguro, revisiones, cambios de aceite y filtros con alertas programadas.
7. **Tacógrafo:** repositorio de ficheros (.ddd u otros) con metadatos y registro de lectura.
8. **Salud laboral y RRHH:** gestión de fichajes, ausencias, revisiones médicas, incidencias y calendario de ausencias.
9. **Avisos de placas/vallas:** recordatorios para reservas de aparcamiento con seguimiento de solicitud, colocación y retirada.
10. **Checklist de materiales:** control de carga/descarga por equipo, incidencias de inventario y accesibilidad desde tablets o PDA.
11. **Guardamuebles completo:** clientes, metros cúbicos ocupados, ubicación interna detallada, fotos/etiquetas y facturación periódica con alertas de impagos.
12. **Finanzas integradas:** reservas, transferencias, albaranes, facturas con gestión de IVA/retenciones, cobros/pagos, balance con km, combustible y costes. Exportación CSV/Excel.
13. **Inventario:** consumo por trabajo, stocks mínimos, valoración y alertas de reposición.
14. **PWA operativa para operarios:** jornada diaria, checklist, fotos y firma con soporte offline mínimo.

## Cálculo de presupuestos
Inputs principales: m³ estimados, pisos y ascensor (origen/destino), distancia en km, zona/código postal, tiempos de acceso de carga/descarga, número de operarios, vehículo requerido, embalaje, subidas sin ascensor, permisos (placas/vallas), guardamuebles (m³/mes), extras (muebles especiales, piano, desmontaje/montaje).

Fórmula orientativa:
```
Precio = Base_por_m3 * m3 + ManoObra_horas * tarifa_operario * nº_operarios + Km * €/km + Embalaje (material + tiempo) + Extras + Margen
```
Reglas y ajustes: redondeo mínimo, precio de salida, recargos por fines de semana/festivos y accesos difíciles, descuentos promocionales opcionales y desglose detallado.

## Entidades principales
- `users`, `auth_tokens`
- `clientes`
- `presupuestos`
- `trabajos`
- `asignaciones`
- `vehiculos`
- `tacografo_archivos`
- `materiales`, `materiales_trabajo`
- `incidencias`
- `firmas`
- `guardamuebles_clientes`, `guardamuebles_unidades`, `guardamuebles_contratos`, `guardamuebles_facturas`
- `rrhh_fichajes`, `rrhh_ausencias`, `rrhh_medicas`
- `recordatorios`
- `finanzas_reservas`, `finanzas_albaranes`, `finanzas_facturas`, `finanzas_movimientos`, `finanzas_balance_mensual`
- `rutas_cache`

## API REST mínima (JWT + RBAC)
- `/auth`: register, login, refresh, logout.
- `/clientes`: CRUD y búsqueda por nombre o código postal.
- `/presupuestos`: creación con cálculo, listado, cambio de estado y envío de PDF.
- `/trabajos`: CRUD, cambio de estado, subrecursos `/asignaciones`, `/incidencias`, `/materiales`.
- `/vehiculos`: CRUD, subrecursos `/mantenimientos` y `/tacografo`.
- `/rrhh`: fichajes, ausencias, revisiones médicas.
- `/guardamuebles`: clientes, unidades, contratos y facturación.
- `/inventario`: materiales, stocks mínimos y consumos.
- `/finanzas`: reservas, albaranes, facturas, movimientos y balance.
- `/recordatorios`: CRUD de placas, visitas, mantenimientos y revisiones médicas.
- `/rutas`: cálculo de rutas con uso de caché.

## Frontend (React + Tailwind)
- **Dashboard Oficina:** visión diaria/semanal, trabajos, avisos de caducidades, placas, revisiones médicas y cobros pendientes.
- **Calendario interactivo:** vista día/semana/mes, filtros, creación de trabajo desde huecos y acceso a la ficha de trabajo.
- **Gestión de presupuestos:** formulario con cálculo automático, desglose, PDF, envío y control de estados; trigger de visita comercial cuando corresponda.
- **Mapa Leaflet:** visualización de rutas y geocodificación de direcciones.
- **Ficha de trabajo:** asignaciones, materiales, incidencias, fotos y firma del cliente, con botones de cambio de estado.
- **Equipos y RRHH:** fichajes, ausencias y revisiones médicas con alertas visuales.
- **Vehículos:** estado general, kilometraje, caducidades, histórico de mantenimiento y subida de tacógrafo.
- **Inventario:** stock, mínimos, movimientos y modo checklist para carga.
- **Guardamuebles:** mapa interno/tabla, contratos y facturación periódica, alertas de impagos.
- **Finanzas:** reservas, albaranes, facturas, movimientos, balance mensual y exportaciones CSV/Excel.
- **Notificaciones en vivo:** actualizaciones de trabajos, incidencias y vencimientos vía Socket.IO.
- **PWA operarios:** vista “Mi jornada”, checklist del vehículo, incidencias, fotos y firma con sincronización offline mínima.

## Automatizaciones y alertas
- Trigger automático de visita comercial para presupuestos > 700 €.
- Alertas programadas (30/15/7 días) para ITV, seguros, revisiones de vehículos y revisiones médicas.
- Facturación mensual y avisos de impago en guardamuebles.
- Recordatorios de placas/vallas antes del servicio y el mismo día a las 07:00.
- Ajuste automático de inventario al completar trabajos, generando alertas al caer por debajo del stock mínimo.
- Checklist obligatorio del vehículo antes del fichaje de salida de un conductor.
- Caché de rutas para acelerar cálculos futuros.

## Plan por fases
**Fase 1 (MVP – 2 semanas):** autenticación y roles, gestión de clientes, calendario de trabajos, ficha básica de trabajos (asignaciones, incidencias, estados), presupuestos con cálculo inicial y PDF, mapa OSM, inventario mínimo, PWA “Mi jornada” y sockets para cambios de estado.

**Fase 2 (2–3 semanas):** mantenimiento de vehículos y alertas, RRHH (fichajes, ausencias, revisiones médicas), checklist de materiales, guardamuebles (estructura y contratos), exportación CSV/Excel y balance básico.

**Fase 3 (2–3 semanas):** facturación completa (reservas, albaranes, facturas), movimientos de costes (km, combustible, peajes), guardamuebles recurrente, disparador de visitas >700 €, modo offline extendido y gestión de adjuntos (fotos, tacógrafo).

## Semillas y documentación
- Seeds iniciales: 10 clientes, 15 trabajos, 5 vehículos, 12 materiales y 6 usuarios (uno por rol).
- README con `.env` de ejemplo y `docker-compose` para levantar API, base de datos y frontend.
- Tests sugeridos: autenticación, cálculo de presupuestos y ciclo de vida de trabajos.

