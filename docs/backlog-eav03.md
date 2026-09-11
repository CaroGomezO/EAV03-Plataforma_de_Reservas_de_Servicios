# Product Backlog — EAV03 · Plataforma de Reservas de Servicios

Copia legible y versionada del backlog que vive en Azure DevOps
(`dev.azure.com/FabricaEscuela20262/EAV03`). La fuente de verdad son los work items;
este documento es para no perder el contexto y las decisiones al cambiar de máquina
o cerrar el editor.

- **Proceso:** Agile. **Team:** EAV03 Team. Todo cuelga de la raíz `EAV03` (aún sin sprints).
- **Códigos de título:** `E-01`, `FE-001`, `HU-0001` (separador ` · `).
- **Estado a 2026-09-11:** 5 épicas · 15 features · 28 historias, todas en estado *New*,
  con criterios de aceptación en Gherkin (en la descripción y en el campo Acceptance Criteria).
- **Terminología:** el administrador de cada negocio se llama **"propietario"**. Ya
  **no existe** el rol "administrador de la plataforma" en el MVP: el registro de
  negocios es 100% autoservicio (ver punto 8 del historial de cambios).
- Redactado con el modelo **INVEST + Gherkin** (skill `historias-usuario-invest`), sin
  tecnicismos y sin inventar reglas: lo que falta por decidir está en *Preguntas abiertas*.

---

## 1. Estructura del MVP

### E-01 · Acceso e Identidad
- **FE-001 · Registro y autenticación**
  - HU-0001 · Registrarme como cliente en la plataforma
  - HU-0041 · Registrarme como propietario
  - HU-0002 · Iniciar sesión en mi cuenta
  - HU-0003 · Recuperar el acceso a mi cuenta
- **FE-016 · Registro y activación de negocios**
  - HU-0042 · Crear mi negocio

### E-02 · Administración del Negocio
- **FE-002 · Gestión del catálogo de servicios**
  - HU-0004 · Registrar un servicio en el catálogo
  - HU-0005 · Editar un servicio del catálogo
  - HU-0006 · Activar o desactivar un servicio
- **FE-003 · Gestión de proveedores del servicio**
  - HU-0007 · Registrar un proveedor
  - HU-0008 · Asignar un servicio a un proveedor
  - HU-0009 · Desactivar un proveedor
- **FE-004 · Gestión de recursos**
  - HU-0010 · Registrar un recurso
  - HU-0011 · Editar o retirar un recurso
- **FE-005 · Gestión de políticas de reserva y cancelación**
  - HU-0012 · Configurar los plazos de reserva y cancelación de un servicio
  - HU-0013 · Configurar los límites de reservas y reprogramaciones

### E-03 · Gestión de Agendas y Disponibilidad
- **FE-006 · Configuración de horarios de atención del proveedor**
  - HU-0014 · Definir mi horario semanal de atención
- **FE-007 · Definición de capacidad de un servicio**
  - HU-0015 · Definir la capacidad simultánea de un servicio

### E-04 · Reservas del Cliente
- **FE-008 · Búsqueda y consulta de disponibilidad**
  - HU-0016 · Consultar franjas disponibles de un servicio
  - HU-0017 · Filtrar la disponibilidad por fecha, proveedor o modalidad
- **FE-009 · Creación de una reserva**
  - HU-0018 · Reservar una franja disponible de un servicio
  - HU-0019 · Consultar mis reservas y su detalle
- **FE-010 · Modificación o reprogramación de una reserva**
  - HU-0020 · Reprogramar una reserva a otra franja disponible
- **FE-011 · Cancelación de una reserva**
  - HU-0021 · Cancelar una reserva propia

### E-05 · Gestión Operativa de Reservas
- **FE-012 · Visualización y control de la agenda del proveedor**
  - HU-0022 · Consultar mi agenda del día
  - HU-0023 · Consultar el detalle de una reserva de mi agenda
- **FE-013 · Registro de asistencia y no-show**
  - HU-0024 · Registrar la asistencia de un cliente a su cita
  - HU-0025 · Registrar una inasistencia (no-show)
- **FE-014 · Creación de reservas a nombre del cliente**
  - HU-0026 · Crear una reserva a nombre de un cliente

---

## 2. Reglas de negocio vigentes (decisiones del equipo)

