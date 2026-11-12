# TutorHUB – Esquema de la Base de Datos (MySQL)

Script de creación de la base de datos

## Decisiones de implementación

| Tema | Decisión |
|------|----------|
| Nombres | Identificadores en `snake_case` sin tildes (`codigo_tutoria`, `hora_finalizacion`) para evitar problemas de portabilidad. |
| `modalidad` | `BOOLEAN` (alias de `TINYINT(1)`): `TRUE` = presencial, `FALSE` = virtual. |
| `tutor.estado` | `BOOLEAN`: `TRUE` = activo, `FALSE` = inactivo. |
| `persona.telefono` | `VARCHAR(15)` único. Un celular colombiano (10 dígitos) desborda un `INT`. |
| Lugar | Si la tutoría es presencial **debe** tener `(edificio, aula)`; si es virtual **no** debe tenerlo (`CHECK`). |
| Cupo | Un trigger impide inscribir más estudiantes que `cantidad_maxima_estudiantes`. |
| Estado actual | La vista `v_tutoria_estado_actual` devuelve el último estado de cada tutoría según `tutoria_estado`. |
| Fechas con hora | `DATETIME` en lugar de `TIMESTAMP`, para evitar conversiones de zona horaria y el límite del año 2038. |
| FK de lugar | Usa `ON UPDATE RESTRICT` porque MySQL no permite que una columna con acción referencial `CASCADE` participe en un `CHECK`. |
| Codificación | `utf8mb4`, para guardar correctamente tildes y la letra ñ. |

## Script

