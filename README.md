# TutorHUB

**Sistema integral para la gestión de asesorías académicas.**

## Integrantes

| Nombre | Código |
|--------|--------|
| Alex Duvan Lopez Solano | 2242040 |
| Dayron Stiven Galeano Mejía | 2250153 |
| Brandon Andrés Jaimes Romero | 2250156 |
| Karoll Nataly Sánchez Ortega | 2250169 |

---

## Idea general del proyecto

TutorHUB es el diseño de una base de datos para un sistema que permite a una universidad **gestionar la reserva de tutorías** para sus estudiantes. Contempla tutorías **individuales** y **grupales**, la información de los **tutores**, sus **horarios** y los **lugares** donde se llevan a cabo.

El sistema maneja:

- **Personas**, que pueden ser estudiantes (con programa académico y semestre) o tutores (con escuela y estado activo/inactivo).
- **Asignaturas** sobre las que se ofrecen tutorías.
- **Tutorías**, cada una con su tipo, descripción, cupo máximo, sesión programada (fecha, horas y modalidad presencial o virtual) y estado (programada, realizada o cancelada).
- **Lugares** físicos (edificio, aula y capacidad) para las tutorías presenciales.
- **Inscripciones** de los estudiantes a las tutorías.

El proyecto recorre el proceso completo de diseño de una base de datos: desde el modelo conceptual hasta su implementación en SQL.

---

## Índice de entregas

| Entrega | Carpeta | Contenido | Guía de la carpeta |
|---------|---------|-----------|--------------------|
| **1** | [`Modelo_E-R`](./Modelo_E-R) | Diagrama entidad-relación y actividad exploratoria | [README.md](./Modelo_E-R/README.md) |
| **2** | [`Modelo Relacional`](./Modelo%20Relacional) | Diagrama relacional de ERDPlus, archivo de ERDPlus e informe de normalización | [README.md](./Modelo%20Relacional/README.md) |
| **3** | [`SQL`](./SQL) | Esquema, carga de datos y consultas | [README.md](./SQL/README.md) |

### Primera entrega – `Modelo_E-R`
Modelo conceptual del sistema: el diagrama entidad-relación y una actividad exploratoria con conceptos clave, tendencias y herramientas del mercado.

### Segunda entrega – `Modelo Relacional`
Paso del modelo conceptual al relacional: diagrama relacional hecho en ERDPlus, el archivo `.erdplus` editable y el informe de normalización de la base de datos.

### Tercera entrega – `SQL`
Implementación en MySQL: script del esquema, carga de datos de prueba y diez consultas relacionadas con el tema del proyecto.