### Multi-negocio e identidad
- La plataforma es **multi-negocio**: cada negocio (tenant) tiene sus datos
  completamente aislados de los demás. **Ya no existe el rol "administrador de la
  plataforma"** en el MVP (eliminado 2026-09-11).
- **Registro 100% autoservicio, sin fricción.** No hay ningún admin ni intermediario
  que apruebe la creación de una cuenta o de un negocio: quedan activos de inmediato.
- El **propietario** se registra solo (HU-0041) y queda autenticado de inmediato.
  En el mismo flujo es redirigido a crear su negocio (HU-0042: nombre, dirección,
  identificación fiscal única, moneda base ISO 4217), que se activa sin aprobación
  de nadie. El "Usuario Propietario" y el "Negocio" son entidades **completamente
  separadas** a nivel de datos y arquitectura, aunque la UX las encadena en un
  único flujo (sin romper la experiencia con dos pasos desconectados).
- Un negocio tiene un único propietario, pero **un propietario puede tener varios
  negocios** con la misma cuenta (confirmado 2026-09-11). El propietario queda
  **bloqueado del resto de la plataforma** hasta crear al menos su primer negocio.
  **No hay mecanismo de desactivación de negocios en el MVP** (esa capacidad se
  fue junto con el rol de administrador de la plataforma).
- Cada negocio tiene su propio catálogo, proveedores, recursos y políticas, aislados
  de los demás; los gestiona su propietario.
- **Cliente:** registro autoservicio. Identificador = correo. Datos obligatorios:
  nombre completo, correo, celular, cédula, dirección de vivienda. Sin verificación
  previa. Sin inicio de sesión con proveedores externos.
- **Login y recuperación** (HU-0002 / HU-0003): mismo mecanismo para cualquier cuenta.
  Bloqueo tras **5 intentos fallidos en 15 min**. Sesión expira a los **30 min** de
  inactividad. Recuperación por **código OTP al correo, válido 5 min**.
- No hay roles internos que administrar: un usuario puede ser cliente, proveedor
  y/o propietario de un negocio a la vez.

### Catálogo, proveedores y recursos
- **Servicio:** nombre, duración, descripción, modalidad, precio. Distintas
  duraciones/modalidades = servicios distintos. Un servicio puede tener **uno o
  varios proveedores**; necesita ≥1 para ser reservable.
- **Proveedor:** nombre completo, ocupación, celular, correo, contraseña. Su cuenta
  de acceso se crea automáticamente al registrarlo (lo hace el propietario del negocio).
- **Recurso:** nombre, tipo, cantidad. Asociado a **un único servicio**, no
  compartible. La **cantidad del recurso = capacidad simultánea del servicio**:
  límite duro, sin sobrecupo, sin reservas solapadas, sin descansos entre citas.
- Al **desactivar o eliminar** un servicio, proveedor o recurso: se cancelan sus
  reservas futuras. Se permite la eliminación definitiva.

### Políticas
- **Plazos mínimos** de reserva, reprogramación y cancelación: los configura el
  **proveedor, por servicio**, eligiendo la unidad de un menú (minutos / horas / días).
  Un cambio de plazos **no afecta** reservas ya creadas.
- **Límites globales** (iguales para todos los clientes): **20** reservas activas por
  cliente · **5** reprogramaciones por reserva.

### Reservas del cliente
- La reserva se **confirma de inmediato** al elegir una franja libre. No hay
  aprobación previa.
- Consultar la disponibilidad **requiere estar autenticado**. Agenda abierta hasta
  **12 meses** hacia el futuro. Filtros: **fecha, proveedor, modalidad**.
- El cliente solo reserva **para sí mismo**. No se piden datos adicionales al reservar.
- **Estados de una reserva:** agendada · atendida · cancelada · no-show.

### Operación (proveedor / propietario)
- **Solo el propietario** puede editar o mover reservas, y **solo él** crea reservas
  a nombre de un cliente (registrando antes al cliente con todos sus datos).
- El **proveedor** solo ve, de cada cita, el **nombre y el teléfono** del cliente
  (más servicio, fecha, hora, estado). Sin notas internas.
- El proveedor puede **marcar asistencia / no-show** únicamente **después de la hora
  de la cita**; es la única acción que puede aplicar sobre una reserva.
- El **horario del proveedor** es recurrente semanal. En el MVP no se modifica tras
  crearlo, ni hay bloqueos ni ausencias.

