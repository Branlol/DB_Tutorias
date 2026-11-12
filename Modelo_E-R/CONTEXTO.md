# Primera entrega – Modelo E-R

Esta carpeta contiene el **modelo conceptual** de TutorHUB y la investigación previa que lo sustenta.

## Índice de elementos

| Elemento | Descripción |
|----------|-------------|
| `Entidad_relacion.pdf` | Diagrama entidad-relación del sistema |
| `Actividad_exploratoria.md` | Investigación sobre conceptos, tendencias y herramientas del mercado |

---

## 1. Diagrama entidad-relación

### Cómo leer el diagrama

| Figura | Significado |
|--------|-------------|
| Rectángulo | Entidad |
| Óvalo | Atributo |
| Óvalo con texto subrayado | Atributo identificador (clave) |
| Óvalo con doble borde | Atributo multivaluado |
| Óvalo que agrupa otros óvalos | Atributo compuesto |
| Rombo | Relación entre entidades |
| Triángulo invertido | Jerarquía de generalización/especialización (relación «Es») |
| Etiquetas como `(1,n)`, `1:N`, `N:M` | Cardinalidad y tipo de la relación |

### Entidades y atributos

| Entidad | Atributos |
|---------|-----------|
| **Persona** | Correo, **Código**, Nombre, Apellido, Teléfono |
| **Estudiante** (subtipo de Persona) | Programa académico, Semestre |
| **Tutor** (subtipo de Persona) | Escuela, Estado |
| **Asignatura** | Nombre, **Código** |
| **Tutoría** | **Código**, Tipo, Descripción, Cantidad máxima de estudiantes, Estado, Observaciones y el atributo compuesto **Sesión** (Fecha, Hora de inicio, Hora de finalización, Modalidad) |
| **Lugar** | **Edificio**, **Aula**, Capacidad (clave compuesta por Edificio + Aula) |

### Relaciones

| Relación | Entidades | Tipo | Significado |
|----------|-----------|------|-------------|
| **Es** | Persona → Estudiante / Tutor | Jerarquía | Una persona es estudiante o tutor |
| **Recibe** | Estudiante – Tutoría | N:M | Un estudiante puede inscribirse a varias tutorías y una tutoría puede tener varios estudiantes |
| **Realiza** | Tutor – Tutoría | 1:N | Un tutor dicta varias tutorías; cada tutoría la dicta un solo tutor |
| **Corresponde** | Asignatura – Tutoría | 1:N | Cada tutoría pertenece a una asignatura; una asignatura puede tener muchas tutorías o ninguna |
| **Se realiza en** | Tutoría – Lugar | 1:N | Una tutoría presencial se da en un lugar; si es virtual no requiere lugar |

---

## 2. Actividad exploratoria

Documento de investigación que contextualiza el problema y justifica el diseño. Está organizado en tres grandes secciones.

### 1. Contexto del problema
Descripción del caso: una universidad que quiere gestionar la reserva de tutorías individuales y grupales. Detalla las reglas del negocio sobre personas, estudiantes, tutores, asignaturas, tutorías, sesiones, inscripciones y lugares.

### 2. Conceptos y tendencias actuales
- **Conceptos importantes:** tutor, estudiante, tutoría individual, tutoría grupal, reserva o agendamiento de citas y la importancia de estos conceptos para el diseño de la base de datos.
- **Otros conceptos:** tutoría virtual, tutoría híbrida, retroalimentación (*feedback*), analítica del aprendizaje (*learning analytics*) y asignaturas para dar tutorías, con ejemplos por programa académico.
- **Tendencias actuales en el área:** reserva en línea, sincronización con calendarios digitales, recordatorios automáticos, tutorías virtuales (Zoom, Teams), estadísticas de asistencia e integración con el sistema académico de la universidad.

### 3. Herramientas en el mercado
- **Q10:** software académico integral en la nube, usado en instituciones colombianas, que integra gestión académica, administrativa y aprendizaje virtual.
- **Sistema de agendamiento de tutorías de las UTS:** desarrollo local a la medida, con reserva por asignatura, días y horas disponibles, modalidades presencial o virtual y una ventana de 48 horas para que el tutor confirme.
