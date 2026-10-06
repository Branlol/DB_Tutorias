# Informe de Normalización – TutorHUB

## Integrantes
- Alex Duvan Lopez Solano – 2242040
- Dayron Stiven Galeano Mejía – 2250153
- Brandon Andrés Jaimes Romero – 2250156
- Karoll Nataly Sánchez Ortega – 2250169

---

## 1. Relación Universal (Tabla sin normalizar)

Partiendo del modelo E-R, la relación universal que contiene toda la información del sistema sería:

```
TutorHUB_Universal(
    persona_codigo, persona_correo, persona_nombre, persona_apellido, persona_telefono,
    estudiante_programa_academico, estudiante_semestre,
    tutor_escuela, tutor_estado,
    tutoria_codigo, tutoria_tipo, tutoria_descripcion, tutoria_max_estudiantes,
    tutoria_estado, tutoria_observaciones,
    sesion_fecha, sesion_hora_inicio, sesion_hora_finalizacion, sesion_modalidad,
    asignatura_codigo, asignatura_nombre,
    lugar_edificio, lugar_aula, lugar_capacidad
)
```

**Problemas identificados:**
- Atributos no atómicos:
  - Atributo compuesto "Sesión" anidado dentro de Tutoría.
  - Atributo "Nombre" de Persona que agrupa nombres y apellidos.
- Relación N:M Estudiante-Tutoría genera redundancia masiva.
- Datos de Persona se repiten para cada tutoría.
- Dependencias parciales y transitivas múltiples.

---

## 2. Primera Forma Normal (1FN)

### Regla
Una relación está en 1FN si:
- Todos los atributos son atómicos (no existen grupos repetitivos, campos compuestos ni atributos multivaluados).
- Existe una clave primaria definida.

### Aplicación

**Problemas de atomicidad identificados:**
1. El atributo compuesto **Sesión** en la entidad Tutoría agrupa `{Fecha, Hora_inicio, Hora_finalizacion, Modalidad}`.
2. El atributo **Nombre** en Persona es compuesto (contiene nombres y apellidos).

**Solución aplicada:**
- Se descompone **Nombre** en dos campos atómicos: `Nombre` y `Apellido` en la tabla `PERSONA`.
- Se descompone el atributo compuesto **Sesión** en atributos individuales directamente en la tabla `TUTORIA`.
- Se resuelve la relación N:M mediante la tabla intermedia `ESTUDIANTE_TUTORIA`.
- Se separa el historial de estados en la tabla `TUTORIA_ESTADO`.

```
PERSONA(Código, Correo, Nombre, Apellido, Teléfono)
    PK: Código

ESTUDIANTE(Código_estudiante, Programa_académico, Semestre)
    PK: Código_estudiante
    FK: Código_estudiante → PERSONA(Código)

TUTOR(Código_tutor, Escuela, Estado)
    PK: Código_tutor
    FK: Código_tutor → PERSONA(Código)

ASIGNATURA(Código_asignatura, Nombre)
    PK: Código_asignatura

LUGAR(Edificio, Aula, Capacidad)
    PK: (Edificio, Aula)

TUTORIA(Código_tutoria, Nombre_del_tipo, Descripción, Cantidad_máxima_de_estudiantes,
        Fecha, Hora_de_inicio, Hora_de_finalización, Modalidad,
        Observaciones, Código_asignatura, Edificio, Aula, Código_tutor)
    PK: Código_tutoria
    FK: Código_tutor → TUTOR(Código_tutor)
    FK: Código_asignatura → ASIGNATURA(Código_asignatura)
    FK: (Edificio, Aula) → LUGAR(Edificio, Aula)

ESTUDIANTE_TUTORIA(Código_estudiante, Código_tutoria)
    PK: (Código_estudiante, Código_tutoria)
    FK: Código_estudiante → ESTUDIANTE(Código_estudiante)
    FK: Código_tutoria → TUTORIA(Código_tutoria)

TUTORIA_ESTADO(Código_tutoria, Fecha_desde, Estado)
    PK: (Código_tutoria, Fecha_desde)
    FK: Código_tutoria → TUTORIA(Código_tutoria)
```

