# Tercera entrega – SQL

## Índice de elementos

| Archivo | Contenido |
|---------|-----------|
| [`esquema.md`](./esquema.md) | Script de creación de la base de datos `tutorhub`: 8 tablas, restricciones, un trigger y una vista |
| [`datos.md`](./datos.md) | Datos de prueba, con más de 5 registros por tabla |
| [`consultas.md`](./consultas.md) | 10 consultas relacionadas con el tema del proyecto |

**Orden de ejecución:** esquema → carga de datos → consultas. 

---

## Descripción de lo generado

### Esquema (`esquema.md`)
Crea la base de datos `tutorhub` y las 8 tablas del modelo normalizado: `persona`, `estudiante`, `tutor`, `asignatura`, `lugar`, `tutoria`, `estudiante_tutoria` y `tutoria_estado`. Incluye:

- **Llaves primarias y foráneas** que respetan el diagrama relacional, con la jerarquía Persona → Estudiante/Tutor y la llave compuesta `(edificio, aula)` de `lugar`.
- **Restricciones de integridad** (`CHECK` y `UNIQUE`): tipo de tutoría válido, cupo positivo, cupo 1 para las individuales, hora de fin posterior a la de inicio, estados permitidos, correo y teléfono únicos, y que un tutor no tenga dos tutorías en la misma fecha y hora.
- **Regla de modalidad:** una tutoría presencial exige lugar y una virtual no puede tenerlo.
- **Trigger `trg_validar_cupo`:** impide inscribir más estudiantes que el cupo máximo de la tutoría.
- **Vista `v_tutoria_estado_actual`:** devuelve el estado vigente de cada tutoría a partir del historial.

### Carga de datos (`carga_datos.md`)
11 personas (6 estudiantes y 5 tutores), 6 asignaturas, 6 lugares, 9 tutorías, 23 inscripciones y 16 registros de historial de estados. Los datos cubren casos variados: tutorías individuales y grupales, presenciales y virtuales, realizadas, programadas y cancelada, un tutor inactivo y registros sin relacionar (una asignatura y un aula sin tutorías) para probar los `LEFT JOIN`.

### Consultas (`consultas.md`)
Diez consultas sobre agenda de tutorías, ocupación de cupos, asignaturas más solicitadas, carga de los tutores, historial de un estudiante, uso de aulas, tutorías canceladas con los correos a notificar, participación por programa académico, tutorías programadas con cupo disponible y resumen por modalidad y tipo.

---

## ¿Por qué se hizo así?

**Por qué MySQL.** Es un motor relacional muy extendido, gratuito y fácil de instalar, y a partir de la versión 8.0.16 aplica las restricciones `CHECK`, que el modelo necesita para proteger sus reglas de negocio.

**Por qué restricciones y trigger.** El enunciado del problema trae reglas que no se expresan solo con llaves: una tutoría presencial necesita lugar, una individual atiende a un solo estudiante y no se puede superar el cupo. Dejarlas en la base de datos garantiza que se cumplan sin importar desde qué aplicación se inserten los datos.

**Por qué un historial de estados.** El informe de normalización separó el estado en `tutoria_estado` para conservar la trazabilidad (cuándo se programó, realizó o canceló cada tutoría). La vista `v_tutoria_estado_actual` evita repetir en cada consulta la lógica para obtener el último estado.

**Por qué esos datos de prueba.** Se diseñaron para que cada consulta devuelva resultados con sentido y para ejercitar los casos límite: tutorías sin cupo libre, canceladas, virtuales sin lugar y entidades sin relaciones.

**Por qué esas consultas.** Responden a necesidades reales de un sistema de tutorías: los estudiantes necesitan ver qué tutorías tienen cupo, los tutores y coordinadores necesitan medir su carga y la ocupación de las aulas, y la universidad necesita estadísticas de participación para planear mejor sus recursos, que son algunas de las tendencias descritas en la actividad exploratoria.

**Decisiones de adaptación.** El teléfono se guarda como `VARCHAR`, porque un celular colombiano no cabe en un `INT`; los campos de fecha y hora usan `DATETIME` y los nombres van sin tildes para evitar problemas entre distintos sistemas. Los detalles están en `01_esquema.md`.
