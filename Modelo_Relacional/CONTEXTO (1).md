# Segunda entrega – Modelo Relacional

Esta carpeta contiene el paso del modelo conceptual (E-R) al **modelo relacional**, junto con el análisis de normalización.

## Índice de elementos

| Elemento | Descripción |
|----------|-------------|
| Diagrama relacional | Diagrama exportado desde ERDPlus con las 8 tablas y sus llaves |
| `Tutorías.erdplus` | Archivo editable del diagrama en ERDPlus |
| `Informe_Normalización.md` | Informe de normalización de la base de datos |

---

## 1. Diagrama relacional

El diagrama tiene **8 tablas**. Cada tabla muestra sus columnas, su llave primaria (PK) y sus llaves foráneas (FK), y las flechas indican a qué tabla apunta cada FK.

### Tablas

| Tabla | Llave primaria | Descripción |
|-------|----------------|-------------|
| **Persona** | Código | Datos comunes: correo (único), nombre, apellido y teléfono (único) |
| **Estudiante** | Código_estudiante (también FK a Persona) | Programa académico y semestre |
| **Tutor** | Código_tutor (también FK a Persona) | Escuela y estado (activo/inactivo) |
| **Asignatura** | Código_asignatura | Nombre de la materia |
| **Lugar** | (Edificio, Aula) | Capacidad del espacio físico |
| **Tutoria** | Código_tutoría | Tipo, descripción, cupo máximo, fecha, horas, modalidad y observaciones. Referencia a Asignatura, Tutor y, de forma opcional, Lugar |
| **Estudiante_Tutoria** | (Código_estudiante, Código_tutoría) | Tabla intermedia de las inscripciones |
| **Tutoria_estado** | (Código_tutoría, Fecha_desde) | Historial de estados de cada tutoría |

### Relaciones y cómo funcionan

| Relación | Cómo funciona |
|----------|---------------|
| Persona → Estudiante | Cada estudiante es una persona: su llave primaria es a la vez foránea hacia `Persona.Código` |
| Persona → Tutor | Igual que el anterior: cada tutor es una persona |
| Asignatura → Tutoria | 1:N. Cada tutoría guarda el código de su asignatura |
| Tutor → Tutoria | 1:N. Cada tutoría guarda el código del tutor que la dicta |
| Lugar → Tutoria | 1:N. La FK compuesta `(Edificio, Aula)` es **opcional**: queda vacía cuando la tutoría es virtual |
| Estudiante ↔ Tutoria | N:M resuelta con `Estudiante_Tutoria`, que une un estudiante con una tutoría |
| Tutoria → Tutoria_estado | 1:N. Una tutoría acumula varios estados a lo largo del tiempo (programada, realizada, cancelada), cada uno con su fecha de inicio |

Además, el diagrama define que un mismo tutor no puede tener dos tutorías con la misma fecha y hora de inicio (grupo de unicidad compuesto).

---

## 2. Archivo de ERDPlus

`Tutorías.erdplus` es el archivo fuente del diagrama relacional. **Se puede subir a la página de ERDPlus** ([erdplus.com](https://erdplus.com)) para abrirlo, editarlo, volver a exportarlo como imagen o generar el script SQL del modelo.

---

## 3. Resumen del informe de normalización

El informe parte de una **relación universal** con todos los atributos del sistema y la lleva paso a paso hasta el modelo final.

| Forma normal | Qué se hizo |
|--------------|-------------|
| **1FN** | Se separó `Nombre` en `Nombre` y `Apellido`, se descompuso el atributo compuesto `Sesión` dentro de `Tutoria`, se resolvió la relación N:M con `Estudiante_Tutoria` y se creó `Tutoria_estado` para el historial |
| **2FN** | Se verificó que ninguna tabla tenga dependencias parciales sobre una llave compuesta |
| **3FN** | Se verificó que ningún atributo no clave dependa de otro atributo no clave |
| **BCNF** | Se verificó que todo determinante sea superclave (incluida la clave alternativa `Fecha + Hora_de_inicio + Código_tutor` en `Tutoria`) |
| **4FN** | No existen dependencias multivaluadas no triviales |
| **5FN** | Las dependencias de unión quedan cubiertas por las claves candidatas |
| **6FN** | No se aplicó: llevaría de 8 a más de 30 tablas, con más `JOIN` y sin beneficio real para el dominio |

**Conclusión:** el modelo queda normalizado hasta **5FN**, con 8 tablas, sin redundancia ni inconsistencias.

El informe cierra con los **cambios respecto al modelo E-R**: la jerarquía Persona → Estudiante/Tutor se implementó con tablas separadas, `Sesión` se descompuso en columnas de `Tutoria`, la relación con `Lugar` usa una FK compuesta opcional (nula en tutorías virtuales) y el estado de la tutoría pasó a una tabla de historial.