### Gestión del backlog en Azure DevOps
- Story Points, Business Value y Definition of Ready: el equipo los define
  **manualmente**. Nada de esto está automatizado.

---

## 3. Fuera del alcance del MVP

En Azure DevOps quedan en estado **Removed** (recuperables; no aparecen en el board
por defecto). No se pueden borrar del todo desde la integración usada.

| Removido | Motivo |
|---|---|
| **E-06 · Notificaciones y Recordatorios** (3 features, 3 HU) | No está en las 5 épicas del MVP |
| **E-07 · Reportes y Analítica de Ocupación** (3 features, 3 HU) | Ídem |
| Feature de confirmación/rechazo de reservas + sus 2 HU | La reserva es inmediata |
| Feature de bloqueos y ausencias + sus 2 HU | Fuera del MVP |
| HU "Actualizar mi horario de atención" | Fuera del MVP |
| Feature de cuentas y roles internos + sus 3 HU | No hay roles internos que gestionar |
| **FE-015** (Gestión de negocios por un admin) + **HU-0027, HU-0028, HU-0029** | Pivote 2026-09-11: registro autoservicio, sin rol Administrador de la Plataforma |

También quedan fuera del MVP (sin work item): consecuencias de cancelar fuera de
plazo, automatización ante no-show, y la gestión de sedes múltiples dentro de un
mismo negocio.

---

## 4. Preguntas abiertas que quedan

| HU | Pregunta |
|---|---|
| HU-0016 · Consultar franjas disponibles | ¿Qué datos de cada franja se muestran (proveedor, modalidad, duración, precio)? |
| HU-0019 · Consultar mis reservas | ¿Se muestran también las reservas pasadas? ¿Por cuánto tiempo? |
| HU-0009 · Desactivar un proveedor | Si era el único proveedor de un servicio: ¿reservas pendientes de reasignación, canceladas, o se bloquea la desactivación? |
| HU-0022 · Consultar mi agenda del día | ¿Qué vistas se necesitan (día, semana, lista, calendario)? ¿El propietario ve la agenda de todos los proveedores? |
| HU-0002 · Iniciar sesión en mi cuenta | ¿A qué pantalla se redirige a un cliente y a un proveedor al iniciar sesión? (el destino del propietario ya está definido) |
| HU-0042 · Crear mi negocio | ¿Cómo selecciona/cambia el propietario entre sus negocios? ¿A cuál se redirige por defecto al iniciar sesión con varios? ¿El aislamiento de datos entre los negocios de un mismo propietario es total? |

---

## 5. Mapeo código ↔ ID de work item

El código del título **no coincide** con el ID interno de Azure DevOps.

### Épicas
| Código | ID |
|---|---|
| E-01 | 1 |
| E-02 | 2 |
| E-03 | 3 |
| E-04 | 4 |
| E-05 | 5 |

### Features
| Código | ID | Código | ID |
|---|---|---|---|
| FE-001 | 8  | FE-009 | 18 |
| FE-002 | 10 | FE-010 | 19 |
| FE-003 | 11 | FE-011 | 20 |
| FE-004 | 12 | FE-012 | 21 |
| FE-005 | 13 | FE-013 | 23 |
| FE-006 | 14 | FE-014 | 24 |
| FE-007 | 16 | FE-016 | 90 |
| FE-008 | 17 | | |

### Historias de usuario
| Código | ID | Código | ID | Código | ID |
|---|---|---|---|---|---|
| HU-0001 | 42 | HU-0011 | 55 | HU-0021 | 36 |
| HU-0002 | 43 | HU-0012 | 56 | HU-0022 | 58 |
| HU-0003 | 44 | HU-0013 | 57 | HU-0023 | 59 |
| HU-0004 | 48 | HU-0014 | 37 | HU-0024 | 62 |
| HU-0005 | 49 | HU-0015 | 41 | HU-0025 | 63 |
| HU-0006 | 50 | HU-0016 | 31 | HU-0026 | 64 |
| HU-0007 | 51 | HU-0017 | 32 | HU-0041 | 91 |
| HU-0008 | 52 | HU-0018 | 33 | HU-0042 | 92 |
| HU-0009 | 53 | HU-0019 | 34 | | |
| HU-0010 | 54 | HU-0020 | 35 | | |

