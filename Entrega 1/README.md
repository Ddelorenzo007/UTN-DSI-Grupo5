# Actividad 1 — Modelo de Dominio y de Sistema
## Sistema de Gestión de Estacionamiento UTN FRLP (Grupo 07 — Proyecto Final)

Este documento acompaña al diagrama de clases del modelo de dominio y deja registradas las
correcciones y justificaciones surgidas de la devolución de la cátedra sobre la primera versión
entregada (`Entrega1_1.jpg`).

---

## 1. Correcciones aplicadas

### 1.1 Se eliminaron las subclases `Administrador`, `Empleado` y `Autoridad` de `Usuario`

**Observación de la cátedra:** un modelo de dominio no debería tener clases sin atributos propios;
además, cuando un requerimiento dice "el administrador puede hacer X", se refiere a un **actor** de
caso de uso, no necesariamente a una clase de dominio.

**Corrección:** las cuatro variantes de usuario dejan de ser subclases y pasan a representarse con
un atributo `rol: char` en `Usuario`. Ninguna de esas subclases tenía atributos propios más allá
de los heredados, y `Autoridad` en particular no participaba de ninguna asociación — era un actor
puro, sin relevancia estructural en el dominio.

### 1.2 Se separó la cuenta de estacionamiento (`Cuenta`) de la identidad del usuario

**Observación de la cátedra:** ¿el saldo no sería de una cuenta? ¿Los administradores y empleados no
podrían ser también conductores con vehículos propios?

**Corrección:** se creó la clase `Cuenta` (con `saldoActual`), asociada a `Usuario` con
multiplicidad `1 — 0..1`. `Vehiculo` y `Transaccion` ahora se asocian a `Cuenta`, no a un rol
específico. Esto permite que cualquier usuario, sin importar su rol, tenga una cuenta de
estacionamiento — resolviendo el problema de que antes un Administrador no podía, a la vez, ser
Conductor.

### 1.3 Se eliminó el atributo `tipo` (Ingreso/Egreso) de `Acceso`

**Observación de la cátedra:** ¿tiene sentido un tipo de acceso si `Acceso` ya tiene
`fechaHoraIngreso` y `fechaHoraEgreso`?

**Corrección:** `Acceso` representa una estadía completa (se crea al ingresar, se completa al
egresar), no un evento puntual de ingreso o egreso por separado. El atributo era redundante con las
dos fechas y se eliminó, junto con la enumeración `TipoAcceso`.

---

## 2. Decisiones justificadas (se mantienen sin cambios)

### 2.1 `RegistroAsistencia`

Registra cuándo entra y sale a trabajar cada usuario con rol de personal, y los días trabajados.
**Aclaración importante:** esta entidad no surge de ningún RF ni HU puntual del documento de
alcance — fue agregada por el equipo pensando en sustentar métricas de turnos/carga operativa para
los reportes (RF-024/025). Se consultó a la cátedra si corresponde mantenerla; de mantenerse, se
asocia directamente a `Usuario` (`Usuario "1" --> "0..*" RegistroAsistencia`), ya que no existe una
subclase `Empleado` a la cual colgarla.

### 2.2 `montoExtra` en `Transaccion`

La vigencia de un descuento (`vigenciaDesde`/`vigenciaHasta` en `Descuento`) define el rango de
fechas en que puede aplicarse, no una propiedad de la transacción en sí. Para que el historial de
transacciones no dependa del valor *actual* de un `Descuento` (que podría cambiar más adelante), se
agregó `montoExtra: float` y `montoFinal: float` a `Transaccion`, que congela el beneficio efectivamente
otorgado en el momento.

### 2.3 `nivelRestriccion` en `Vehiculo`

**Justificación (HU-032.3):** *"Bloqueo del botón 'Confirmar Ingreso' si la restricción es de
carácter crítico"*. Esto implica que existen niveles de restricción (no todas bloquean el ingreso),
por lo que se usa la enumeración `NivelRestriccion {Ninguna, Leve, Critica}` en vez de un booleano
simple.

**No se modela un historial de restricciones como clase aparte:** se revisó todo el documento de
Historias de Usuario y el RF-032 original, y no hay ningún requerimiento que pida consultar
restricciones pasadas, quién las aplicó o cuándo se levantaron. HU-032 solo describe una evaluación
sobre el estado *actual* del vehículo. Agregar una clase `Restriccion` con historial sería
estructura no solicitada por el negocio — mismo tipo de error señalado por la cátedra con
`Autoridad`.

### 2.4 `RegistroInvitado` como entidad separada de `Vehiculo`

**Justificación (HU-018.1):** el sistema debe solicitar obligatoriamente "Nombre, Apellido, Patente
y Detalle/Motivo" del invitado — la patente ya vive en `Vehiculo`, pero el nombre y apellido de la
persona son datos de identidad distintos del vehículo en sí, por lo que se modelan en una entidad
separada asociada `0..1 — 0..1`.

### 2.5 `cuposDisponibles` en `Estacionamiento` como atributo calculado

No se persiste como contador que se actualiza manualmente (generaría inconsistencias ante fallos o
accesos simultáneos). Se calcula como:

```
cuposDisponibles = capacidadMaxima − cantidad de Acceso con estado "Dentro" para ese Estacionamiento
```

## 3. Fuentes utilizadas

- Plan de Gestión del Alcance — Estacionamiento UTN (Grupo 07)
- Matriz de Trazabilidad de Requerimientos (criterios de aceptación)
- Documento de Historias de Usuario (HU-001 a HU-038)
- Entrevista con Grupo 07 (respuestas por mail sobre invitados, saldo, accesos y cupos)
- Devolución de la cátedra (Demian) sobre `Entrega1_1.jpg`