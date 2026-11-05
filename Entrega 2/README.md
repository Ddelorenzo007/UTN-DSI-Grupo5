# Actividad 2 - Modelado de Sistema
## Sistema de Gestión de Estacionamiento UTN FRLP (Grupo 07 — Proyecto Final)

Este documento acompaña al diagrama de clases del modelo de sistemas, desarrollando los 2 requerimientos funadmenaltes del proyecto: Alta de un Acceso y Alta de un Registro de invitado.

---

## 1. Correcciones aplicadas de la entrega 1

### 1.1 Los vehículos se diferencian por tipo

**Observación de la cátedra:** los vehículos pueden ser de 2 tipos (auto o moto) y los espacios del estacionamiento se diferencian por tipo.

**Corrección:** los vehículos ahora tiene tipo auto o moto y los cupos del estacionamiento son cargados y calculados en base al tipo de vehículo.

### 1.2 Usuario sin vehículo

**Observación de la cátedra:** un usuario podría no tener vehículo.

**Corrección:** se agregó la multiplicidad 0..1 del lado de vehículo.

### 1.3 Clase Tarifa

Se agregó la clase Tarifa que hace referencia al monto que debe cobrarse al momento de dar de alta un acceso.

---

## 3. Fuentes utilizadas

- Plan de Gestión del Alcance — Estacionamiento UTN (Grupo 07)
- Matriz de Trazabilidad de Requerimientos (criterios de aceptación)
- Documento de Historias de Usuario (HU-001 a HU-038)
- Tercer corrección de la cátedra (Demian) sobre `Entrega1.3`