Los IDs 6, 7, 9, 15, 22, 25–30, 38–40, 45–47, 60–61, 65–70, 71–73, 89 son ítems en
estado Removed (conservan su código antiguo, que **no se reutiliza**). Los últimos
cuatro (FE-015=71, HU-0027=72, HU-0028=73, HU-0029=89) son del flujo de registro
por Administrador de la Plataforma, retirado el 2026-09-11.

---

## 6. Historial de cambios

1. **2026-09-02** — Backlog inicial: 7 épicas / 23 features / 40 HU.
2. **2026-09-04** — Aplicadas las 62 respuestas del equipo. Recorte a 5 épicas;
   removidos los ítems de la sección 3. Renumerados los códigos activos sin huecos.
3. **2026-09-04** — Correcciones: un servicio admite varios proveedores;
   asistencia/no-show = booleano que marca el proveedor tras la hora, sin otros permisos.
4. **2026-09-04** — Faltaba el acceso del administrador: HU-0002 y HU-0003
   generalizadas a cualquier cuenta; FE-001 renombrada a "Registro y autenticación".
5. **2026-09-04** — Confirmado multi-negocio: añadidas FE-015, HU-0027 y HU-0028;
   E-01 y E-02 actualizadas.
6. **2026-09-10** — Rename de terminología (pedido del equipo): el "administrador
   del negocio" pasa a llamarse **"propietario"** en todas las épicas, features e
   HU. Se conserva "administrador de la plataforma" para el super admin. FE-015
   renombrada. HU-0002 y HU-0004 ya venían alineadas por el equipo.
7. **2026-09-11** — HU-0027 dividida en dos (pedido del equipo). Ya no cubre a la
   vez negocio y propietario: **HU-0027** queda solo con los datos del negocio
   (nombre, dirección, ID fiscal, moneda base) y **HU-0029** (nueva) se lleva la
   cuenta del propietario (nombre, correo, celular, contraseña). El flujo real
   **no cambia**: sigue siendo un único formulario/transacción que crea ambos
   juntos; la división es solo de las HU y sus criterios de aceptación, para
   facilitar el tracking. FE-015 actualizada con las 3 HU.
8. **2026-09-11 (mismo día) — Pivote a registro self-service.** El equipo definió
   3 principios de arquitectura: el registro es **100% autoservicio, sin
   aprobación** (el negocio queda activo de inmediato); **"Usuario Propietario"**
   y **"Negocio"** son entidades completamente separadas a nivel de datos; y al
   registrarse, el usuario queda autenticado y es **redirigido de inmediato** a
   crear su negocio. Esto **reemplaza por completo** el flujo anterior y **elimina
   del MVP el rol "Administrador de la Plataforma"** (ni siquiera queda para
   desactivar negocios). Se retiraron (Removed) FE-015, HU-0027, HU-0028, HU-0029
   y sus tareas técnicas (incluida la de sembrar la cuenta del super admin, ya sin
   sentido). Se crearon **FE-016 · Registro y activación de negocios** (bajo E-01)
   con **HU-0042 · Crear mi negocio**, y **HU-0041 · Registrarme como propietario**
   (bajo FE-001, junto a HU-0001/02/03) — ambas reutilizan las validaciones de
   campo que ya existían, sin inventar nada nuevo. Se reescribió **HU-0002** (login)
   para quitar el escenario de Administrador de la Plataforma. Pendiente del
   equipo: replanear Sprint 1 (perdió HU-0027 con sus 5 SP y 5 tasks) y crear
   tasks nuevas para HU-0041/HU-0042.
9. **2026-09-11 (mismo día)** — Respondidas las 2 preguntas abiertas de HU-0042:
   el propietario **queda bloqueado** del resto de la plataforma hasta crear su
   primer negocio, y **sí puede crear más de un negocio** con la misma cuenta (no
   es un propietario = un negocio). Se agregaron 2 escenarios a HU-0042 (bloqueo
   sin negocio, creación de un negocio adicional) y sus preguntas abiertas se
   reemplazaron por 3 nuevas que surgen del multi-negocio por propietario: cómo
   selecciona/cambia de negocio, a cuál se redirige por defecto al iniciar sesión,
   y si el aislamiento de datos entre los negocios de un mismo propietario es total.

> Documento generado para acompañar el backlog en Azure DevOps. Mantener sincronizado
> a mano si se editan work items directamente en Azure.