```sql


CREATE DATABASE IF NOT EXISTS tutorhub
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE tutorhub;

-- Limpieza para poder re-ejecutar el script
SET FOREIGN_KEY_CHECKS = 0;
DROP VIEW  IF EXISTS v_tutoria_estado_actual;
DROP TABLE IF EXISTS tutoria_estado;
DROP TABLE IF EXISTS estudiante_tutoria;
DROP TABLE IF EXISTS tutoria;
DROP TABLE IF EXISTS lugar;
DROP TABLE IF EXISTS asignatura;
DROP TABLE IF EXISTS tutor;
DROP TABLE IF EXISTS estudiante;
DROP TABLE IF EXISTS persona;
SET FOREIGN_KEY_CHECKS = 1;

CREATE TABLE persona (
    codigo    INT          NOT NULL,
    correo    VARCHAR(80)  NOT NULL,
    nombre    VARCHAR(80)  NOT NULL,
    apellido  VARCHAR(80)  NOT NULL,
    telefono  VARCHAR(15)  NOT NULL,
    CONSTRAINT pk_persona          PRIMARY KEY (codigo),
    CONSTRAINT uq_persona_correo   UNIQUE (correo),
    CONSTRAINT uq_persona_telefono UNIQUE (telefono)
) ENGINE = InnoDB;

CREATE TABLE estudiante (
    codigo_estudiante   INT          NOT NULL,
    programa_academico  VARCHAR(80)  NOT NULL,
    semestre            INT          NOT NULL,
    CONSTRAINT pk_estudiante PRIMARY KEY (codigo_estudiante),
    CONSTRAINT fk_estudiante_persona FOREIGN KEY (codigo_estudiante)
        REFERENCES persona (codigo)
        ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT ck_estudiante_semestre CHECK (semestre BETWEEN 1 AND 12)
) ENGINE = InnoDB;

CREATE TABLE tutor (
    codigo_tutor  INT          NOT NULL,
    escuela       VARCHAR(80)  NOT NULL,
    estado        BOOLEAN      NOT NULL DEFAULT TRUE,
    CONSTRAINT pk_tutor PRIMARY KEY (codigo_tutor),
    CONSTRAINT fk_tutor_persona FOREIGN KEY (codigo_tutor)
        REFERENCES persona (codigo)
        ON UPDATE CASCADE ON DELETE CASCADE
) ENGINE = InnoDB;

CREATE TABLE asignatura (
    codigo_asignatura  INT          NOT NULL,
    nombre             VARCHAR(80)  NOT NULL,
    CONSTRAINT pk_asignatura PRIMARY KEY (codigo_asignatura)
) ENGINE = InnoDB;

CREATE TABLE lugar (
    edificio   VARCHAR(80)  NOT NULL,
    aula       VARCHAR(10)  NOT NULL,
    capacidad  INT          NOT NULL,
    CONSTRAINT pk_lugar PRIMARY KEY (edificio, aula),
    CONSTRAINT ck_lugar_capacidad CHECK (capacidad > 0)
) ENGINE = InnoDB;

CREATE TABLE tutoria (
    codigo_tutoria                 INT          NOT NULL,
    tipo                           VARCHAR(80)  NOT NULL,
    descripcion                    VARCHAR(200) NOT NULL,
    cantidad_maxima_estudiantes    INT          NOT NULL,
    fecha                          DATE         NOT NULL,
    hora_inicio                    TIME         NOT NULL,
    hora_finalizacion              TIME         NOT NULL,
    modalidad                      BOOLEAN      NOT NULL,
    observaciones                  TEXT         NOT NULL,
    codigo_asignatura              INT          NOT NULL,
    edificio                       VARCHAR(80)  NULL,
    aula                           VARCHAR(10)  NULL,
    codigo_tutor                   INT          NOT NULL,

    CONSTRAINT pk_tutoria PRIMARY KEY (codigo_tutoria),

    -- Un tutor no puede dictar dos tutorías en la misma fecha y hora
    CONSTRAINT uq_tutoria_horario_tutor UNIQUE (fecha, hora_inicio, codigo_tutor),

    CONSTRAINT fk_tutoria_asignatura FOREIGN KEY (codigo_asignatura)
        REFERENCES asignatura (codigo_asignatura)
        ON UPDATE CASCADE ON DELETE RESTRICT,
    CONSTRAINT fk_tutoria_tutor FOREIGN KEY (codigo_tutor)
        REFERENCES tutor (codigo_tutor)
        ON UPDATE CASCADE ON DELETE RESTRICT,
    CONSTRAINT fk_tutoria_lugar FOREIGN KEY (edificio, aula)
        REFERENCES lugar (edificio, aula)
        ON UPDATE RESTRICT ON DELETE RESTRICT,

    CONSTRAINT ck_tutoria_tipo       CHECK (tipo IN ('individual', 'grupal')),
    CONSTRAINT ck_tutoria_cupo       CHECK (cantidad_maxima_estudiantes > 0),
    CONSTRAINT ck_tutoria_individual CHECK (tipo <> 'individual' OR cantidad_maxima_estudiantes = 1),
    CONSTRAINT ck_tutoria_horas      CHECK (hora_finalizacion > hora_inicio),

    CONSTRAINT ck_tutoria_modalidad_lugar CHECK (
        (modalidad = TRUE  AND edificio IS NOT NULL AND aula IS NOT NULL) OR
        (modalidad = FALSE AND edificio IS NULL     AND aula IS NULL)
    )
) ENGINE = InnoDB;

CREATE TABLE estudiante_tutoria (
    codigo_estudiante  INT NOT NULL,
    codigo_tutoria     INT NOT NULL,
    CONSTRAINT pk_estudiante_tutoria PRIMARY KEY (codigo_estudiante, codigo_tutoria),
    CONSTRAINT fk_et_estudiante FOREIGN KEY (codigo_estudiante)
        REFERENCES estudiante (codigo_estudiante)
        ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT fk_et_tutoria FOREIGN KEY (codigo_tutoria)
        REFERENCES tutoria (codigo_tutoria)
        ON UPDATE CASCADE ON DELETE CASCADE
) ENGINE = InnoDB;

CREATE TABLE tutoria_estado (
    codigo_tutoria  INT          NOT NULL,
    fecha_desde     DATETIME     NOT NULL,
    estado          VARCHAR(15)  NOT NULL,
    CONSTRAINT pk_tutoria_estado PRIMARY KEY (codigo_tutoria, fecha_desde),
    CONSTRAINT fk_te_tutoria FOREIGN KEY (codigo_tutoria)
        REFERENCES tutoria (codigo_tutoria)
        ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT ck_te_estado CHECK (estado IN ('programada', 'realizada', 'cancelada'))
) ENGINE = InnoDB;

DROP TRIGGER IF EXISTS trg_validar_cupo;

DELIMITER $$

CREATE TRIGGER trg_validar_cupo
BEFORE INSERT ON estudiante_tutoria
FOR EACH ROW
BEGIN
    DECLARE v_max    INT;
    DECLARE v_actual INT;

    SELECT cantidad_maxima_estudiantes
      INTO v_max
      FROM tutoria
     WHERE codigo_tutoria = NEW.codigo_tutoria;

    SELECT COUNT(*)
      INTO v_actual
      FROM estudiante_tutoria
     WHERE codigo_tutoria = NEW.codigo_tutoria;

    IF v_actual >= v_max THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'La tutoría ya alcanzó su cupo máximo de estudiantes';
    END IF;
END$$

DELIMITER ;

CREATE VIEW v_tutoria_estado_actual AS
SELECT te.codigo_tutoria,
       te.estado,
       te.fecha_desde
FROM tutoria_estado te
JOIN (
    SELECT codigo_tutoria, MAX(fecha_desde) AS ultima_fecha
    FROM tutoria_estado
    GROUP BY codigo_tutoria
) ult ON ult.codigo_tutoria = te.codigo_tutoria
     AND ult.ultima_fecha   = te.fecha_desde;
```