**Resultado:** Todos los atributos son atómicos. Los grupos repetitivos de la relación N:M Estudiante-Tutoría se resolvieron con tabla intermedia. El historial de estados se separó en su propia tabla. Estado: Cumple 1FN.

---

## 3. Segunda Forma Normal (2FN)

### Regla
Una relación está en 2FN si:
- Está en 1FN.
- No existen dependencias parciales (todo atributo no primo depende de la totalidad de la clave primaria, no de una parte de ella).

### Análisis por tabla

| Tabla | Clave Primaria | ¿Dependencia parcial? | Estado |
|-------|----------------|----------------------|--------|
| PERSONA | Código | PK simple: no aplica | Cumple 2FN |
| ESTUDIANTE | Código_estudiante | PK simple: no aplica | Cumple 2FN |
| TUTOR | Código_tutor | PK simple: no aplica | Cumple 2FN |
| ASIGNATURA | Código_asignatura | PK simple: no aplica | Cumple 2FN |
| TUTORIA | Código_tutoria | PK simple: no aplica | Cumple 2FN |
| LUGAR | (Edificio, Aula) | Capacidad depende de (Edificio, Aula) en conjunto, no de una sola parte. Un edificio tiene múltiples aulas y un número de aula puede repetirse en diferentes edificios: no hay dependencia parcial | Cumple 2FN |
| ESTUDIANTE_TUTORIA | (Código_estudiante, Código_tutoria) | No contiene atributos no primos: no aplica | Cumple 2FN |
| TUTORIA_ESTADO | (Código_tutoria, Fecha_desde) | Estado depende de la clave compuesta completa (una tutoría puede cambiar de estado en distintas fechas): no hay dependencia parcial | Cumple 2FN |

**Resultado:** No se encontraron dependencias parciales en ninguna tabla. Estado: Cumple 2FN.

---

## 4. Tercera Forma Normal (3FN)

### Regla
Una relación está en 3FN si:
- Está en 2FN.
- No existen dependencias transitivas (ningún atributo no primo depende funcionalmente de otro atributo no primo).

### Análisis por tabla

**PERSONA(Código, Correo, Nombre, Apellido, Teléfono)**
- Código → Correo, Nombre, Apellido, Teléfono
- `Correo` y `Teléfono` son claves candidatas (valores únicos por persona), por lo que determinan atributos pero son superclaves y no generan dependencia transitiva entre atributos no primos.
- Estado: Cumple 3FN.

**ESTUDIANTE(Código_estudiante, Programa_académico, Semestre)**
- Código_estudiante → Programa_académico, Semestre
- ¿Programa_académico → Semestre? No. Un programa tiene múltiples semestres posibles; el semestre depende directamente del estudiante, no del programa.
- Estado: Cumple 3FN.

**TUTOR(Código_tutor, Escuela, Estado)**
- Código_tutor → Escuela, Estado
- ¿Escuela → Estado? No. La escuela no determina el estado de actividad del tutor.
- Estado: Cumple 3FN.

**ASIGNATURA(Código_asignatura, Nombre)**
- Relación de dos atributos. No existe posibilidad de dependencia transitiva.
- Estado: Cumple 3FN.

**LUGAR(Edificio, Aula, Capacidad)**
- (Edificio, Aula) → Capacidad
- Posee un único atributo no primo (Capacidad). No puede existir dependencia transitiva.
- Estado: Cumple 3FN.

