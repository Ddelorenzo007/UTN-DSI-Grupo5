# Actividad 1 — Modelo de Dominio
## Sistema de Gestión de Estacionamiento UTN FRLP (Grupo 07 — Proyecto Final)

Este documento acompaña al diagrama de clases del modelo de dominio (`Entrega1.3`) y deja
registradas las correcciones y justificaciones surgidas de las dos rondas de devolución de la
cátedra, incorporando también lo aclarado por Grupo 07 en las entrevistas.

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

### 1.4 Se eliminaron los atributos `id` de todo el diagrama

**Observación de la cátedra:** un identificador subrogado es un patrón de mapeo
objeto-relacional, no un concepto del negocio.

**Corrección:** se sacaron todos los `idX` (idUsuario, idTransaccion, idAcceso, etc.) del modelo de
dominio. Se mantienen `patente`, `dni` y `email` porque son atributos reales del negocio, no claves
subrogadas — pertenecen al dominio, a diferencia de un autoincremental de base de datos.

### 1.5 `Descuento` se renombró a `Promoción`

**Observación de la cátedra:** lo que se modelaba como `Descuento` es en realidad una promoción —
el descuento sería el efecto puntual sobre una transacción, propio de ella, no la regla en sí.

**Corrección:** la clase pasa a llamarse `Promoción` (la regla configurable: porcentaje, montoMínimo,
vigencia, estado). El efecto puntual del descuento queda representado en `Transacción` mediante el
atributo `montoExtra`.

### 1.6 `Restricción` y `Anulación` como objetos propios, no atributos sueltos

**Observación de la cátedra:** tanto en `Vehículo` como en `Transacción` había datos que podían no
existir y que no dependían propiamente de la entidad (`motivoAnulación`, `motivoRestricción`,
`nivelRestricción`); además, si se anula una transacción debería registrarse quién lo hizo y por qué.

**Corrección:** se crearon las clases `Restricción` (`nivel`, `motivo`, `fecha`) y `Anulación`
(`motivo`, `fecha`), cada una asociada a `Usuario` (`Usuario "1" --> "0..*" Restriccion/Anulacion`,
obligatorio: ninguna de las dos puede existir sin quien la generó). `Vehículo` se asocia a
`Restricción` en `0..1 — 0..1`, y `Transacción` a `Anulación` en `1 — 0..1`.

### 1.7 `cuposDisponibles` y `montoFinal` pasan a ser operaciones, no atributos

**Observación de la cátedra:** si `cuposDisponibles` únicamente se calcula a modo de consulta,
entonces es un comportamiento, no un atributo derivado.

**Corrección:**
```
Estacionamiento:: cuposDisponibles() {
  return self.capacidadMáxima - (self.Acceso() -> filter(a | a.estaDentro()) -> size())
}
Acceso:: estaDentro(): boolean {
  return self.estado == "Dentro"
}
```
Mismo criterio se aplicó a `Transacción.montoFinal()`, que se calcula como `monto + montoExtra` sin
necesidad de persistirse aparte, ya que ambos valores quedan congelados en el momento de la
transacción.

### 1.8 Corrección de multiplicidades

- **`Usuario`–`Anulación`** y **`Transacción`–`Anulación`**: el lado de `Usuario` y de `Transacción`
  debe ser `1` obligatorio, no `0..1` — una `Anulación` no puede existir sin su transacción ni sin el usuario que la hizo.
- **`Usuario`–`Notificación`**: el `0..1` (emisor) y el `1` (destinatario) van pegados a `Usuario`,
  no a `Notificación` — cada notificación tiene como máximo un emisor y exactamente un destinatario,
  no al revés.

---

## 2. Decisiones justificadas (se mantienen, con su razón)

### 2.1 `RegistroAsistencia`

Registra cuándo entra y sale a trabajar cada usuario con rol de personal, y los días trabajados. No
surge de ningún RF ni HU puntual del documento de alcance — se agregó pensando en sustentar métricas
de turnos/carga operativa para los reportes (RF-024/025). Se asocia directamente a `Usuario`
(`Usuario "1" --> "0..*" RegistroAsistencia`), ya que no existe una subclase `Empleado` a la cual
colgarla.

### 2.2 `montoExtra` se mantiene como atributo, no como operación