**TUTORIA(Código_tutoria, Nombre_del_tipo, Descripción, Cantidad_máxima_de_estudiantes, Fecha, Hora_de_inicio, Hora_de_finalización, Modalidad, Observaciones, Código_asignatura, Edificio, Aula, Código_tutor)**
- Código_tutoria → todos los demás atributos de la relación.
- ¿Nombre_del_tipo → Cantidad_máxima_de_estudiantes? No. La capacidad máxima es un valor específico definido para cada tutoría programada, no un valor fijo determinado por el tipo.
- Los atributos de sesión (Fecha, Hora_de_inicio, Hora_de_finalización, Modalidad) dependen directamente de Código_tutoria (cada tutoría representa una única sesión programada).
- (Edificio, Aula) corresponden a claves foráneas hacia LUGAR y no determinan transitivamente otros atributos no primos dentro de TUTORIA.
- Estado: Cumple 3FN.

**TUTORIA_ESTADO(Código_tutoria, Fecha_desde, Estado)**
- (Código_tutoria, Fecha_desde) → Estado
- Posee un único atributo no primo. No existe dependencia transitiva.
- Estado: Cumple 3FN.

**ESTUDIANTE_TUTORIA:** No contiene atributos no primos. Estado: Cumple 3FN.

**Resultado:** No se detectaron dependencias transitivas en el esquema. Estado: Cumple 3FN.

---

## 5. Forma Normal de Boyce-Codd (FNBC / BCNF)

### Regla
Una relación está en BCNF si:
- Está en 3FN.
- Para toda dependencia funcional X → Y, X es una superclave.

### Análisis

**PERSONA:**
- Dependencias funcionales: Código → Correo, Nombre, Apellido, Teléfono; Correo → Código, Nombre, Apellido, Teléfono; Teléfono → Código, Correo, Nombre, Apellido.
- `Código` es la clave primaria; `Correo` y `Teléfono` son claves candidatas únicas. Todos los determinantes son superclaves.
- Estado: Cumple BCNF.

**ESTUDIANTE:**
- Dependencia funcional: Código_estudiante → Programa_académico, Semestre.
- `Código_estudiante` es superclave.
- Estado: Cumple BCNF.

**TUTOR:**
- Dependencia funcional: Código_tutor → Escuela, Estado.
- `Código_tutor` es superclave.
- Estado: Cumple BCNF.

**ASIGNATURA:**
- Dependencia funcional: Código_asignatura → Nombre.
- `Código_asignatura` es superclave.
- Estado: Cumple BCNF.

**LUGAR:**
- Dependencia funcional: (Edificio, Aula) → Capacidad.
- `(Edificio, Aula)` es la clave primaria y superclave.
- Estado: Cumple BCNF.

**TUTORIA:**
- Dependencias funcionales: 
  - `Código_tutoria` → Nombre_del_tipo, Descripción, ..., Código_tutor, Código_asignatura, Edificio, Aula.
  - `(Fecha, Hora_de_inicio, Código_tutor)` → Código_tutoria, Nombre_del_tipo, Descripción, ... (Clave candidata alternativa: un tutor no puede tener dos tutorías en la misma fecha y hora).
- Tanto `Código_tutoria` como `(Fecha, Hora_de_inicio, Código_tutor)` son superclaves de la relación.
- Estado: Cumple BCNF.

**TUTORIA_ESTADO:**
- Dependencia funcional: (Código_tutoria, Fecha_desde) → Estado.
- `(Código_tutoria, Fecha_desde)` es la clave primaria y superclave.
- Estado: Cumple BCNF.

**ESTUDIANTE_TUTORIA:** 
- (Código_estudiante, Código_tutoria) → conjunto vacío (sin atributos no primos).
- Estado: Cumple BCNF.

**Resultado:** Todas las tablas cumplen BCNF.

---

## 6. Cuarta Forma Normal (4FN)

### Regla
Una relación está en 4FN si:
- Está en BCNF.
- No existen dependencias multivaluadas no triviales (para toda dependencia multivaluada X →→ Y, X es una superclave).

### Análisis

Las dependencias multivaluadas aparecen cuando dos o más atributos independientes tienen múltiples valores asociados a una misma clave dentro de una misma tabla.