A diferencia de `montoFinal`, `montoExtra` depende de `Promoción.porcentaje`, que puede seguir
cambiando en el tiempo (un administrador puede editar o desactivar una promoción). Si se calculara
al vuelo en vez de guardarse, las transacciones históricas cambiarían de valor cada vez que cambie
la promoción — por eso se "fotografía" el monto bonificado en el momento de aplicarse.

### 2.3 Niveles de restricción (`Leve`/`Crítica`) en `Restricción`

**Justificación (HU-032.3):** *"Bloqueo del botón 'Confirmar Ingreso' si la restricción es de
carácter crítico"*. Implica que existen niveles de restricción (no todas bloquean el ingreso), por
eso se usa una enumeración en vez de un booleano simple.

### 2.4 `RegistroInvitado` como entidad separada de `Vehículo`

**Justificación (HU-018.1):** el sistema debe solicitar obligatoriamente "Nombre, Apellido, Patente
y Detalle/Motivo" del invitado — el nombre y apellido de la persona son datos de identidad distintos
del vehículo en sí. Además, `Vehiculo "1" --> "0..*" RegistroInvitado`, porque un mismo vehículo
puede ser usado por distintas personas en distintas visitas; para saber cuál invitado corresponde a
cada acceso puntual, se agregó `Acceso "0..1" --> "0..1" RegistroInvitado`.

### 2.5 `marca`, `modelo` y `color` opcionales en `Vehículo`

Un vehículo invitado se registra solo con la patente (HU-018.1); el resto de los datos se completa
únicamente si, más adelante, el vehículo se asocia formalmente a un conductor (HU-005).

### 2.6 Dos asociaciones `Usuario`–`Acceso` (ingreso y egreso)

**Justificación (HU-033.1):** el detalle de un acceso debe mostrar por separado qué operador
registró el ingreso y cuál el egreso — pueden ser personas distintas por cambio de turno. El modelo
no registra, en cambio, quién *conducía* el vehículo: eso se obtiene indirectamente vía
`Vehículo → Usuario` (dueño) o `Vehículo → RegistroInvitado`, igual que una barrera real que
identifica por patente, no por ocupante — ningún RF pide lo segundo.

### 2.7 `Promoción.activa` y `Estacionamiento.cerradoManual` como booleanos independientes

**Justificación (HU-014.2 y HU-013.4):** en ambos casos la historia de usuario plantea dos
condiciones independientes con un "Y"/"O" explícito: una promoción es válida si está dentro de su
vigencia por fecha **y** además tiene el switch en `Activa` (permite pausarla manualmente sin perder
la fecha de vencimiento configurada); un estacionamiento se considera cerrado si está fuera de
horario **o** fue marcado como cerrado manualmente (para excepciones puntuales: obras, cortes de
luz, eventos). Ninguno de los dos booleanos duplica a la otra condición.

### 2.8 `cuposDisponibles` no distingue invitados de conductores registrados

El cálculo cuenta todos los `Acceso` con estado `Dentro` de un `Estacionamiento`, sin pasar por
`RegistroInvitado`: un lugar ocupado por un invitado ocupa el mismo espacio físico que uno ocupado
por un conductor registrado, así que no hace falta esa distinción para el cupo.

### 2.9 ¿Una cuenta puede estar inactiva? ¿Cómo reflejan esa situación?

No se modela un estado independiente en Cuenta. Consultado con Grupo 07, confirmaron que el saldo
y el estado pertenecen al Usuario, no a la cuenta en sí: si el usuario no está en estado Activo
(Bloqueado o Deshabilitado), no puede iniciar sesión y en garita se le bloquea el ingreso, por lo
que tampoco se le debita. Solo un administrador puede rehabilitarlo. Por eso Cuenta solo tiene
saldoActual, y el control de inactividad se resuelve consultando Usuario.estado antes de operar
sobre la cuenta asociada.

---

## 3. Fuentes utilizadas

- Plan de Gestión del Alcance — Estacionamiento UTN (Grupo 07)
- Matriz de Trazabilidad de Requerimientos (criterios de aceptación)
- Documento de Historias de Usuario (HU-001 a HU-038)
- Primera entrevista con Grupo 07, por mail (invitados, saldo, accesos y cupos)
- Segunda entrevista con Grupo 07, por mail (estado de cuenta, autoría de restricciones,
  terminología Promoción/Descuento)
- Primera corrección de la cátedra (Demian) sobre `Entrega1.1`
- Segunda corrección de la cátedra (Demian) sobre `Entrega1.3`