| Tabla | ¿Dependencia multivaluada? | Justificación |
|-------|---------------------------|---------------|
| PERSONA | No | Cada persona tiene un único correo, nombre, apellido y teléfono registrado |
| ESTUDIANTE | No | Cada estudiante pertenece a un programa y cursa un semestre |
| TUTOR | No | Cada tutor pertenece a una escuela y posee un estado |
| ASIGNATURA | No | Relación funcional directa entre código y nombre |
| LUGAR | No | Cada espacio físico (edificio, aula) tiene una sola capacidad definida |
| TUTORIA | No | Cada tutoría tiene asignada una sesión, un tutor, una asignatura y un lugar |
| ESTUDIANTE_TUTORIA | No | Tabla de enlace directo binario entre estudiante y tutoría |
| TUTORIA_ESTADO | No | Cada registro temporal asocia un único estado a la tutoría en esa fecha |

**Resultado:** No existen dependencias multivaluadas no triviales. Estado: Cumple 4FN.

---

## 7. Quinta Forma Normal (5FN)

### Regla
Una relación está en 5FN (Forma Normal de Proyección-Unión) si:
- Está en 4FN.
- No puede descomponerse en relaciones más pequeñas sin pérdida de información (toda dependencia de unión está implicada por las claves candidatas).

### Análisis

La 5FN aplica cuando existen dependencias de unión (join dependencies) no implicadas por claves candidatas, situación habitual en relaciones n-arias (ternarias o superiores) con dependencias cíclicas.

**Revisión de tablas:**
- **ESTUDIANTE_TUTORIA(Código_estudiante, Código_tutoria):** Es una relación binaria no descomponible.
- **TUTORIA_ESTADO(Código_tutoria, Fecha_desde, Estado):** Relación temporal binaria respecto a la entidad tutoría, sin ciclos de unión.
- Las demás relaciones corresponden a entidades y tablas normalizadas cuyas dependencias de unión son triviales o están dadas por sus claves primarias.

**Resultado:** Estado: Cumple 5FN.

---

## 8. Sexta Forma Normal (6FN)

### Regla
Una relación está en 6FN si:
- Está en 5FN.
- La relación no admite dependencias de unión no triviales, lo que en la práctica exige que cada tabla contenga como máximo un atributo no clave junto a la clave primaria.

### Análisis

La 6FN tiene aplicación primordialmente teórica o en motores especializados de bases de datos temporales (donde cada atributo varía de forma independiente en el tiempo). 

Llevar el modelo TutorHUB a 6FN implicaría descomponer cada tabla en relaciones atómicas binarias:

```
PERSONA_CORREO(Código, Correo)
PERSONA_NOMBRE(Código, Nombre)
PERSONA_APELLIDO(Código, Apellido)
PERSONA_TELEFONO(Código, Teléfono)
TUTORIA_TIPO(Código_tutoria, Nombre_del_tipo)
TUTORIA_DESCRIPCION(Código_tutoria, Descripción)
... (para cada uno de los atributos del sistema)
```

**Decisión técnica:** No se aplica 6FN debido a que:
1. Incrementaría de forma innecesaria la cantidad de tablas de 8 a más de 30.
2. Afectaría negativamente el rendimiento de las consultas debido al alto número de operaciones JOIN requeridas.
3. El dominio de TutorHUB no requiere granularidad temporal independiente por cada atributo individual.
4. No aporta beneficios prácticos al modelo del sistema.

---

## 9. Conclusión: Nivel de Normalización Adecuado

El modelo relacional de TutorHUB se encuentra normalizado hasta **5FN (Quinta Forma Normal)**, nivel óptimo que garantiza la ausencia de redundancia e inconsistencias sin incurrir en la sobrecomplejidad de la 6FN.

### Resumen del proceso de normalización:

| Paso | Forma Normal | Acción Realizada |
|------|-------------|------------------|
| 1 | 1FN | Descomposición de atributos no atómicos: "Nombre" en (Nombre, Apellido) y "Sesión" en (Fecha, Horas, Modalidad); resolución de relación N:M mediante tabla intermedia ESTUDIANTE_TUTORIA |
| 2 | 2FN | Verificación de ausencia de dependencias parciales en claves compuestas y simples |
| 3 | 3FN | Verificación de ausencia de dependencias transitivas entre atributos no primos |
| 4 | BCNF | Verificación de que todo determinante de una dependencia funcional es superclave |
| 5 | 4FN | Verificación de ausencia de dependencias multivaluadas no triviales |
| 6 | 5FN | Verificación de que todas las dependencias de unión están cubiertas por claves candidatas |
| 7 | 6FN | No aplicada por considerarse no justificada para los requerimientos del dominio |

### Esquema Relacional Final (8 tablas):

```
1. PERSONA (Código, Correo, Nombre, Apellido, Teléfono)
2. ESTUDIANTE (Código_estudiante, Programa_académico, Semestre)
3. TUTOR (Código_tutor, Escuela, Estado)
4. ASIGNATURA (Código_asignatura, Nombre)
5. LUGAR (Edificio, Aula, Capacidad)
6. TUTORIA (Código_tutoria, Nombre_del_tipo, Descripción, Cantidad_máxima_de_estudiantes,
            Fecha, Hora_de_inicio, Hora_de_finalización, Modalidad,
            Observaciones, Código_asignatura, Edificio, Aula, Código_tutor)
7. ESTUDIANTE_TUTORIA (Código_estudiante, Código_tutoria)
8. TUTORIA_ESTADO (Código_tutoria, Fecha_desde, Estado)
```

---

## 10. Cambios respecto al Modelo E-R Original

Se realizaron las siguientes transformaciones y ajustes formales al pasar del modelo E-R al modelo relacional:

1. **Atomicidad de Nombre (1FN):** Se dividió el atributo `Nombre` de Persona en `Nombre` y `Apellido` para cumplir estrictamente con la primera forma normal.
2. **Jerarquía Persona → Estudiante / Tutor:** Se implementó mediante tablas separadas por subtipo. Cada subtipo tiene como clave primaria y foránea su respectivo código referenciando a `PERSONA.Código` (`Código_estudiante` y `Código_tutor`).
3. **Atributo compuesto Sesión:** Se descompuso en sus componentes escalares dentro de `TUTORIA` (`Fecha`, `Hora_de_inicio`, `Hora_de_finalización`, `Modalidad`).
4. **Relación N:M Estudiante-Tutoría (Recibe):** Se implementó mediante la tabla intermedia `ESTUDIANTE_TUTORIA`.
5. **Relación Tutoría-Lugar (Se realiza en):** Se estructuró como relación 1:N mediante clave foránea compuesta `(Edificio, Aula)` en `TUTORIA`. Esto obedece a la regla de negocio de que cada sesión de tutoría tiene asignado un único espacio físico (o valor nulo si la modalidad es virtual).
6. **Clave primaria de LUGAR:** Se definió como clave compuesta `(Edificio, Aula)` para garantizar la unicidad de aulas en diferentes bloques o edificios.
7. **Tabla TUTORIA_ESTADO (nueva):** Se creó una tabla especializada para registrar la trazabilidad histórica de los estados de cada tutoría (`programada`, `realizada`, `cancelada`) junto con su fecha de cambio (`Fecha_desde`), superando el atributo estático simple del modelo E-R inicial.
8. **Relación 1:N Tutor-Tutoría (Realiza):** Se implementó como clave foránea `Código_tutor` en `TUTORIA`.
9. **Relación 1:N Asignatura-Tutoría (Corresponde):** Se implementó como clave foránea `Código_asignatura` en `TUTORIA`.
10. **Restricción de unicidad de horario por tutor (Grupo 1 / Unique compuesto):** Se definió una clave alternativa compuesta por `(Fecha, Hora_de_inicio, Código_tutor)` para asegurar la integridad de negocio: un mismo tutor no puede impartir dos tutorías distintas en el mismo horario y fecha.
