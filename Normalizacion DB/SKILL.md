---
name: skill-normalizacion-db
display_name: Skill Normalización DB
description: >
  Usa esta skill cuando el usuario pida diseñar, analizar, refactorizar o normalizar
  una base de datos relacional en cualquiera de estos motores: PostgreSQL, MySQL,
  MariaDB, SQL Server o SQLite. Triggers: esquemas desordenados (0FN), tablas con
  celdas multivaluadas o datos redundantes, pedidos explícitos de "normalizar",
  "aplicar 1FN/2FN/3FN/BCNF", "diseñar base de datos", preguntas sobre dependencias
  funcionales, claves primarias/foráneas, anomalías de inserción/actualización/
  eliminación, solicitudes de DDL, modelado empresarial con catálogos/historial/
  auditoría, particionamiento, multitenancy, soft delete, seguridad de datos PII,
  migración sin downtime, o cualquier decisión de arquitectura de datos para los
  motores objetivo. NO uses esta skill para MongoDB u otros motores NoSQL,
  modelado dimensional puro (Star Schema, Data Vault), ni tuning de queries sin
  contexto de diseño de esquema.
version: "4.1"
engines: [PostgreSQL, MySQL, MariaDB, SQL Server, SQLite]
---

# Normalización de Bases de Datos Relacionales + Arquitectura Empresarial

Actúa como el arquitecto de datos líder en un equipo de ingeniería que entrega
sistemas sin deuda técnica desde el primer día. Tu trabajo no es aplicar reglas
mecánicamente — es diagnosticar el problema real del esquema, elegir el nivel
correcto de normalización para el contexto y motor específico, modelar las
entidades, atributos, eventos, catálogos, relaciones y agregados del dominio,
y explicar cada decisión con precisión quirúrgica. Un esquema mal diseñado hoy
es una crisis de integridad mañana.

Esta skill integra dos dimensiones:
- **Normalización formal** (1FN → 2FN → 3FN → BCNF → 4FN → 5FN) con
  diagnóstico de anomalías, dependencias funcionales y anti-patrones.
- **Arquitectura empresarial** (catálogos, historización, auditoría, retención,
  multitenancy, identificadores, integridad, índices, seguridad, partición y
  migración sin downtime) para producir esquemas listos para producción.

---

## ⚡ Inicio Rápido (30 segundos)

Si acabas de cargar esta skill y necesitas saber **por dónde empezar**, sigue
este flujo. No leas toda la skill: carga solo la fase que aplique.

### Paso 1 — Identifica el modo de trabajo

```
¿El usuario envió un esquema existente?
├── NO → Modo DISEÑO_DESDE_CERO → ir a FASE -1 (Descubrimiento de Dominio)
└── SÍ → Modo ANÁLISIS/REFACTOR → ir a FASE 0 (Anti-patrones críticos)
```

### Paso 2 — Haz las 3 preguntas de la FASE 1 (bloquean ejecución)

```
1. ¿Qué motor?  (PostgreSQL | MySQL | MariaDB | SQL Server | SQLite)
2. ¿OLTP u OLAP? (escrituras transaccionales vs lectura analítica)
3. ¿Tolerancia? (crítico: finanzas/salud/legal | tolerable: logs/métricas)
```

**Si el usuario no responde**, asume: PostgreSQL + OLTP + Crítico + sin
multitenancy + sin soft delete + PK BIGINT. Documenta cada supuesto.

### Paso 3 — Detecta dimensiones empresariales (palabras clave)

Escanea el mensaje del usuario buscando estas palabras. Si aparece alguna,
carga la fase correspondiente ADEMÁS del flujo base:

| Palabra clave del usuario | Fase a cargar |
|---|---|
| "multi-tenant", "varios clientes/empresas", "aislar datos por organización" | **FASE MULTITENANT** ⭐ |
| "PII", "datos sensibles", "GDPR", "HIPAA", "encriptar", "compliance" | **FASE SEGURIDAD** |
| "histórico de cambios", "valor anterior", "SCD", "línea de tiempo" | **FASE HISTORIZACIÓN** |
| "auditoría", "quién modificó", "trazabilidad", "log de cambios" | **FASE AUDITORÍA** |
| "no borrar", "soft delete", "recuperar registro eliminado" | **FASE RETENCIÓN** |
| "millones de filas", "crece rápido", "particionar", "archivo" | **FASE PARTICIONAMIENTO** |
| "migrar de...", "ya hay datos", "producción", "cero downtime" | **FASE MIGRACIÓN** |
| "UUID", "offline", "sistema distribuido", "microservicio" | **FASE IDENTIFICADORES** |
| "caro, grande, complejo", "chequeos de negocio", "reglas de validación" | **FASE INTEGRIDAD EMPRESARIAL** |
| "lento", "optimizar", "índice", "búsqueda rápida" | **FASE ÍNDICES** |
| "catálogo", "estados", "tipos", "lista de valores" | **FASE CATÁLOGOS** |
| "estado", "tipo", "categoría", "región" (literal) | **FASE CATÁLOGOS** |

### Paso 4 — Aplica el árbol de decisión de la FASE 2

```
¿OLTP?
├── SÍ + Crítico → BCNF + auditoría + soft delete + historización si aplica
├── SÍ + Tolerable → 3FN (evaluar BCNF si no degrada)
└── NO (OLAP) → Estrella + SCD2 en dimensiones que cambien
```

### Paso 5 — Entrega según la FASE 6 (contrato de output)

Tu respuesta final SIEMPRE debe tener las **9 secciones obligatorias** en
el orden exacto. Si el contexto no aplicó alguna (ej: usuario no pidió
multitenancy), igual muestra la sección con: "No aplica — usuario no
reportó necesidad de aislamiento entre tenants."

### Mapa mental de los 3 flujos más comunes

```
FLUJO A — "Normaliza este esquema"
  F0 → F1 → F2 → F3 → F5 → F6

FLUJO B — "Diseña desde cero un sistema X"
  F-1 → FCardinalidad → FCatálogos → F1 → F2 → F3 → F5 → F6
  (si multi-tenant) → añadir FMultitenant
  (si sensible) → añadir FSeguridad
  (si histórico) → añadir FHistorización
  (si auditoría) → añadir FAuditoría

FLUJO C — "Migra producción sin downtime"
  F0 → F1 → FMigración → F5 → F6
  (todo se hace con patrón expand-contract)
```

### Anti-errores frecuentes (checklist de 1 minuto)

Antes de responder, valida que NO estás cayendo en:

- ❌ Escribir DDL sin haber hecho las preguntas de Fase 1
- ❌ Elegir motor sin confirmar con el usuario
- ❌ Asumir "necesito UUID" sin preguntarlo
- ❌ Aplicar 3FN/BCNF a un caso OLAP (romper el flujo de F2)
- ❌ Saltarte la sección `[DIAGNÓSTICO ARQUITECTÓNICO]` y entregar solo el DDL
- ❌ No documentar supuestos cuando el usuario no responde
- ❌ Entregar DDL genérico sin adaptar al motor (Fase 5 es obligatoria)
- ❌ Olvidar el `PRAGMA foreign_keys = ON` en SQLite
- ❌ Olvidar `ENGINE=InnoDB` en MySQL/MariaDB
- ❌ Usar `NEWID()` en SQL Server en vez de `NEWSEQUENTIALID()`
- ❌ Confundir `utf8` con `utf8mb4` en MySQL
- ❌ Proponer `DROP COLUMN` directo en producción con datos

---

## 🧭 Índice de Navegación (Tabla de Contenidos)

Usa este índice para saltar directo a la fase o sección relevante. **No leas
la skill completa en cada turno**: carga solo la fase que aplique al contexto
del usuario y referencia las demás cuando las necesites.

### Parte I — Núcleo de Normalización
- [FASE 0 — Detección Silenciosa de Anti-Patrones Críticos](#fase-0--detección-silenciosa-de-anti-patrones-críticos)
- [FASE 1 — Diagnóstico Obligatorio (Bloqueo de Ejecución)](#fase-1--diagnóstico-obligatorio-bloqueo-de-ejecución)
- [FASE 2 — Árbol de Decisión Arquitectónico](#fase-2--árbol-de-decisión-arquitectónico)
- [FASE 3 — Protocolos de Ejecución por Forma Normal (OLTP)](#fase-3--protocolos-de-ejecución-por-forma-normal-oltp)
  - Paso 0: Estado Inicial (0FN)
  - Paso 1: Primera Forma Normal (1FN)
  - Paso 2: Segunda Forma Normal (2FN)
  - Paso 3: Tercera Forma Normal (3FN)
  - Paso 4: BCNF
- [FASE 4 — Formas Superiores (4FN / 5FN)](#fase-4--formas-superiores-solo-si-el-contexto-lo-exige)
- [FASE 5 — Diferencias por Motor: DDL y Restricciones Nativas](#fase-5--diferencias-por-motor-ddl-y-restricciones-nativas)
- [FASE 6 — Contrato de Output (Entrega Obligatoria)](#fase-6--contrato-de-output-entrega-obligatoria)

### Parte II — Arquitectura Empresarial
- [FASE -1 — Descubrimiento de Dominio](#fase-1--descubrimiento-de-dominio-arquitectura-empresarial)
- [FASE CARDINALIDAD — Determinación Explícita](#fase-cardinalidad--determinación-explícita)
- [FASE CATÁLOGOS — Detección Automática](#fase-catálogos--detección-automática)
- [FASE HISTORIZACIÓN — Detección de Atributos Temporales](#fase-histórización--detección-de-atributos-temporales)
- [FASE AUDITORÍA — Trazabilidad Completa de Cambios](#fase-auditoría--trazabilidad-completa-de-cambios)
- [FASE RETENCIÓN — Política de Eliminación](#fase-retención--política-de-eliminación)
- [FASE MULTITENANT — Aislamiento por Tenant](#fase-multitenant--aislamiento-por-tenant) ⭐
- [FASE IDENTIFICADORES — Estrategia de PK](#fase-identificadores--estrategia-de-pk)
- [FASE INTEGRIDAD EMPRESARIAL — Restricciones de Negocio](#fase-integridad-empresarial--restricciones-de-negocio)
- [FASE ÍNDICES — Optimización de Acceso](#fase-índices--optimización-de-acceso)
- [FASE SEGURIDAD — Clasificación de Datos Sensibles](#fase-seguridad--clasificación-de-datos-sensibles)
- [FASE PARTICIONAMIENTO — Escalabilidad Horizontal](#fase-particionamiento--escalabilidad-horizontal)
- [FASE MIGRACIÓN SIN DOWNTIME — Despliegue Seguro](#fase-migración-sin-downtime--despliegue-seguro)

### Parte III — Referencia y Plantillas
- [Reglas de Nomenclatura](#reglas-de-nomenclatura)
- [Anti-Patrones de PKs por Motor](#anti-patrones-de-pks--tabla-de-referencia-por-motor)
- [Ejemplos End-to-End Completos](#-ejemplos-end-to-end-completos)
- [Anti-Patrones Avanzados](#-anti-patrones-avanzados)
- [Herramientas de Migración](#-herramientas-de-migración)
- [Guía de Decisión Rápida](#-guía-de-decisión-rápida)

---

## 🎯 Guía de Decisión Rápida (Qué Fase Cargar Según el Contexto)

**Carga solo las fases que el contexto del usuario demande.** El resto se
ignora hasta que sea necesario. Esto evita respuestas sobreextendidas.

| Si el usuario pide... | Cargar estas fases | Ignorar |
|---|---|---|
| "Normaliza este esquema" / "Aplica 1FN/2FN/3FN" | Fases 0, 1, 2, 3, 6 | -1, Cardinalidad, Catálogos, Hist, Audit, Ret, Multitenant, Ident, Integ, Índices, Seg, Part, Migr |
| "Diseña una base de datos" desde cero | Fases -1, 1, 2, Cardinalidad, 3, 5, 6, Nomenclatura | 4FN/5FN (raras) |
| "Refactoriza a multi-tenant" | Fases 1, Multitenant ⭐, Identificadores, 5, 6, Migración | Hist, Audit (a menos que se pida) |
| "Necesito auditoría completa" | Fases 1, Auditoría, Retención, Identificadores, 5, 6 | Multitenant, Particionamiento |
| "Necesito saber histórico de X" | Fases 1, Historización, Identificadores, 5, 6 | -1, Multitenant |
| "Migra esto sin downtime" | Fases 1, Migración, Auditoría, Identificadores, 5 | -1, Cardinalidad, Hist, Part |
| "El sistema está lento" | Fases 0, Índices, Particionamiento, 5, 6 | -1, Hist, Audit, Ret, Multitenant |
| "Tengo datos sensibles (PII/salud/finanzas)" | Fases 1, Seguridad, Identificadores, 5, 6, Migración | -1, Catálogos, Hist, Part |
| "Tabla con 50+ columnas" | Fases 0, -1, Cardinalidad, 3, 5, 6 | Audit, Multitenant, Part |
| "Esquema de [e-commerce / SaaS / salud / finanzas]" | Saltar a [Ejemplos End-to-End](#-ejemplos-end-to-end-completos) y adaptar | - |
| "Usa Flyway/Liquibase/Alembic/sqitch" | Saltar a [Herramientas de Migración](#-herramientas-de-migración) | - |

**Regla de oro:** si el usuario no especifica contexto, cargar **Fases 0,
1, 2, 5, 6** como mínimo viable. El resto se activa cuando el usuario
responda las preguntas de la Fase 1 o lo pida explícitamente.

---

## FASE 0 — Detección Silenciosa de Anti-Patrones Críticos

Antes de interactuar con el usuario, analiza el esquema buscando estos cinco
anti-patrones. Si detectas alguno, exponlo inmediatamente como riesgo crítico
al comienzo de tu respuesta, antes de cualquier otra cosa.

**1. Anomalías de Actualización:**
Datos redundantes que obligarán a actualizar múltiples filas para un solo cambio
lógico. Riesgo: inconsistencia silenciosa cuando una actualización parcial falla.
Señal: el mismo valor de negocio aparece literal en más de una fila.

**2. Abuso de Valores NULL:**
Atributos que estarán vacíos en la mayoría de las tuplas. Riesgo: desperdicio
de almacenamiento, lógica de JOINs complicada y resultados inesperados en
agregaciones (COUNT, AVG ignoran NULLs). Señal: columnas opcionales que solo
aplican a un subconjunto de filas — indicio de que hay dos entidades mezcladas.

**3. Tuplas Falsas (Spurious Tuples):**
Diseños que al hacer un JOIN natural generan combinaciones de datos que no
existen en el mundo real. Causa: FK mal definida o relación sin tabla
de intersección correcta. Señal: el JOIN de dos tablas produce más filas
que las esperadas por el negocio.

**4. Columnas "Comodín":**
`campo_extra`, `datos_adicionales`, `misc`, `json_blob` usados para esconder
entidades sin modelar. Riesgo: imposible de indexar, validar o consultar
de forma eficiente. Acción: siempre preguntar qué datos van ahí y modelarlos.
JSONB/TEXT(json) solo es aceptable para datos semiestructurados ocasionales
y documentados, nunca como sustituto de entidad.

**5. Tablas "Dios":**
Tablas con más de 25-30 columnas son señal de que se mezclan dos o más entidades
del dominio. Acción: identificar las entidades implícitas y descomponerlas antes
de aplicar formas normales.

---

## FASE -1 — Descubrimiento de Dominio (Arquitectura Empresarial)

Antes de evaluar formas normales, identificar el modelo de dominio completo:

- **Entidades** (cosas con identidad propia: Cliente, Producto, Pedido)
- **Atributos** (propiedades de las entidades)
- **Eventos** (hechos del negocio: Compra, Cancelación, Pago)
- **Catálogos** (listas discretas de valores válidos)
- **Relaciones** (vínculos entre entidades)
- **Agregados** (grupos de entidades que se modifican juntas)

Clasificar cada tabla resultante como una y solo una de estas categorías:

- `[MAESTRO]` — datos casi estáticos (Clientes, Productos, Empleados)
- `[TRANSACCIONAL]` — hechos del negocio (Pedidos, Facturas, Movimientos)
- `[CATÁLOGO]` — listas de valores discretos (estados, tipos, regiones)
- `[INTERSECCIÓN]` — resolución de relaciones N:M
- `[HISTÓRICO]` — registro temporal de cambios (SCD)
- `[AUDITORÍA]` — log inmutable de operaciones

Documentar la categoría junto al nombre de cada tabla en el esquema final.

---

## FASE CARDINALIDAD — Determinación Explícita

Determinar explícitamente la cardinalidad de cada relación:

- **1:1** — una entidad A posee exactamente una entidad B y viceversa
- **1:N** — una entidad A posee muchas entidades B; cada B pertenece a una A
- **N:M** — muchas A se relacionan con muchas B (requiere tabla de intersección)

**Preguntas obligatorias por cada relación:**

1. ¿Puede existir B sin A? (define obligatoriedad y dirección)
2. ¿La relación es obligatoria? (¿toda A debe tener al menos una B?)
3. ¿Puede repetirse? (¿una A puede tener varias B en el tiempo?)

Documentar cardinalidades en el diagrama de relaciones:

```
Cliente (1) ────── (N) Pedido
Pedido  (1) ────── (N) Línea_Pedido (N) ────── (1) Producto
Empleado (N) ────── (M) Proyecto
```

---

## FASE CATÁLOGOS — Detección Automática

Detectar columnas con dominios discretos y de baja cardinalidad:

- `estado`
- `tipo`
- `categoría`
- `región`
- `país`
- `moneda`
- `rol`
- `prioridad`
- `canal`

**Criterio de promoción a tabla catálogo:**
Si los valores se repiten en múltiples filas, se usan en reglas de negocio,
se filtran frecuentemente, o se proyecta que crecerán (nuevos tipos/estados),
crear una tabla catálogo dedicada.

Ejemplo de transformación:

```sql
-- ANTES: estado como literal repetido
CREATE TABLE pedidos (
    id     INT PRIMARY KEY,
    estado VARCHAR(20)  -- 'Pendiente', 'Procesado', 'Anulado'
);

-- DESPUÉS: estado como FK a catálogo
CREATE TABLE catalogo_estado_pedido (
    id     INT PRIMARY KEY,
    codigo VARCHAR(20)  NOT NULL UNIQUE,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE pedidos (
    id                INT PRIMARY KEY,
    estado_id         INT NOT NULL REFERENCES catalogo_estado_pedido(id)
);
```

**Beneficio clave:** permite añadir restricciones CHECK, traducciones
(i18n), activar/desactivar valores, y mantener trazabilidad histórica
incluso cuando un valor de catálogo desaparece.

---

## FASE HISTORIZACIÓN — Detección de Atributos Temporales

Detectar atributos cuyo valor cambia a lo largo del tiempo y donde
conocer el valor histórico es relevante para el negocio:

- `salario`
- `cargo` / `puesto`
- `departamento`
- `precio` (de lista, de costo, de venta)
- `estado`
- `ubicación` / `dirección`
- `categoría de cliente`
- `plan` / `suscripción`
- `gerente` / `supervisor`

**Pregunta obligatoria al usuario:**

> "¿Necesitas conocer valores históricos de estos atributos?
> (Sí / No / Solo para algunos)"

**Si la respuesta es SÍ → implementar Slow Changing Dimensions:**

### Opción A — Columna de vigencia en la misma tabla (SCD Tipo 2 simple)

```sql
CREATE TABLE empleado_cargo (
    id            INT PRIMARY KEY,
    empleado_id   INT NOT NULL REFERENCES empleados(id),
    cargo_id      INT NOT NULL REFERENCES catalogo_cargo(id),
    fecha_inicio  DATE NOT NULL,
    fecha_fin     DATE,                 -- NULL = vigente
    es_actual     BOOLEAN NOT NULL DEFAULT TRUE,
    CHECK (fecha_fin IS NULL OR fecha_fin >= fecha_inicio)
);

-- Solo puede haber un cargo vigente por empleado
CREATE UNIQUE INDEX uq_empleado_cargo_actual
    ON empleado_cargo(empleado_id) WHERE es_actual = TRUE;
```

### Opción B — Tabla de historial dedicada (SCD Tipo 4)

```sql
CREATE TABLE empleado_salario_historial (
    id            BIGINT PRIMARY KEY,
    empleado_id   INT NOT NULL REFERENCES empleados(id),
    salario       NUMERIC(12,2) NOT NULL CHECK (salario > 0),
    fecha_inicio  DATE NOT NULL,
    fecha_fin     DATE,
    motivo        VARCHAR(200),
    autorizado_por INT REFERENCES empleados(id)
);
```

Documentar en la sección `[ESTRATEGIA DE HISTORIZACIÓN]` qué entidades
requieren historial, qué técnica se usó (SCD 1/2/3/4) y por qué.

---

## FASE AUDITORÍA — Trazabilidad Completa de Cambios

**Pregunta obligatoria al usuario:**

> "¿Necesitas trazabilidad completa de cambios (quién, cuándo, qué cambió)?
> (Sí / No / Solo en tablas críticas)"

**Si la respuesta es SÍ, aplicar dos niveles:**

### Nivel 1 — Columnas de auditoría en cada tabla

```sql
CREATE TABLE pedidos (
    id           BIGINT PRIMARY KEY,
    -- columnas de negocio
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by   INT NOT NULL REFERENCES usuarios(id),
    updated_at   TIMESTAMPTZ,
    updated_by   INT REFERENCES usuarios(id),
    deleted_at   TIMESTAMPTZ,         -- soft delete
    deleted_by   INT REFERENCES usuarios(id)
);
```

### Nivel 2 — Tablas de auditoría dedicadas (audit log)

Para sistemas con compliance (finanzas, salud, legal), crear tablas
de auditoría inmutables que registran TODA operación:

```sql
CREATE TABLE auditoria_pedido (
    id              BIGINT PRIMARY KEY,
    pedido_id       BIGINT NOT NULL,
    operacion       VARCHAR(10) NOT NULL,  -- INSERT/UPDATE/DELETE
    datos_antes     JSONB,                  -- snapshot anterior
    datos_despues   JSONB,                  -- snapshot nuevo
    usuario_id      INT NOT NULL,
    ip_origen       INET,
    user_agent      TEXT,
    timestamp_audit TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Inmutabilidad: solo INSERT permitido
REVOKE UPDATE, DELETE ON auditoria_pedido FROM app_user;
```

Documentar en `[ESTRATEGIA DE AUDITORÍA]` las columnas y tablas aplicadas,
y los triggers o políticas que las mantienen.

---

## FASE RETENCIÓN — Política de Eliminación

**Pregunta obligatoria al usuario:**

> "¿Los registros pueden eliminarse físicamente o se requiere
> conservación (soft delete)?"

### Opción A — Eliminación física (DELETE)

```sql
DELETE FROM pedidos WHERE id = 123;
```

Adecuada solo para datos sin valor histórico ni regulatorio
(por ejemplo: carritos abandonados, sesiones expiradas).

### Opción B — Soft Delete (recomendado para datos transaccionales)

```sql
CREATE TABLE pedidos (
    id            BIGINT PRIMARY KEY,
    -- columnas
    activo        BOOLEAN NOT NULL DEFAULT TRUE,
    fecha_eliminacion TIMESTAMPTZ,
    eliminado_por    INT REFERENCES usuarios(id)
);

-- Las queries operativas filtran por activo = TRUE (índice parcial)
CREATE INDEX idx_pedidos_activos ON pedidos(id) WHERE activo = TRUE;
```

### Opción C — Hard delete diferido con grace period

Combinación: soft delete en aplicación, job programado que purga
físicamente después de N días (cumple regulaciones de "derecho al olvido").

Documentar la política y la cascada:

```sql
-- Las FK deben respetar la política
ALTER TABLE lineas_pedido
    ADD CONSTRAINT fk_linea_pedido
    FOREIGN KEY (pedido_id) REFERENCES pedidos(id)
    ON DELETE RESTRICT;  -- no borrar pedidos con líneas

-- O permitir cascada lógica vía aplicación
```

---

## FASE MULTITENANT — Aislamiento por Tenant

**Detectar si el sistema es multitenant** buscando columnas como:

- `tenant_id`
- `empresa_id`
- `organizacion_id`
- `cuenta_id`

**Reglas obligatorias:**

1. **Toda tabla de negocio debe tener `tenant_id NOT NULL`** y una FK
   a la tabla de tenants.
2. **Toda FK entre tablas de negocio debe respetar el tenant**: al insertar
   un hijo, su `tenant_id` debe coincidir con el del padre. Implementar
   con triggers o composite FKs.
3. **Todo índice debe comenzar con `tenant_id`** para que las queries
   multi-tenant no escaneen datos de otros tenants.
4. **Row Level Security** (PostgreSQL) o políticas equivalentes para
   defensa en profundidad.

```sql
-- PostgreSQL: RLS como red de seguridad
ALTER TABLE pedidos ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON pedidos
    USING (tenant_id = current_setting('app.tenant_id')::INT);

-- Cualquier intento de leer datos de otro tenant es rechazado por el motor
```

```sql
-- MySQL/SQL Server: vista filtrada por sesión
CREATE VIEW v_pedidos AS
    SELECT * FROM pedidos
    WHERE tenant_id = @app_tenant_id;
```

**Advertencia crítica:** si una FK entre tablas multitenant cruza tenants
diferentes, la integridad referencial se rompe. Generar siempre un check:

```sql
-- Trigger de validación
CREATE FUNCTION fn_validar_tenant() RETURNS TRIGGER AS $$
BEGIN
    IF NEW.tenant_id != (SELECT tenant_id FROM pedidos WHERE id = NEW.pedido_id) THEN
        RAISE EXCEPTION 'Tenant mismatch: viola aislamiento';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

## FASE IDENTIFICADORES — Estrategia de PK

Seleccionar la estrategia de PK según el contexto del sistema:

| Contexto | PK recomendada | Razón |
|----------|----------------|-------|
| Monolito simple | `BIGINT AUTO_INCREMENT/IDENTITY` | Eficiencia en JOINs e índices clustered |
| Sistemas distribuidos / microservicios | `UUID v4` o `UUID v7` | Generación sin coordinación |
| Offline-first / mobile sync | `UUID` | Permite generar IDs sin conexión |
| Integraciones externas (APIs públicas) | `UUID` | No expone volumen ni secuencia de negocio |
| Data warehouse / append-only | `BIGINT AUTO_INCREMENT` o `UUID` | Volumen alto, no edita |
| Eventos / logs / auditoría | `UUID` o `(timestamp, sequence)` | Permite ordenamiento temporal |

**Reglas universales:**

- PKs **surrogadas** (generadas por el sistema), **estables** (nunca cambian),
  **únicas** y **sin significado de negocio**.
- Email, código de producto, número de documento: candidatos a `UNIQUE`, no a PK.
- En SQL Server con `UNIQUEIDENTIFIER`, usar `NEWSEQUENTIALID()` (no `NEWID()`)
  para evitar fragmentación del índice clustered.
- En PostgreSQL 13+, preferir `GENERATED ALWAYS AS IDENTITY` sobre `SERIAL`.

**Documentar en la entrega la razón de la elección:**

```
PK Strategy: UUID v7
Razón: sistema distribuido con 3 servicios emisores de pedidos.
       UUID v7 agrega timestamp, permitiendo ordenamiento cronológico
       sin sacrificar la generación descentralizada.
```

---

## FASE INTEGRIDAD EMPRESARIAL — Restricciones de Negocio

Generar automáticamente restricciones CHECK que reflejen reglas del negocio
—no limitarse a PK, FK, NOT NULL y UNIQUE.

**Patrones frecuentes:**

```sql
-- Stock nunca negativo
CHECK (stock >= 0)

-- Precios positivos
CHECK (precio > 0)

-- Vigencia temporal coherente
CHECK (fecha_fin IS NULL OR fecha_fin >= fecha_inicio)

-- Edad mínima legal
CHECK (edad >= 18)

-- Descuento dentro de rango válido
CHECK (descuento BETWEEN 0 AND 100)

-- Salario mínimo vs. máximo por categoría
CHECK (salario BETWEEN salario_minimo AND salario_maximo)

-- Cantidad de hijos no negativa
CHECK (cantidad_hijos >= 0)

-- Email con formato válido (en capa de aplicación + CHECK básico)
CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')

-- Estados mutuamente excluyentes
CHECK (NOT (anulado = TRUE AND entregado = TRUE))

-- Coherencia de totales
CHECK (total = subtotal - descuento + impuestos)
```

**Regla:** si el negocio expresa una invariante, debe existir una restricción
que la haga cumplir automáticamente.

---

## FASE ÍNDICES — Optimización de Acceso

Identificar los patrones de acceso:

- JOINs frecuentes
- Filtros (`WHERE`)
- Búsquedas (`LIKE`, `ILIKE`, full-text)
- Ordenamientos (`ORDER BY`)
- Reportes y agregaciones

**Generar índices según el motor:**

```sql
-- Índice simple en FK (casi siempre necesario)
CREATE INDEX idx_pedidos_cliente ON pedidos(cliente_id);

-- Índice único (constraints de unicidad de negocio)
CREATE UNIQUE INDEX uq_clientes_email ON clientes(email);

-- Índice compuesto (orden de columnas = orden de selectividad)
CREATE INDEX idx_pedidos_cliente_fecha
    ON pedidos(cliente_id, fecha DESC);

-- Índice cubridor (evita lookups a la tabla base)
CREATE INDEX idx_pedidos_reporte
    ON pedidos(cliente_id, fecha)
    INCLUDE (total, estado_id);

-- Índice parcial (optimiza queries sobre subset)
CREATE INDEX idx_pedidos_activos
    ON pedidos(cliente_id) WHERE activo = TRUE;

-- Índice único parcial (garantiza una sola versión vigente)
CREATE UNIQUE INDEX uq_empleado_cargo_actual
    ON empleado_cargo(empleado_id) WHERE es_actual = TRUE;
```

**Justificar cada índice:** indicar la query que lo motiva. No crear índices
"por si acaso" — cada índice tiene costo de mantenimiento en escrituras.

**Anti-patrón:** índices redundantes. Un índice `(a, b)` cubre queries por `a`
solo, pero NO por `b` solo. Revisar siempre.

---

## FASE SEGURIDAD — Clasificación de Datos Sensibles

**Pregunta obligatoria al usuario:**

> "¿Existen datos sensibles? (PII, finanzas, salud, credenciales, datos legales)"

**Si la respuesta es SÍ, clasificar:**

| Categoría | Ejemplos | Riesgo |
|-----------|----------|--------|
| PII identificable | nombre, email, teléfono, dirección, DNI | Suplantación, doxxing |
| PII sensible | origen racial, opinión política, salud, orientación sexual, religión | Discriminación |
| Financiera | salario, cuenta bancaria, tarjeta | Fraude, robo |
| Salud | diagnósticos, tratamientos, medicación | Discriminación laboral, seguros |
| Credenciales | password, token, API key, private key | Compromiso total del sistema |
| Legal | contratos, litigios, NIF de empresa | Cumplimiento normativo |
| Menores | datos de menores de 14 años | Consentimiento parental obligatorio |

**Recomendaciones según motor:**

```sql
-- PostgreSQL: encriptación a nivel de columna
CREATE EXTENSION pgcrypto;
ALTER TABLE clientes
    ADD COLUMN email_encrypted BYTEA;

UPDATE clientes
    SET email_encrypted = pgp_sym_encrypt(email, current_setting('app.encryption_key'));

-- PostgreSQL: Row Level Security
ALTER TABLE empleados ENABLE ROW LEVEL SECURITY;
CREATE POLICY empleados_visibles ON empleados
    USING (departamento_id = current_setting('app.departamento_id')::INT
           OR current_setting('app.rol') = 'admin');

-- SQL Server: Always Encrypted para columnas sensibles
ALTER TABLE clientes
    ADD email_encrypted VARBINARY(256);

-- Data Masking (no encripta, pero ofusca para no autorizados)
ALTER TABLE clientes
    ALTER COLUMN email ADD MASKED WITH (FUNCTION = 'email()');

-- Aplicación: hashing de credenciales con bcrypt/argon2
-- NUNCA almacenar passwords en texto plano
-- NUNCA usar MD5 o SHA1 para passwords
```

**Cumplimiento normativo común:**

- GDPR (UE): derecho al olvido, portabilidad, consentimiento explícito
- LOPD (España): análogo a GDPR
- LFPDPPP (México): principios de consentimiento, información, calidad
- HIPAA (EEUU salud): encriptación en tránsito y reposo
- PCI-DSS (pagos): nunca almacenar CVV, tokenización de tarjetas

**Documentar en `[ANÁLISIS DE SEGURIDAD]`:**
- Datos sensibles detectados (clasificados por categoría)
- Recomendaciones aplicadas (encriptación, RLS, masking, hashing)
- Normativas que aplican y su cumplimiento

---

## FASE PARTICIONAMIENTO — Escalabilidad Horizontal

**Evaluar particionamiento si se cumple alguno:**

- Volumen esperado > 10 millones de filas
- Tamaño de tabla > 100 GB
- Crecimiento acelerado (logs, eventos, series temporales)
- Queries por rango temporal (último mes, último año)

**Estrategia por motor:**

### PostgreSQL

```sql
-- Particionamiento por rango (fecha, id, hash)
CREATE TABLE eventos (
    id         BIGINT GENERATED ALWAYS AS IDENTITY,
    tenant_id  INT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL,
    datos      JSONB
) PARTITION BY RANGE (created_at);

-- Crear particiones por mes
CREATE TABLE eventos_2026_01 PARTITION OF eventos
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

CREATE TABLE eventos_2026_02 PARTITION OF eventos
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');

-- Partición por hash (distribución equitativa)
CREATE TABLE sesiones (
    id        BIGINT,
    usuario_id INT NOT NULL
) PARTITION BY HASH (usuario_id);

CREATE TABLE sesiones_p0 PARTITION OF sesiones
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
```

### MySQL / MariaDB

```sql
CREATE TABLE eventos (
    id         BIGINT NOT NULL AUTO_INCREMENT,
    created_at DATETIME NOT NULL,
    datos      JSON
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  PARTITION BY RANGE (YEAR(created_at) * 100 + MONTH(created_at)) (
    PARTITION p202601 VALUES LESS THAN (202602),
    PARTITION p202602 VALUES LESS THAN (202603),
    PARTITION p202603 VALUES LESS THAN (202604),
    PARTITION pmax    VALUES LESS THAN MAXVALUE
);
```

### SQL Server

```sql
-- Crear función y esquema de partición
CREATE PARTITION FUNCTION pfEventosFecha (DATETIME2)
    AS RANGE RIGHT FOR VALUES
    ('2026-01-01', '2026-02-01', '2026-03-01', '2026-04-01');

CREATE PARTITION SCHEME psEventosFecha
    AS PARTITION pfEventosFecha ALL TO ([PRIMARY]);

CREATE TABLE eventos (
    id         BIGINT IDENTITY(1,1),
    created_at DATETIME2 NOT NULL,
    datos      NVARCHAR(MAX)
) ON psEventosFecha(created_at);
```

### SQLite

SQLite no soporta particionamiento nativo. Workaround: bases de datos
separadas por período (`eventos_2026q1.db`, `eventos_2026q2.db`) con
vistas UNION en la aplicación.

**Documentar en `[ANÁLISIS DE ESCALABILIDAD]`:**

- Crecimiento esperado (filas/mes, GB/mes)
- Necesidad de particionamiento (sí/no, tipo, criterio)
- Riesgos de bloqueo (tablas muy grandes, lock escalation)
- Estrategia de archivo/archivado (cold storage, compresión)

---

## FASE MIGRACIÓN SIN DOWNTIME — Despliegue Seguro

Si existe producción con datos, NUNCA aplicar cambios destructivos
directamente. Usar el patrón **expand-contract**:

### Paso 1 — EXPAND (añadir sin romper)

```sql
-- Crear nueva tabla o columnas en paralelo
CREATE TABLE clientes_new (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(200) NOT NULL UNIQUE
    -- nuevo esquema
);
```

### Paso 2 — BACKFILL (copiar datos)

```sql
-- Backfill en lotes para no bloquear
INSERT INTO clientes_new (id, email)
SELECT id, email FROM clientes_old
WHERE id BETWEEN 1 AND 10000
ON CONFLICT DO NOTHING;
```

### Paso 3 — VALIDATE (verificar integridad)

```sql
-- Comparar conteos, sums, checksums
SELECT COUNT(*) FROM clientes_old;
SELECT COUNT(*) FROM clientes_new;

-- Verificar huérfanos
SELECT COUNT(*) FROM lineas_pedido lp
LEFT JOIN clientes_new c ON c.id = lp.cliente_id
WHERE c.id IS NULL;
```

### Paso 4 — DUAL WRITE (escritura doble)

Modificar la aplicación para escribir en ambas tablas (vieja y nueva)
durante el período de transición. Backfill continúa para datos históricos.

### Paso 5 — SWITCH (cambiar lecturas)

Cambiar las queries de lectura a la nueva tabla. Mantener la vieja
para auditoría y rollback.

### Paso 6 — CLEANUP (retirar esquema antiguo)

```sql
-- Solo después de validar que todo funciona
DROP TABLE clientes_old;
```

**Reglas críticas:**

- NUNCA usar `ALTER TABLE ... DROP COLUMN` en producción con datos,
  hasta validar que ninguna query o procedimiento la referencia.
- Cambios de tipo de dato: crear nueva columna, backfill, swap, drop.
- Cambios de FK: crear nueva columna FK, backfill, validar,
  swap en aplicación, drop columna vieja.
- Cada paso debe tener un script de rollback.

**Documentar en `[PLAN DE MIGRACIÓN]`:**

- Pasos exactos de despliegue con orden de ejecución
- Tiempo estimado de cada paso
- Riesgos por paso
- Scripts de rollback por paso
- Criterios de Go/No-Go entre pasos

---

## FASE 1 — Diagnóstico Obligatorio (Bloqueo de Ejecución)

**NO sugieras ninguna tabla ni escribas DDL hasta haber recibido respuesta
a estas tres preguntas.** Si el usuario envía un esquema sin contexto, detente
y solicita las respuestas antes de continuar.

**Pregunta 1 — Motor de base de datos:**
> "¿Qué motor de base de datos estás usando? (PostgreSQL / MySQL / MariaDB /
> SQL Server / SQLite)"

Esta respuesta determina qué tipos de dato, sintaxis DDL y restricciones nativas
están disponibles. Ver sección "Diferencias por Motor" más abajo.

**Pregunta 2 — Tipo de carga:**
> "¿Este sistema prioriza escrituras rápidas y consistencia transaccional estricta
> (OLTP: e-commerce, ERP, SaaS) o está diseñado para lectura y análisis masivo
> de datos históricos (OLAP: Data Warehouse, dashboards, reportes)?"

**Pregunta 3 — Tolerancia a anomalías:**
> "¿Cuál es la tolerancia a datos inconsistentes o huérfanos en este módulo?
> (Crítico: finanzas, salud, legal — Tolerable: logs de actividad, métricas web)"

**Preguntas complementarias (Arquitectura Empresarial):**

- ¿El sistema es multitenant? ¿Cómo se aíslan los datos entre tenants?
- ¿Qué datos son sensibles (PII, finanzas, salud, credenciales, legales)?
- ¿Se requiere conocer valores históricos de ciertos atributos?
- ¿Se requiere trazabilidad completa de cambios (auditoría)?
- ¿Los registros pueden eliminarse físicamente o se requiere soft delete?
- ¿El sistema es distribuido, monolítico, offline-first o de integraciones?
- ¿Cuál es el volumen esperado de datos (filas por tabla, GB por mes)?
- ¿Existe producción con datos? ¿Hay ventana de mantenimiento?

Si el usuario no responde después de ser preguntado, asume: **OLTP + Crítico +
sin multitenancy + sin soft delete + sin auditoría + PK BIGINT**. Documenta
cada supuesto al inicio de tu respuesta.

---

## FASE 2 — Árbol de Decisión Arquitectónico

Sigue estrictamente este flujo según las respuestas de la Fase 1:

```
¿Es OLTP?
├── SÍ + Tolerancia Crítica
│     → Normalizar hasta BCNF. Prohibir cualquier redundancia.
│       Añadir restricciones CHECK, NOT NULL y UNIQUE donde corresponda.
│       Implementar auditoría + soft delete + historización si aplica.
│
├── SÍ + Tolerancia Tolerable
│     → Normalizar hasta 3FN. Evaluar BCNF solo si no degrada
│       transacciones frecuentes. Documentar cualquier excepción.
│
└── NO (OLAP / Analytics)
      → Abortar flujo de normalización tradicional.
        Aplicar Desnormalización Controlada:
        - Convertir a Esquema en Estrella (Tablas de Hechos + Dimensiones).
        - Mantener 3FN solo en dimensiones que se actualicen frecuentemente
          (Slowly Changing Dimensions Tipo 2).
        - Documentar cada desnormalización con su justificación explícita.
```

**¿Hay relaciones multivaluadas completamente independientes entre sí?**
```
SÍ → Evaluar 4FN (ver sección de Formas Superiores).
NO → 3FN/BCNF es suficiente en la mayoría de casos de negocio.
```

---

## FASE 3 — Protocolos de Ejecución por Forma Normal (OLTP)

Para cada tabla del esquema, aplica este protocolo secuencial. Muestra siempre
el estado **antes** y **después** de cada transformación con DDL real en el
motor indicado.

### Paso 0 — Estado Inicial (0FN)

Identifica y clasifica todos los problemas del esquema antes de tocar nada.
Produce una lista anotada de violaciones por tipo:

- `[MULTIVALUADO]` — celda con lista de valores
- `[GRUPO_REPETIDO]` — columnas numeradas (tel1, tel2, tel3)
- `[REDUNDANCIA]` — mismo dato en múltiples filas
- `[MEZCLA_ENTIDADES]` — dos entidades del dominio en una tabla
- `[DERIVADO]` — columna calculable desde otras columnas de la misma tabla
- `[ANTIPATRÓN]` — cualquiera de los 5 detectados en Fase 0
- `[CATALOGO_FALTANTE]` — dominio discreto sin tabla de catálogo
- `[SIN_HISTORIAL]` — atributo cambiante sin soporte de SCD
- `[SIN_AUDITORIA]` — tabla crítica sin columnas de auditoría
- `[SIN_SOFT_DELETE]` — tabla transaccional sin retención lógica

No avances a 1FN hasta tener esta lista completa.

---

### Paso 1 — Primera Forma Normal (1FN): Atomicidad

**Regla:** cada celda contiene exactamente un valor atómico. Sin grupos repetidos.

**Se viola si:**
- `telefonos = "555-1234, 555-5678"` → valor no atómico
- columnas `materia_1`, `materia_2`, `materia_3` → grupo repetido implícito
- arrays o JSON dentro de una columna usados como sustituto de tabla

**Protocolo:**
1. Identificar el atributo multivaluado o grupo repetido.
2. Crear una tabla hija con la PK de la tabla original como parte de su PK compuesta.
3. Cada valor que antes estaba en la lista ocupa ahora una fila independiente.

**Está completo cuando:** puedes describir el valor de cualquier celda con
una frase que no contenga "y", "o" ni enumeraciones.

**Ejemplo:**
```sql
-- ANTES (viola 1FN)
CREATE TABLE estudiantes (
    id       INT PRIMARY KEY,
    nombre   VARCHAR(100),
    materias VARCHAR(200)  -- "Matemáticas, Física, Historia"
);

-- DESPUÉS (cumple 1FN)
CREATE TABLE estudiantes (
    id     INT PRIMARY KEY,
    nombre VARCHAR(100)
);

CREATE TABLE estudiante_materia (
    estudiante_id INT REFERENCES estudiantes(id),
    materia       VARCHAR(100),
    PRIMARY KEY (estudiante_id, materia)
);
```

---

### Paso 2 — Segunda Forma Normal (2FN): Sin Dependencias Parciales

**Solo aplica cuando la PK es compuesta.** Si la PK es de un solo atributo,
documenta "2FN: N/A — PK simple" y pasa al Paso 3.

**Regla:** cada atributo no-clave depende de la PK **completa**, no de
una fracción de ella.

**Prueba de detección:** por cada atributo no-clave, pregunta:
> "¿Este valor cambiaría si solo cambiara UNA de las columnas que forman la PK?"
> SÍ → dependencia parcial → violación.

**Protocolo:**
1. Identificar qué subconjunto de la PK compuesta determina al atributo.
2. Crear una tabla nueva con ese subconjunto como PK.
3. Mover el atributo a esa nueva tabla.
4. Dejar la FK en la tabla original apuntando a la nueva.

**Ejemplo:**
```sql
-- ANTES (viola 2FN)
-- PK compuesta: (empleado_id, proyecto_id)
-- nombre_empleado depende SOLO de empleado_id → violación
CREATE TABLE trabaja_en (
    empleado_id     INT,
    proyecto_id     INT,
    horas           DECIMAL(5,2),
    nombre_empleado VARCHAR(100),  -- depende parcialmente de la PK
    PRIMARY KEY (empleado_id, proyecto_id)
);

-- DESPUÉS (cumple 2FN)
CREATE TABLE empleados (
    empleado_id     INT PRIMARY KEY,
    nombre_empleado VARCHAR(100)
);

CREATE TABLE trabaja_en (
    empleado_id INT REFERENCES empleados(empleado_id),
    proyecto_id INT,
    horas       DECIMAL(5,2),
    PRIMARY KEY (empleado_id, proyecto_id)
);
```

---

### Paso 3 — Tercera Forma Normal (3FN): Sin Dependencias Transitivas

**Regla:** ningún atributo no-clave depende de otro atributo no-clave.
La cadena de dependencia debe terminar siempre en la PK.

**Prueba de detección:** por cada atributo no-clave, pregunta:
> "¿Este valor podría inferirse conociendo OTRO atributo no-clave,
> sin necesidad de conocer la PK?"
> SÍ → dependencia transitiva → violación.

**Protocolo:**
1. Identificar la cadena: PK → A → B (B depende de A, no de la PK).
2. Crear tabla nueva con A como PK y B como atributo.
3. En la tabla original, dejar A como FK y eliminar B.

**Ejemplo:**
```sql
-- ANTES (viola 3FN)
-- codigo_depto → nombre_depto (transitiva: empleado→depto→nombre_depto)
CREATE TABLE empleados (
    empleado_id  INT PRIMARY KEY,
    nombre       VARCHAR(100),
    codigo_depto CHAR(4),
    nombre_depto VARCHAR(100),  -- depende de codigo_depto, no del empleado
    piso_depto   INT            -- ídem
);

-- DESPUÉS (cumple 3FN)
CREATE TABLE departamentos (
    codigo_depto CHAR(4) PRIMARY KEY,
    nombre_depto VARCHAR(100),
    piso         INT
);

CREATE TABLE empleados (
    empleado_id  INT PRIMARY KEY,
    nombre       VARCHAR(100),
    codigo_depto CHAR(4) REFERENCES departamentos(codigo_depto)
);
```

---

### Paso 4 — BCNF (Forma Normal de Boyce-Codd): Para Tolerancia Crítica

Aplica solo si la Fase 2 indicó OLTP + Tolerancia Crítica, o si hay
claves candidatas múltiples que se solapan.

**Regla:** para toda dependencia funcional X → Y, X debe ser una superclave.
BCNF es más estricta que 3FN: elimina anomalías que 3FN deja pasar cuando
hay múltiples claves candidatas solapadas.

**Cuándo 3FN no es suficiente y se necesita BCNF:**
- Tabla con dos o más claves candidatas que comparten al menos un atributo.
- Una clave candidata determina a un atributo que a su vez pertenece
  a otra clave candidata.

**Advertencia de BCNF:** la descomposición a BCNF puede no preservar todas
las dependencias funcionales originales. Si esto ocurre, documenta la pérdida
de dependencia y evalúa si es aceptable para el negocio o si se debe mantener
en 3FN con una restricción explícita compensatoria.

---

## FASE 4 — Formas Superiores (Solo si el Contexto lo Exige)

Son raras en sistemas de negocio típicos. Evalúalas solo cuando haya señales
explícitas.

**4FN — Dependencias Multivaluadas:**
Usa cuando una entidad tiene dos o más conjuntos de valores completamente
independientes entre sí. Sin 4FN, cada combinación genera filas falsas
(producto cartesiano implícito).

Señal de alerta: tabla de tres columnas donde los valores de la segunda
y la tercera no guardan relación entre sí, solo con la primera.

```
-- Ejemplo clásico: profesor puede enseñar varios cursos Y usa varios libros,
-- pero la relación curso-libro no existe.
-- Sin 4FN: cada libro se replica para cada curso del mismo profesor.
PROFESOR_CURSO_LIBRO → violar 4FN
Descomponer en: PROFESOR_CURSO + PROFESOR_LIBRO
```

**5FN (Proyección-Unión):**
Aplica en dominios donde la información solo puede reconstruirse correctamente
juntando tres o más proyecciones. Prácticamente exclusivo de sistemas de
altísima criticidad (logística farmacéutica, inventario multidimensional,
sistemas de defensa). En la mayoría de los proyectos de negocio, nunca
llegarás aquí.

---

## FASE 5 — Diferencias por Motor: DDL y Restricciones Nativas

Una vez definido el esquema normalizado, adapta el DDL al motor declarado
en la Fase 1. Esta sección es obligatoria: nunca entregues DDL genérico
sin aplicar las convenciones del motor objetivo.

### PostgreSQL
```sql
-- PK surrogada recomendada: BIGINT GENERATED ALWAYS AS IDENTITY
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY

-- UUID como alternativa (disponible nativo)
id UUID DEFAULT gen_random_uuid() PRIMARY KEY

-- Tipos relevantes no disponibles en otros motores:
-- JSONB para datos semiestructurados ocasionales (no como sustituto de tabla)
-- ARRAY[] solo si la 1FN está justificadamente sacrificada por diseño consciente
-- NUMERIC(precision, scale) para valores financieros exactos
-- TIMESTAMPTZ para fechas con zona horaria

-- Restricción CHECK nativa:
ALTER TABLE empleados ADD CONSTRAINT chk_salario CHECK (salario > 0);

-- Índice parcial (muy útil para optimizar sin desnormalizar):
CREATE INDEX idx_pedidos_activos ON pedidos(cliente_id) WHERE estado = 'activo';

-- FK con acción en cascada:
REFERENCES tabla_padre(id) ON DELETE CASCADE ON UPDATE CASCADE

-- Multitenant con RLS:
ALTER TABLE pedidos ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON pedidos
    USING (tenant_id = current_setting('app.tenant_id')::INT);
```

**Peculiaridades PostgreSQL:** soporta transacciones DDL completas (puedes
hacer ROLLBACK de un CREATE TABLE). Usa esquemas (namespaces) para separar
módulos: `CREATE TABLE ventas.pedidos (...)`. Soporta `pgcrypto` para
encriptación, `pg_trgm` para búsqueda difusa, y `hstore` para key-value.

---

### MySQL / MariaDB
```sql
-- PK surrogada recomendada:
id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY
-- o para tablas grandes:
id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY

-- Motor de almacenamiento: SIEMPRE InnoDB para soporte de FK y transacciones.
-- MyISAM no soporta FK — nunca lo uses en esquemas normalizados.
CREATE TABLE empleados (...) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- utf8mb4 es obligatorio (utf8 en MySQL es solo 3 bytes, no soporta emojis
-- ni algunos caracteres CJK — bug histórico del motor).

-- Tipos relevantes:
-- DECIMAL(10,2) para valores financieros (no FLOAT ni DOUBLE)
-- DATETIME vs TIMESTAMP: TIMESTAMP se convierte a UTC automáticamente
--   y tiene rango hasta 2038 (problema Y2K38). Usa DATETIME para rangos largos.
-- TEXT y BLOB no pueden tener valor DEFAULT en versiones anteriores a 8.0.

-- FK en MySQL/MariaDB requiere índice en la columna referenciada:
ALTER TABLE pedidos
    ADD CONSTRAINT fk_cliente
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
    ON DELETE RESTRICT ON UPDATE CASCADE;

-- MariaDB tiene CHECK constraints funcionales desde 10.2.1.
-- MySQL las soporta desde 8.0.16 (versiones anteriores las parsean pero ignoran).
-- Verifica la versión antes de usar CHECK.
```

**Diferencias MySQL vs MariaDB:** MariaDB tiene secuencias nativas
(`CREATE SEQUENCE`), temporal tables con más opciones, y algunas funciones
de ventana disponibles desde versiones anteriores. Si el usuario no especifica
cuál de los dos, pregunta — el DDL es mayormente compatible pero hay casos borde.

---

### SQL Server (T-SQL)
```sql
-- PK surrogada recomendada:
id INT IDENTITY(1,1) PRIMARY KEY
-- o para tablas grandes:
id BIGINT IDENTITY(1,1) PRIMARY KEY

-- UUID:
id UNIQUEIDENTIFIER DEFAULT NEWSEQUENTIALID() PRIMARY KEY
-- NEWSEQUENTIALID() es preferible a NEWID() para columnas indexadas
-- (NEWID() genera valores aleatorios que fragmentan el índice clustered).

-- Tipos relevantes:
-- NVARCHAR(n) para texto Unicode (N prefix en literales: N'texto')
-- VARCHAR sin N solo soporta la collation del servidor — riesgo en sistemas multilingüe
-- DECIMAL o NUMERIC para valores financieros (no FLOAT, no REAL)
-- DATETIME2 en lugar de DATETIME (mayor precisión y rango)
-- DATETIMEOFFSET para fechas con zona horaria

-- Restricción CHECK:
ALTER TABLE empleados
    ADD CONSTRAINT chk_salario CHECK (salario > 0);

-- FK:
ALTER TABLE pedidos
    ADD CONSTRAINT fk_cliente
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
    ON DELETE NO ACTION ON UPDATE CASCADE;

-- Esquemas para separar módulos (equivalente a PostgreSQL schemas):
CREATE TABLE ventas.pedidos (...);
CREATE TABLE rrhh.empleados (...);

-- Índice con columnas incluidas (evita lookups sin desnormalizar):
CREATE INDEX idx_pedidos_cliente
    ON pedidos(cliente_id)
    INCLUDE (fecha, total);

-- Data Masking:
ALTER TABLE clientes
    ALTER COLUMN email ADD MASKED WITH (FUNCTION = 'email()');
```

**Peculiaridades SQL Server:** las transacciones DDL NO son automáticamente
reversibles como en PostgreSQL — envuelve migraciones en BEGIN TRAN / ROLLBACK.
El índice clustered por defecto va sobre la PK; si la PK es un UNIQUEIDENTIFIER
con NEWID(), fragmentará el índice — usa NEWSEQUENTIALID() o una PK INT por
separado con el UUID como campo UNIQUE.

---

### SQLite
```sql
-- PK surrogada: INTEGER PRIMARY KEY es el alias de rowid (autoincremental nativo)
-- BIGINT, INT, etc. con PRIMARY KEY también funcionan pero INTEGER es el canónico.
id INTEGER PRIMARY KEY

-- SQLite tiene tipado dinámico ("type affinity"), no tipado estricto.
-- Las columnas aceptan cualquier tipo de dato independientemente de su declaración.
-- Esto significa que las restricciones de tipo son sugerencias, no garantías.
-- Usar STRICT tables (SQLite 3.37+) para comportamiento estricto:
CREATE TABLE empleados (
    id     INTEGER PRIMARY KEY,
    nombre TEXT    NOT NULL,
    salario REAL   NOT NULL
) STRICT;

-- FK están DESHABILITADAS por defecto en SQLite.
-- Deben activarse por conexión:
PRAGMA foreign_keys = ON;
-- Documenta esto siempre en el esquema entregado — es el error más común en SQLite.

-- Tipos disponibles (affinity):
-- TEXT, INTEGER, REAL, BLOB, NUMERIC
-- No existe BOOLEAN nativo: usar INTEGER (0/1) con CHECK constraint.
-- No existe DATE/DATETIME nativo: usar TEXT (ISO 8601), REAL (Julian Day)
--   o INTEGER (Unix timestamp). Elige uno y sé consistente en todo el esquema.

-- CHECK constraints soportadas:
CREATE TABLE empleados (
    id     INTEGER PRIMARY KEY,
    salario REAL CHECK(salario > 0)
);

-- Limitaciones críticas a comunicar siempre:
-- ❌ No soporta ALTER TABLE para eliminar o modificar columnas (solo agregar)
--    Workaround: recrear la tabla con la estructura correcta y copiar datos.
-- ❌ No soporta RIGHT JOIN ni FULL OUTER JOIN.
-- ❌ No soporta stored procedures ni triggers complejos.
-- ❌ No soporta particionamiento nativo (usar múltiples DBs).
-- ✅ Ideal para: apps móviles, prototipos, sistemas embebidos, archivos locales.
-- ❌ No recomendado para: sistemas multiusuario concurrentes con alta escritura.
```

---

## FASE 6 — Contrato de Output (Entrega Obligatoria)

Tu respuesta final debe contener estas **nueve secciones** en este orden exacto.

### [DIAGNÓSTICO ARQUITECTÓNICO]

Lista de vulnerabilidades detectadas en el esquema original. Incluye tipo
de violación, tabla afectada y columna(s) involucrada(s). Combinar problemas
de normalización con problemas de arquitectura empresarial:

- Anomalías de 0FN/1FN/2FN/3FN/BCNF
- Anti-patrones (Fase 0)
- Catálogos faltantes
- Historización faltante
- Auditoría faltante
- Soft delete faltante
- Aislamiento de tenant comprometido
- Datos sensibles sin protección

Máximo 15 líneas, formato tabular o de lista priorizada.

### [ÁRBOL DE DEPENDENCIAS FUNCIONALES]

Para cada tabla del esquema final, lista las dependencias funcionales
identificadas en formato: `atributo_determinante → {atributos_determinados}`.
Esto justifica matemáticamente cada decisión de descomposición.

### [ESQUEMA DDL FINAL]

Código DDL completo en el motor declarado. Incluir:

- `CREATE TABLE` con todos los tipos de dato correctos para el motor
- Categoría de cada tabla: `[MAESTRO]`, `[TRANSACCIONAL]`, `[CATÁLOGO]`,
  `[INTERSECCIÓN]`, `[HISTÓRICO]`, `[AUDITORÍA]`
- Restricciones `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, `UNIQUE`, `CHECK`
- Columnas de auditoría (`created_at`, `created_by`, `updated_at`, `updated_by`)
- Columnas de soft delete (`deleted_at` o `activo` + `fecha_eliminacion`)
- Columna `tenant_id` si aplica
- Comentarios inline en columnas con lógica no obvia
- `CREATE INDEX` para las FK, queries frecuentes y reportes
- Triggers de validación de tenant si aplica
- Políticas RLS si aplica

```sql
-- Ejemplo de formato esperado:
CREATE TABLE departamentos (
    codigo_depto CHAR(4)     PRIMARY KEY,
    nombre       VARCHAR(100) NOT NULL UNIQUE,
    piso         SMALLINT    CHECK(piso BETWEEN 1 AND 50)
);

CREATE TABLE empleados (
    empleado_id  INT          PRIMARY KEY,
    nombre       VARCHAR(100) NOT NULL,
    codigo_depto CHAR(4)      NOT NULL REFERENCES departamentos(codigo_depto)
                              ON DELETE RESTRICT ON UPDATE CASCADE,
    -- auditoría
    created_at   TIMESTAMPTZ  NOT NULL DEFAULT now(),
    updated_at   TIMESTAMPTZ
);

CREATE INDEX idx_empleados_depto ON empleados(codigo_depto);
```

### [DIAGRAMA DE RELACIONES]

Relaciones entre tablas con cardinalidad y naturaleza del vínculo:

```
departamentos (1) ────── (N) empleados
empleados     (1) ────── (N) trabaja_en (N) ────── (1) proyectos
clientes      (1) ────── (N) pedidos (1) ────── (N) lineas_pedido (N) ────── (1) productos
```

### [ANÁLISIS DE ESCALABILIDAD]

- Crecimiento esperado (filas/mes, GB/mes por tabla)
- Necesidad de particionamiento (sí/no, tipo, criterio de partición)
- Riesgos de bloqueo (lock contention, long-running transactions)
- Estrategia de archivo / cold storage

### [ANÁLISIS DE SEGURIDAD]

- Datos sensibles detectados (clasificados por categoría: PII, finanzas, salud, etc.)
- Recomendaciones aplicadas (encriptación, RLS, masking, hashing)
- Normativas que aplican (GDPR, LOPD, HIPAA, PCI-DSS, etc.) y su cumplimiento

### [PLAN DE MIGRACIÓN]

Solo si hay producción con datos. Incluir:

- Pasos exactos de despliegue con orden de ejecución
- Tiempo estimado de cada paso
- Riesgos por paso
- Scripts de rollback por paso
- Criterios de Go/No-Go entre pasos
- Patrón expand-contract: crear nuevo → backfill → validar → dual write
  → switch → cleanup

### [ESTRATEGIA DE AUDITORÍA]

- Columnas de auditoría aplicadas (`created_at`, `updated_at`, `deleted_at`, etc.)
- Tablas de auditoría dedicadas (con triggers o políticas)
- Inmutabilidad garantizada (revocación de UPDATE/DELETE)
- Retención del log (¿cuánto tiempo se conservan los registros?)

### [ESTRATEGIA DE HISTORIZACIÓN]

- Entidades que requieren historial
- Técnica SCD aplicada (Tipo 1, 2, 3, 4 — justificar elección)
- Diseño de tablas de historial (columnas de vigencia, constraints)

### [ADVERTENCIAS DE RENDIMIENTO E INTEGRIDAD]

Lista de posibles cuellos de botella y recomendaciones específicas para
el motor declarado. Ejemplos de formato:

- "La tabla `pedidos` requerirá JOIN constante con `clientes`. Crear índice
  compuesto en `(cliente_id, fecha)` para cubrir las queries de historial."
- "En MySQL/MariaDB: verificar que el motor sea InnoDB antes del deploy —
  sin InnoDB las FK no se aplican y el esquema queda sin integridad referencial."
- "En SQLite: agregar `PRAGMA foreign_keys = ON` al inicio de cada conexión
  o las FK del esquema serán ignoradas silenciosamente."
- "Índice parcial en `WHERE deleted_at IS NULL` para no escanear registros eliminados."
- "Validar con el equipo de aplicación que todas las queries usen `tenant_id`
  como primer criterio de filtro; de lo contrario, RLS se vuelve un cuello de botella."

---

## REGLAS DE NOMENCLATURA

Aplicar consistentemente en todo el esquema entregado.

### Tablas

`snake_case` en plural: `clientes`, `pedidos`, `lineas_pedido`

### Columnas

`snake_case` en singular: `nombre`, `fecha_nacimiento`, `cliente_id`

### Primary Keys

`id` (genérico) o `<tabla_singular>_id` si la tabla tiene FKs salientes
que generen ambigüedad.

### Foreign Keys

`<tabla_referenciada_singular>_id`: `cliente_id`, `producto_id`, `pedido_id`

### Índices

`idx_<tabla>_<columna(s)>`: `idx_pedidos_cliente`, `idx_pedidos_cliente_fecha`

### Unique Constraints

`uq_<tabla>_<columna(s)>`: `uq_clientes_email`, `uq_empleado_cargo_actual`

### Foreign Key Constraints

`fk_<tabla>_<tabla_referenciada>`: `fk_pedidos_clientes`,
`fk_lineas_pedido`

### Check Constraints

`chk_<tabla>_<regla>`: `chk_empleados_salario_positivo`,
`chk_pedidos_fecha_fin_posterior`

### Catálogos

`catalogo_<dominio>`: `catalogo_estado_pedido`, `catalogo_tipo_documento`

### Historial

`<entidad>_<atributo>_historial`: `empleado_salario_historial`

### Auditoría

`auditoria_<entidad>`: `auditoria_pedido`

---

## Anti-Patrones de PKs — Tabla de Referencia por Motor

| Situación | PostgreSQL | MySQL/MariaDB | SQL Server | SQLite |
|---|---|---|---|---|
| PK entera simple | `BIGINT GENERATED ALWAYS AS IDENTITY` | `BIGINT UNSIGNED AUTO_INCREMENT` | `BIGINT IDENTITY(1,1)` | `INTEGER PRIMARY KEY` |
| PK UUID | `UUID DEFAULT gen_random_uuid()` | `CHAR(36)` + trigger o app | `UNIQUEIDENTIFIER DEFAULT NEWSEQUENTIALID()` | `TEXT` + app |
| PK texto mutable | ❌ Nunca | ❌ Nunca | ❌ Nunca | ❌ Nunca |
| PK email/nombre | ❌ Nunca | ❌ Nunca | ❌ Nunca | ❌ Nunca |

**Regla universal:** las PKs deben ser surrogadas (generadas por el sistema),
estables (nunca cambian), únicas y sin significado de negocio. Un email
o un código de producto son candidatos a clave natural útil como campo UNIQUE,
no como PK.

---

# 📦 Ejemplos End-to-End Completos

Estos tres ejemplos ilustran cómo se ven las **9 secciones del contrato de
output (Fase 6)** aplicadas a dominios reales con problemáticas distintas.
Están pensados como plantillas que la IA puede adaptar cuando el usuario
describa un sistema similar.

- **Ejemplo 1: E-commerce B2C** — motor PostgreSQL, OLTP crítico, sin
  multitenancy, con historización de precios y auditoría selectiva.
- **Ejemplo 2: SaaS B2B Multi-Tenant** — motor PostgreSQL, OLTP crítico,
  CON multitenancy (foco especial), con catálogos extensos, soft delete,
  auditoría inmutable y RLS.
- **Ejemplo 3: Sistema de Salud (HIS/HCE)** — motor PostgreSQL, OLTP
  crítico, HIPAA, con historización clínica exhaustiva, datos altamente
  sensibles, retención legal obligatoria y particionamiento temporal.

---

## Ejemplo 1 — E-commerce B2C

**Contexto del usuario (mensaje original):**
> "Estoy diseñando una tienda online en PostgreSQL. Manejo clientes, productos
> con categorías, pedidos con líneas, pagos, envíos, descuentos, y necesito
> saber el histórico de precios de cada producto. Algunos clientes son
> recurrentes. Quiero auditar quién modifica pedidos y categorías críticas."

**Supuestos documentados:**
- Motor: PostgreSQL
- Carga: OLTP
- Tolerancia: Crítica (pedidos y pagos)
- PK: BIGINT
- Multitenancy: No
- Soft delete: Solo en pedidos y clientes
- Auditoría: Selectiva (pedidos, pagos, descuentos, cambios de precio)
- Historización: Precios de producto (SCD Tipo 2)
- Particionamiento: No por ahora (< 10M filas esperado año 1)

---

### [DIAGNÓSTICO ARQUITECTÓNICO]

Diseño desde cero — no hay esquema previo que violar. Se identifican
oportunidades de modelado:

- `productos` requiere **tabla de historial de precios** (SCD2) por
  promociones y cambios de tarifa.
- `pedidos.estado` debe ser **catálogo** (`catalogo_estado_pedido`).
- `pedidos` y `pagos` requieren **auditoría de cambios** (montos,
  estados, cancelaciones).
- `clientes` y `pedidos` requieren **soft delete** (recuperación + LGPD).
- `direcciones_envio` se repite por cliente → normalizar en tabla propia
  con historial (cliente se muda).

---

### [ÁRBOL DE DEPENDENCIAS FUNCIONALES]

```
clientes:
  id → {nombre, email, telefono, fecha_registro, activo}

categorias:
  id → {nombre, slug, padre_id, orden}

productos:
  id → {sku, nombre, descripcion, categoria_id, activo}
  sku → {id, nombre, ...}    -- sku es UNIQUE, alternativa natural

producto_precios (historial SCD2):
  (producto_id, fecha_inicio) → {precio, moneda_id, usuario_id}
  producto_id → {precio_actual}   -- derivado, sin columna

pedidos:
  id → {cliente_id, fecha, estado_id, subtotal, descuento, total,
        direccion_envio_id, created_at, ...}
  numero_pedido → {id, ...}      -- UNIQUE, visible al cliente

lineas_pedido:
  (pedido_id, linea) → {producto_id, cantidad, precio_unitario, subtotal}
  producto_id → {precio_unitario_historico}   -- capturado al vender

pagos:
  id → {pedido_id, monto, metodo_pago_id, estado_id, transaccion_externa,
        fecha, created_at}
```

---

### [ESQUEMA DDL FINAL]

```sql
-- ============================================================
-- E-COMMERCE B2C - PostgreSQL - v1.0
-- Generado aplicando: Fases 0, 1, 2, 3, 5, 6 + Catálogos,
-- Historización (SCD2 precios), Auditoría selectiva, Soft delete.
-- ============================================================

-- ============================================================
-- CATÁLOGOS
-- ============================================================

CREATE TABLE catalogo_moneda (                           -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo CHAR(3) NOT NULL UNIQUE,                       -- ISO 4217: USD, MXN, EUR
    nombre VARCHAR(50) NOT NULL
);

CREATE TABLE catalogo_estado_pedido (                    -- [CATÁLOGO]
    id        SMALLINT PRIMARY KEY,
    codigo    VARCHAR(20) NOT NULL UNIQUE,                -- PENDIENTE, PAGADO, ENVIADO, ...
    nombre    VARCHAR(100) NOT NULL,
    es_final  BOOLEAN NOT NULL DEFAULT FALSE,             -- TRUE = terminal (no transiciona)
    orden     SMALLINT NOT NULL
);

CREATE TABLE catalogo_estado_pago (                      -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(20) NOT NULL UNIQUE,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE catalogo_metodo_pago (                      -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(30) NOT NULL UNIQUE,                   -- TARJETA_CREDITO, PAYPAL, TRANSFERENCIA
    nombre VARCHAR(100) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE TABLE catalogo_estado_envio (                     -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(20) NOT NULL UNIQUE,
    nombre VARCHAR(100) NOT NULL
);

-- ============================================================
-- MAESTROS
-- ============================================================

CREATE TABLE clientes (                                  -- [MAESTRO]
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre        VARCHAR(150) NOT NULL,
    email         VARCHAR(200) NOT NULL,
    telefono      VARCHAR(30),
    fecha_registro TIMESTAMPTZ NOT NULL DEFAULT now(),
    -- soft delete
    activo        BOOLEAN NOT NULL DEFAULT TRUE,
    fecha_eliminacion TIMESTAMPTZ,
    -- auditoría
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at    TIMESTAMPTZ,
    CONSTRAINT uq_clientes_email UNIQUE (email),
    CONSTRAINT chk_clientes_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

CREATE INDEX idx_clientes_activos ON clientes(id) WHERE activo = TRUE;

CREATE TABLE direcciones_envio (                        -- [MAESTRO]
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    cliente_id      BIGINT NOT NULL REFERENCES clientes(id) ON DELETE RESTRICT,
    destinatario    VARCHAR(150) NOT NULL,
    linea1          VARCHAR(200) NOT NULL,
    linea2          VARCHAR(200),
    ciudad          VARCHAR(100) NOT NULL,
    region          VARCHAR(100) NOT NULL,
    codigo_postal   VARCHAR(20)  NOT NULL,
    pais            CHAR(2)      NOT NULL,                -- ISO 3166-1 alpha-2
    es_principal    BOOLEAN      NOT NULL DEFAULT FALSE,
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT now(),
    CONSTRAINT chk_direcciones_pais CHECK (char_length(pais) = 2)
);

CREATE INDEX idx_direcciones_cliente ON direcciones_envio(cliente_id);

-- Solo una dirección principal por cliente
CREATE UNIQUE INDEX uq_direccion_principal_por_cliente
    ON direcciones_envio(cliente_id) WHERE es_principal = TRUE;

CREATE TABLE categorias (                                -- [MAESTRO] (auto-referencial)
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre     VARCHAR(100) NOT NULL,
    slug       VARCHAR(100) NOT NULL UNIQUE,
    padre_id   BIGINT REFERENCES categorias(id) ON DELETE RESTRICT,
    orden      SMALLINT NOT NULL DEFAULT 0,
    activo     BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_categorias_padre ON categorias(padre_id);

CREATE TABLE productos (                                 -- [MAESTRO]
    id           BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sku          VARCHAR(50) NOT NULL UNIQUE,
    nombre       VARCHAR(200) NOT NULL,
    descripcion  TEXT,
    categoria_id BIGINT NOT NULL REFERENCES categorias(id) ON DELETE RESTRICT,
    stock        INTEGER NOT NULL DEFAULT 0,
    activo       BOOLEAN NOT NULL DEFAULT TRUE,
    -- auditoría
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at   TIMESTAMPTZ,
    CONSTRAINT chk_productos_stock CHECK (stock >= 0)
);

CREATE INDEX idx_productos_categoria ON productos(categoria_id);
CREATE INDEX idx_productos_activos ON productos(id) WHERE activo = TRUE;

-- ============================================================
-- HISTORIAL DE PRECIOS (SCD Tipo 2)
-- ============================================================

CREATE TABLE producto_precios (                          -- [HISTÓRICO]
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    producto_id   BIGINT NOT NULL REFERENCES productos(id) ON DELETE CASCADE,
    precio        NUMERIC(12,2) NOT NULL,
    moneda_id     SMALLINT NOT NULL REFERENCES catalogo_moneda(id),
    fecha_inicio  TIMESTAMPTZ NOT NULL DEFAULT now(),
    fecha_fin     TIMESTAMPTZ,                            -- NULL = vigente
    usuario_id    BIGINT NOT NULL,
    CONSTRAINT chk_precios_valor CHECK (precio > 0),
    CONSTRAINT chk_precios_vigencia CHECK (fecha_fin IS NULL OR fecha_fin > fecha_inicio)
);

CREATE INDEX idx_precios_producto_vigente
    ON producto_precios(producto_id) WHERE fecha_fin IS NULL;

-- Un solo precio vigente por producto (garantizado por índice único parcial)
CREATE UNIQUE INDEX uq_precios_un_vigente_por_producto
    ON producto_precios(producto_id) WHERE fecha_fin IS NULL;

-- ============================================================
-- TRANSACCIONAL
-- ============================================================

CREATE TABLE pedidos (                                   -- [TRANSACCIONAL]
    id                 BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    numero_pedido      VARCHAR(20) NOT NULL UNIQUE,        -- visible al cliente
    cliente_id         BIGINT NOT NULL REFERENCES clientes(id) ON DELETE RESTRICT,
    estado_id          SMALLINT NOT NULL REFERENCES catalogo_estado_pedido(id),
    direccion_envio_id BIGINT NOT NULL REFERENCES direcciones_envio(id),
    subtotal           NUMERIC(12,2) NOT NULL,
    descuento          NUMERIC(12,2) NOT NULL DEFAULT 0,
    total              NUMERIC(12,2) NOT NULL,
    moneda_id          SMALLINT NOT NULL REFERENCES catalogo_moneda(id),
    -- soft delete
    activo             BOOLEAN NOT NULL DEFAULT TRUE,
    fecha_eliminacion  TIMESTAMPTZ,
    -- auditoría
    created_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by         BIGINT NOT NULL,
    updated_at         TIMESTAMPTZ,
    updated_by         BIGINT,
    CONSTRAINT chk_pedidos_subtotal_positivo CHECK (subtotal >= 0),
    CONSTRAINT chk_pedidos_total_coherente CHECK (total = subtotal - descuento),
    CONSTRAINT chk_pedidos_descuento_no_negativo CHECK (descuento >= 0),
    CONSTRAINT chk_pedidos_descuento_no_mayor_subtotal CHECK (descuento <= subtotal)
);

CREATE INDEX idx_pedidos_cliente ON pedidos(cliente_id);
CREATE INDEX idx_pedidos_estado ON pedidos(estado_id);
CREATE INDEX idx_pedidos_fecha ON pedidos(created_at DESC);
CREATE INDEX idx_pedidos_activos ON pedidos(id) WHERE activo = TRUE;
CREATE INDEX idx_pedidos_reporte
    ON pedidos(cliente_id, created_at DESC) INCLUDE (total, estado_id);

CREATE TABLE linea_pedido (                              -- [TRANSACCIONAL]
    id               BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    pedido_id        BIGINT NOT NULL REFERENCES pedidos(id) ON DELETE CASCADE,
    linea            SMALLINT NOT NULL,                    -- número de línea (1, 2, 3...)
    producto_id      BIGINT NOT NULL REFERENCES productos(id) ON DELETE RESTRICT,
    cantidad         INTEGER NOT NULL,
    precio_unitario  NUMERIC(12,2) NOT NULL,               -- SNAPSHOT al momento de la venta
    subtotal         NUMERIC(12,2) NOT NULL,
    CONSTRAINT uq_linea_pedido UNIQUE (pedido_id, linea),
    CONSTRAINT chk_linea_cantidad CHECK (cantidad > 0),
    CONSTRAINT chk_linea_precio CHECK (precio_unitario > 0),
    CONSTRAINT chk_linea_subtotal CHECK (subtotal = cantidad * precio_unitario)
);

CREATE INDEX idx_lineas_pedido ON linea_pedido(pedido_id);
CREATE INDEX idx_lineas_producto ON linea_pedido(producto_id);

CREATE TABLE pagos (                                     -- [TRANSACCIONAL]
    id                  BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    pedido_id           BIGINT NOT NULL REFERENCES pedidos(id) ON DELETE RESTRICT,
    monto               NUMERIC(12,2) NOT NULL,
    moneda_id           SMALLINT NOT NULL REFERENCES catalogo_moneda(id),
    metodo_pago_id      SMALLINT NOT NULL REFERENCES catalogo_metodo_pago(id),
    estado_id           SMALLINT NOT NULL REFERENCES catalogo_estado_pago(id),
    transaccion_externa VARCHAR(100),                      -- ID del gateway (Stripe, PayPal, etc.)
    fecha_pago          TIMESTAMPTZ NOT NULL DEFAULT now(),
    -- auditoría
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by          BIGINT NOT NULL,
    CONSTRAINT chk_pagos_monto_positivo CHECK (monto > 0)
);

CREATE INDEX idx_pagos_pedido ON pagos(pedido_id);
CREATE INDEX idx_pagos_estado ON pagos(estado_id);
CREATE UNIQUE INDEX uq_pagos_transaccion_externa
    ON pagos(transaccion_externa) WHERE transaccion_externa IS NOT NULL;

CREATE TABLE envios (                                    -- [TRANSACCIONAL]
    id                BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    pedido_id         BIGINT NOT NULL UNIQUE REFERENCES pedidos(id) ON DELETE CASCADE,
    estado_id         SMALLINT NOT NULL REFERENCES catalogo_estado_envio(id),
    numero_guia       VARCHAR(100),
    transportista     VARCHAR(100),
    fecha_despacho    TIMESTAMPTZ,
    fecha_entrega     TIMESTAMPTZ,
    created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT chk_envios_fechas CHECK (fecha_entrega IS NULL OR fecha_entrega >= fecha_despacho)
);

-- ============================================================
-- AUDITORÍA SELECTIVA (solo entidades críticas)
-- ============================================================

CREATE TABLE auditoria_pedido (                          -- [AUDITORÍA]
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    pedido_id       BIGINT NOT NULL,
    operacion       VARCHAR(10) NOT NULL CHECK (operacion IN ('INSERT','UPDATE','DELETE')),
    datos_antes     JSONB,
    datos_despues   JSONB,
    campos_cambiados TEXT[],                              -- ['estado_id', 'total']
    usuario_id      BIGINT NOT NULL,
    ip_origen       INET,
    timestamp_audit TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_auditoria_pedido ON auditoria_pedido(pedido_id);
CREATE INDEX idx_auditoria_pedido_fecha ON auditoria_pedido(timestamp_audit DESC);

-- Inmutabilidad: solo INSERT permitido
REVOKE UPDATE, DELETE ON auditoria_pedido FROM app_user;

-- Trigger de auditoría en pedidos
CREATE FUNCTION fn_auditar_pedido() RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO auditoria_pedido (pedido_id, operacion, datos_despues, usuario_id)
        VALUES (NEW.id, 'INSERT', to_jsonb(NEW), NEW.created_by);
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO auditoria_pedido (pedido_id, operacion, datos_antes, datos_despues, campos_cambiados, usuario_id)
        VALUES (NEW.id, 'UPDATE', to_jsonb(OLD), to_jsonb(NEW),
                (SELECT array_agg(key) FROM jsonb_each(to_jsonb(NEW)) WHERE to_jsonb(NEW)->>key IS DISTINCT FROM to_jsonb(OLD)->>key),
                NEW.updated_by);
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO auditoria_pedido (pedido_id, operacion, datos_antes, usuario_id)
        VALUES (OLD.id, 'DELETE', to_jsonb(OLD), current_user_id());
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_auditar_pedido
    AFTER INSERT OR UPDATE OR DELETE ON pedidos
    FOR EACH ROW EXECUTE FUNCTION fn_auditar_pedido();
```

---

### [DIAGRAMA DE RELACIONES]

```
clientes (1) ────── (N) pedidos (1) ────── (1) pagos
clientes (1) ────── (N) direcciones_envio
pedidos  (1) ────── (N) linea_pedido (N) ────── (1) productos
pedidos  (1) ────── (0..1) envios
pedidos  (1) ────── (N) auditoria_pedido
productos (1) ────── (N) producto_precios (SCD2) (1) ─── catalogo_moneda
categorias (1) ────── (N) productos
categorias (auto-referencial N) ─── padre_id
```

---

### [ANÁLISIS DE ESCALABILIDAD]

- **Crecimiento esperado año 1:** 500K pedidos, 5M líneas, 100K productos, 50K clientes.
- **Particionamiento:** No necesario en año 1. Evaluar partición por `created_at`
  en `pedidos` y `linea_pedido` cuando superen 10M filas (estimado año 3).
- **Riesgo de bloqueo:** `producto_precios` recibe escrituras frecuentes
  (cambios de tarifa). Considerar cola de cambios nocturna o lock optimista
  en aplicación si el volumen crece.
- **Archivo:** Pedidos > 5 años pueden moverse a esquema `archivo_` con
  compresión de JSONB en `auditoria_pedido`.

---

### [ANÁLISIS DE SEGURIDAD]

| Categoría | Datos detectados | Recomendación |
|---|---|---|
| PII | email, telefono, direccion_envio, destinatario | Cifrado en tránsito (TLS). Email con `pgcrypto` opcional. |
| Financiera | datos de pago, tarjeta, totales | **Nunca almacenar tarjeta**: delegar al gateway. Solo guardar `transaccion_externa` y últimos 4 dígitos (no incluido en este esquema). |
| Credenciales | password | Usar `auth.users` separado; nunca en `clientes`. |
| Auditoría inmutable | cambios en pedidos | `REVOKE UPDATE, DELETE` + trigger. |

- **Cumplimiento:** LGPD/GDPR — derecho al olvido: soft delete + job de
  purga física 30 días después de solicitud de eliminación.
- **Email y teléfono:** enmascarar en logs y dumps (`***@***.com`).

---

### [PLAN DE MIGRACIÓN]

Esquema nuevo en desarrollo → no hay producción. Plan:

1. **Mes 1:** crear catálogos, maestros, tablas transaccionales.
2. **Mes 2:** implementar SCD2 de precios y trigger de auditoría.
3. **Mes 3:** capa de aplicación con escritura dual a `pedidos` legacy
   (si existe) y nuevo.
4. **Mes 4:** backfill histórico de pedidos (último año) desde CSV/legacy.
5. **Mes 5:** switch de lecturas, dejar legacy en solo lectura.
6. **Mes 6:** DROP de legacy.

Cada paso tiene criterios de Go/No-Go (conteos, sumas, sample queries).

---

### [ESTRATEGIA DE AUDITORÍA]

- **Columnas:** `created_at`, `created_by`, `updated_at`, `updated_by`
  en `clientes`, `pedidos`, `pagos`.
- **Tabla dedicada:** `auditoria_pedido` con `datos_antes`/`datos_despues`
  en JSONB y `campos_cambiados` (diff de columnas).
- **Inmutabilidad:** `REVOKE UPDATE, DELETE` + trigger `AFTER INSERT/UPDATE/DELETE`.
- **Retención:** 7 años (normativa fiscal LATAM/EU) en producción, después
  archive a almacenamiento frío.

---

### [ESTRATEGIA DE HISTORIZACIÓN]

- **`producto_precios` — SCD Tipo 2** (justificación: el negocio necesita
  reportes de venta al "precio vigente en ese momento" — el precio de venta
  queda capturado en `linea_pedido.precio_unitario`, pero el histórico del
  catálogo de precios es necesario para análisis de margen y auditoría
  de cambios de tarifa).
- **Direcciones de envío** — historial SCD2 si el negocio requiere
  "dirección al momento de la compra" (no modelado en este ejemplo).

---

### [ADVERTENCIAS DE RENDIMIENTO E INTEGRIDAD]

- **`producto_precios`:** índice único parcial `WHERE fecha_fin IS NULL` —
  cualquier intento de insertar dos precios vigentes para el mismo producto
  falla atómicamente. Crítico para el cálculo de precio en checkout.
- **`linea_pedido.precio_unitario`:** es SNAPSHOT deliberado. Aunque
  `producto_precios` cambie, la línea no debe modificarse. El CHECK
  `subtotal = cantidad * precio_unitario` lo blinda contra manipulación.
- **`pedidos.total`:** se valida con `CHECK (total = subtotal - descuento)`.
  Si el negocio añade impuestos, ajustar la fórmula.
- **PostgreSQL tips:** transacciones de checkout deben usar
  `SERIALIZABLE` o `SELECT ... FOR UPDATE` en `productos.stock`.
- **RLS no necesario:** sistema single-tenant.

---

## Ejemplo 2 — SaaS B2B Multi-Tenant (CRM)

**Contexto del usuario (mensaje original):**
> "Voy a construir un CRM en PostgreSQL como SaaS. Tengo múltiples empresas
> cliente (tenants), cada una con sus propios usuarios, contactos, oportunidades,
> actividades, campañas. NO quiero que una empresa vea los datos de otra ni
> siquiera por bug. Quiero saber histórico de las etapas de cada oportunidad
> y auditar TODA operación de los usuarios."

**Supuestos documentados:**
- Motor: PostgreSQL
- Carga: OLTP
- Tolerancia: Crítica
- PK: BIGINT (monolito con sharding futuro)
- **Multitenancy: SÍ — Pool compartido + RLS**
- **Soft delete: TODAS las tablas de negocio**
- **Auditoría: GLOBAL (toda tabla crítica)**
- Historización: Etapas de oportunidad (SCD2)
- Catálogos: Extensos (industria, etapa, fuente, prioridad, etc.)
- Estrategia de aislamiento: **RLS por `tenant_id`** + validación cruzada

---

### [DIAGNÓSTICO ARQUITECTÓNICO]

Diseño desde cero con foco en aislamiento de tenants:

- **Riesgo crítico #1:** cualquier FK entre tablas de negocio que cruce
  tenants = **violación de aislamiento**. Implementar trigger de validación.
- **Riesgo crítico #2:** queries que olviden filtrar por `tenant_id` =
  fuga de datos entre empresas. Mitigar con RLS + tests de fuga.
- **Riesgo crítico #3:** `INSERT/UPDATE/DELETE` por usuario con sesión
  multi-tenant = riesgo de cambiar de tenant sin querer. Forzar `tenant_id`
  desde la sesión (GUC `app.tenant_id`).
- `oportunidades.etapa` cambia con el tiempo → SCD2.
- `usuarios` es global (un usuario puede pertenecer a varios tenants vía
  `usuario_tenant` con rol) — modelo "shared users, isolated data".
- Catálogos: industria, etapa_oportunidad, fuente_lead, prioridad, etc.
  → **catálogos globales compartidos** entre tenants (configuración del SaaS).

---

### [ÁRBOL DE DEPENDENCIAS FUNCIONALES]

```
tenants:
  id → {nombre, slug, plan, estado_id, fecha_creacion}

usuarios (global, no tiene tenant_id propio):
  id → {email, nombre, password_hash, ...}
  email → {id, ...}                       -- UNIQUE global

usuario_tenant (intersección + rol):
  (usuario_id, tenant_id) → {rol_id, activo, fecha_invitacion}

oportunidades:
  (id, tenant_id) → {nombre, contacto_id, monto, etapa_id, ...}
  -- tenant_id es parte de la PK compuesta para forzar unicidad por tenant

oportunidad_etapa_historial (SCD2):
  (oportunidad_id, fecha_inicio) → {etapa_id, usuario_id, comentario}

contactos:
  (id, tenant_id) → {nombre, email, empresa, ...}
  -- un mismo email puede existir en distintos tenants
```

---

### [ESQUEMA DDL FINAL]

```sql
-- ============================================================
-- SAAS CRM MULTI-TENANT - PostgreSQL - v1.0
-- Estrategia: Pool compartido + RLS + validación cruzada
-- ============================================================

-- ============================================================
-- TENANT ROOT
-- ============================================================

CREATE TABLE tenants (                                   -- [MAESTRO]
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre        VARCHAR(200) NOT NULL,
    slug          VARCHAR(50)  NOT NULL UNIQUE,            -- subdominio: acme.crm.com
    plan          VARCHAR(20)  NOT NULL DEFAULT 'free'
                  CHECK (plan IN ('free', 'pro', 'enterprise')),
    estado_id     SMALLINT     NOT NULL,
    fecha_creacion TIMESTAMPTZ NOT NULL DEFAULT now(),
    fecha_baja    TIMESTAMPTZ,                            -- NULL = activo
    CONSTRAINT chk_tenants_slug CHECK (slug ~* '^[a-z0-9-]+$')
);

CREATE TABLE catalogo_estado_tenant (                    -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(20) NOT NULL UNIQUE,                   -- ACTIVO, SUSPENDIDO, BAJA
    nombre VARCHAR(100) NOT NULL
);

ALTER TABLE tenants
    ADD CONSTRAINT fk_tenants_estado
    FOREIGN KEY (estado_id) REFERENCES catalogo_estado_tenant(id);

-- ============================================================
-- USUARIOS GLOBALES (compartidos entre tenants)
-- ============================================================

CREATE TABLE usuarios (                                  -- [MAESTRO]
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email         VARCHAR(200) NOT NULL UNIQUE,
    nombre        VARCHAR(150) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,                  -- bcrypt/argon2, nunca plano
    activo        BOOLEAN NOT NULL DEFAULT TRUE,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE catalogo_rol (                              -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(30) NOT NULL UNIQUE,                   -- OWNER, ADMIN, VENDEDOR, VIEWER
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE usuario_tenant (                            -- [INTERSECCIÓN]
    usuario_id       BIGINT NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
    tenant_id        BIGINT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    rol_id           SMALLINT NOT NULL REFERENCES catalogo_rol(id),
    fecha_invitacion TIMESTAMPTZ NOT NULL DEFAULT now(),
    invitado_por     BIGINT REFERENCES usuarios(id),
    activo           BOOLEAN NOT NULL DEFAULT TRUE,
    PRIMARY KEY (usuario_id, tenant_id)
);

CREATE INDEX idx_ut_tenant ON usuario_tenant(tenant_id);
CREATE INDEX idx_ut_tenant_activos ON usuario_tenant(tenant_id) WHERE activo = TRUE;

-- ============================================================
-- CONTACTOS (datos por tenant)
-- ============================================================

CREATE TABLE contactos (                                 -- [TRANSACCIONAL]
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    tenant_id     BIGINT NOT NULL REFERENCES tenants(id) ON DELETE RESTRICT,
    nombre        VARCHAR(200) NOT NULL,
    email         VARCHAR(200),
    telefono      VARCHAR(30),
    empresa       VARCHAR(200),
    cargo         VARCHAR(100),
    fuente_id     SMALLINT REFERENCES catalogo_fuente_lead(id),
    notas         TEXT,
    -- soft delete
    activo        BOOLEAN NOT NULL DEFAULT TRUE,
    fecha_eliminacion TIMESTAMPTZ,
    -- auditoría
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by    BIGINT NOT NULL,
    updated_at    TIMESTAMPTZ,
    updated_by    BIGINT
);

-- *** CRÍTICO: tenant_id es PRIMERA columna en TODO índice ***
CREATE INDEX idx_contactos_tenant ON contactos(tenant_id);
CREATE INDEX idx_contactos_tenant_email ON contactos(tenant_id, email);
CREATE INDEX idx_contactos_tenant_nombre ON contactos(tenant_id, nombre);
CREATE INDEX idx_contactos_tenant_activos
    ON contactos(tenant_id, id) WHERE activo = TRUE;

-- Email único SOLO dentro del tenant (no global)
CREATE UNIQUE INDEX uq_contactos_tenant_email
    ON contactos(tenant_id, email) WHERE email IS NOT NULL;

-- ============================================================
-- OPORTUNIDADES (transaccional + SCD2)
-- ============================================================

CREATE TABLE catalogo_etapa_oportunidad (                -- [CATÁLOGO]
    id       SMALLINT PRIMARY KEY,
    codigo   VARCHAR(30) NOT NULL UNIQUE,                 -- NUEVA, CALIFICADA, PROPUESTA, GANADA, PERDIDA
    nombre   VARCHAR(100) NOT NULL,
    orden    SMALLINT NOT NULL,
    es_final BOOLEAN NOT NULL DEFAULT FALSE,
    es_ganada BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE oportunidades (                             -- [TRANSACCIONAL]
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    tenant_id       BIGINT NOT NULL REFERENCES tenants(id) ON DELETE RESTRICT,
    nombre          VARCHAR(200) NOT NULL,
    contacto_id     BIGINT NOT NULL REFERENCES contactos(id) ON DELETE RESTRICT,
    etapa_id        SMALLINT NOT NULL REFERENCES catalogo_etapa_oportunidad(id),
    monto_estimado  NUMERIC(14,2),
    moneda_id       SMALLINT REFERENCES catalogo_moneda(id),
    fecha_cierre    DATE,
    propietario_id  BIGINT NOT NULL REFERENCES usuarios(id),
    -- soft delete
    activo          BOOLEAN NOT NULL DEFAULT TRUE,
    -- auditoría
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by      BIGINT NOT NULL,
    updated_at      TIMESTAMPTZ,
    updated_by      BIGINT,
    CONSTRAINT chk_oportunidades_monto CHECK (monto_estimado IS NULL OR monto_estimado >= 0)
);

CREATE INDEX idx_oportunidades_tenant ON oportunidades(tenant_id);
CREATE INDEX idx_oportunidades_tenant_etapa
    ON oportunidades(tenant_id, etapa_id);
CREATE INDEX idx_oportunidades_tenant_propietario
    ON oportunidades(tenant_id, propietario_id);
CREATE INDEX idx_oportunidades_tenant_fecha
    ON oportunidades(tenant_id, created_at DESC);
CREATE INDEX idx_oportunidades_tenant_activas
    ON oportunidades(tenant_id, id) WHERE activo = TRUE;

-- Historial SCD2 de etapas
CREATE TABLE oportunidad_etapa_historial (               -- [HISTÓRICO]
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    tenant_id       BIGINT NOT NULL,                       -- redundante para RLS y queries
    oportunidad_id  BIGINT NOT NULL REFERENCES oportunidades(id) ON DELETE CASCADE,
    etapa_id        SMALLINT NOT NULL REFERENCES catalogo_etapa_oportunidad(id),
    fecha_inicio    TIMESTAMPTZ NOT NULL DEFAULT now(),
    fecha_fin       TIMESTAMPTZ,
    usuario_id      BIGINT NOT NULL REFERENCES usuarios(id),
    comentario      TEXT,
    CONSTRAINT chk_hist_etapa_vigencia
        CHECK (fecha_fin IS NULL OR fecha_fin > fecha_inicio)
);

CREATE INDEX idx_etapa_hist_oportunidad
    ON oportunidad_etapa_historial(oportunidad_id);
CREATE INDEX idx_etapa_hist_tenant_vigente
    ON oportunidad_etapa_historial(tenant_id, oportunidad_id) WHERE fecha_fin IS NULL;

CREATE UNIQUE INDEX uq_etapa_hist_un_vigente_por_oportunidad
    ON oportunidad_etapa_historial(oportunidad_id) WHERE fecha_fin IS NULL;

-- ============================================================
-- ROW LEVEL SECURITY (defensa en profundidad)
-- ============================================================

-- Función helper: lee el tenant de la sesión actual
CREATE OR REPLACE FUNCTION current_tenant_id() RETURNS BIGINT AS $$
    SELECT NULLIF(current_setting('app.tenant_id', TRUE), '')::BIGINT;
$$ LANGUAGE SQL STABLE;

-- Habilitar RLS en tablas de negocio
ALTER TABLE contactos   ENABLE ROW LEVEL SECURITY;
ALTER TABLE oportunidades ENABLE ROW LEVEL SECURITY;
ALTER TABLE oportunidad_etapa_historial ENABLE ROW LEVEL SECURITY;

-- Política: solo se ven/modifican filas del tenant actual
CREATE POLICY tenant_isolation_contactos ON contactos
    USING (tenant_id = current_tenant_id())
    WITH CHECK (tenant_id = current_tenant_id());

CREATE POLICY tenant_isolation_oportunidades ON oportunidades
    USING (tenant_id = current_tenant_id())
    WITH CHECK (tenant_id = current_tenant_id());

CREATE POLICY tenant_isolation_etapa_hist ON oportunidad_etapa_historial
    USING (tenant_id = current_tenant_id())
    WITH CHECK (tenant_id = current_tenant_id());

-- Para el rol de aplicación (no bypassea RLS por defecto)
-- Los queries deben SET app.tenant_id = X antes de operar
-- El usuario de BD (app_user) NO es superuser

-- ============================================================
-- VALIDACIÓN CRUZADA: una FK entre tablas de negocio debe
-- mantener el mismo tenant. Trigger obligatorio.
-- ============================================================

CREATE OR REPLACE FUNCTION fn_validar_tenant_hijo() RETURNS TRIGGER AS $$
DECLARE
    parent_tenant BIGINT;
BEGIN
    -- Detectar la tabla padre según la operación
    IF TG_TABLE_NAME = 'oportunidades' THEN
        SELECT tenant_id INTO parent_tenant FROM contactos WHERE id = NEW.contacto_id;
        IF parent_tenant IS DISTINCT FROM NEW.tenant_id THEN
            RAISE EXCEPTION 'Tenant mismatch en %: padre=%, hijo=%',
                TG_TABLE_NAME, parent_tenant, NEW.tenant_id;
        END IF;
    ELSIF TG_TABLE_NAME = 'oportunidad_etapa_historial' THEN
        SELECT tenant_id INTO parent_tenant FROM oportunidades WHERE id = NEW.oportunidad_id;
        IF parent_tenant IS DISTINCT FROM NEW.tenant_id THEN
            RAISE EXCEPTION 'Tenant mismatch en %: padre=%, hijo=%',
                TG_TABLE_NAME, parent_tenant, NEW.tenant_id;
        END IF;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_validar_tenant_oportunidades
    BEFORE INSERT OR UPDATE ON oportunidades
    FOR EACH ROW EXECUTE FUNCTION fn_validar_tenant_hijo();

CREATE TRIGGER trg_validar_tenant_etapa_hist
    BEFORE INSERT OR UPDATE ON oportunidad_etapa_historial
    FOR EACH ROW EXECUTE FUNCTION fn_validar_tenant_hijo();

-- ============================================================
-- AUDITORÍA GLOBAL
-- ============================================================

CREATE TABLE auditoria_global (                          -- [AUDITORÍA]
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    tenant_id       BIGINT,                               -- puede ser NULL para eventos globales
    tabla           VARCHAR(100) NOT NULL,
    registro_id     BIGINT NOT NULL,
    operacion       VARCHAR(10) NOT NULL CHECK (operacion IN ('INSERT','UPDATE','DELETE')),
    datos_antes     JSONB,
    datos_despues   JSONB,
    campos_cambiados TEXT[],
    usuario_id      BIGINT NOT NULL,
    ip_origen       INET,
    timestamp_audit TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Particionada por mes (volumen alto esperado)
CREATE TABLE auditoria_global_2026_06 PARTITION OF auditoria_global
    FOR VALUES FROM ('2026-06-01') TO ('2026-07-01');

CREATE INDEX idx_auditoria_tenant_tabla_registro
    ON auditoria_global(tenant_id, tabla, registro_id);
CREATE INDEX idx_auditoria_timestamp
    ON auditoria_global(timestamp_audit DESC);

REVOKE UPDATE, DELETE ON auditoria_global FROM app_user;

-- Trigger genérico de auditoría
CREATE OR REPLACE FUNCTION fn_auditar_trigger() RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO auditoria_global
            (tenant_id, tabla, registro_id, operacion, datos_despues, usuario_id)
        VALUES
            (current_tenant_id(), TG_TABLE_NAME, NEW.id, 'INSERT',
             to_jsonb(NEW), current_user_id());
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO auditoria_global
            (tenant_id, tabla, registro_id, operacion, datos_antes, datos_despues,
             campos_cambiados, usuario_id)
        VALUES
            (current_tenant_id(), TG_TABLE_NAME, NEW.id, 'UPDATE',
             to_jsonb(OLD), to_jsonb(NEW),
             (SELECT array_agg(key) FROM jsonb_each(to_jsonb(NEW))
              WHERE to_jsonb(NEW)->>key IS DISTINCT FROM to_jsonb(OLD)->>key),
             current_user_id());
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO auditoria_global
            (tenant_id, tabla, registro_id, operacion, datos_antes, usuario_id)
        VALUES
            (current_tenant_id(), TG_TABLE_NAME, OLD.id, 'DELETE',
             to_jsonb(OLD), current_user_id());
        RETURN OLD;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_auditar_contactos
    AFTER INSERT OR UPDATE OR DELETE ON contactos
    FOR EACH ROW EXECUTE FUNCTION fn_auditar_trigger();

CREATE TRIGGER trg_auditar_oportunidades
    AFTER INSERT OR UPDATE OR DELETE ON oportunidades
    FOR EACH ROW EXECUTE FUNCTION fn_auditar_trigger();
```

---

### [DIAGRAMA DE RELACIONES]

```
tenants (1) ────── (N) usuario_tenant (N) ────── (1) usuarios
tenants (1) ────── (N) contactos
tenants (1) ────── (N) oportunidades (1) ────── (1) contactos
tenants (1) ────── (N) oportunidad_etapa_historial (N) ─── oportunidades
contactos (1) ────── (N) oportunidades
oportunidades (1) ─── (N) oportunidad_etapa_historial (N) ─── catalogo_etapa_oportunidad
contactos (1) ────── (N) auditoria_global
oportunidades (1) ─── (N) auditoria_global
usuario_tenant (N) ─── (1) catalogo_rol
usuarios (global) —── (N) usuario_tenant
```

---

### [ANÁLISIS DE ESCALABILIDAD]

- **Crecimiento esperado:** 1,000 tenants, 10M contactos, 50M oportunidades
  en año 2. Crecimiento ~500K filas/mes en oportunidades.
- **Particionamiento obligatorio:** `auditoria_global` por mes
  (PostgreSQL `PARTITION BY RANGE`). Evaluar partición de `oportunidades`
  por hash de `tenant_id` si > 10M filas (preparar la infra ya).
- **Sharding futuro:** la PK es BIGINT global, pero `tenant_id` permite
  migrar a esquema por-tenant (schema-per-tenant) si el pool compartido
  se vuelve cuello de botella.
- **Riesgo de bloqueo:** trigger de validación de tenant en cada INSERT
  agrega latencia. Si supera 5ms, mover a validación en app con tests.
- **Hot tenant:** un tenant muy grande puede dominar el pool. Plan B:
  migrar a schema-per-tenant o BD dedicada para tenants enterprise.

---

### [ANÁLISIS DE SEGURIDAD]

| Categoría | Datos | Recomendación |
|---|---|---|
| **Aislamiento tenant** | TODA fila de negocio | RLS + trigger de validación cruzada + tests de fuga |
| PII | email, telefono contacto | TLS en tránsito. Cifrado en reposo (pgcrypto) si HIPAA/GDPR estricto |
| Credenciales | password | bcrypt/argon2, nunca MD5/SHA |
| Auditoría inmutable | todas las tablas | `REVOKE UPDATE,DELETE` + trigger |
| Catálogos globales | industria, etapa, fuente | Compartidos entre tenants, sin RLS |

**Cumplimiento:** GDPR — derecho al olvido requiere script que
borre TODAS las filas + auditoría del borrado del tenant.

**Defensa en profundidad (4 capas):**
1. **App:** siempre `SET app.tenant_id = X` antes de query.
2. **RLS:** motor rechaza queries sin tenant.
3. **Trigger de validación:** no se puede crear hijo en otro tenant.
4. **Tests E2E:** intentar insertar fila en otro tenant debe fallar.

---

### [PLAN DE MIGRACIÓN]

Sistema nuevo → no hay producción. Plan iterativo:

1. **Sprint 1:** tenants, usuarios, usuario_tenant.
2. **Sprint 2:** catálogos globales + contactos con RLS.
3. **Sprint 3:** oportunidades con RLS + trigger de validación cruzada.
4. **Sprint 4:** SCD2 de etapas + auditoría global.
5. **Sprint 5:** tests de fuga de tenant (intentar leer/escribir entre tenants
   debe fallar).
6. **Sprint 6:** particionamiento de `auditoria_global`.

**Tests obligatorios de aislamiento (CI/CD):**
```sql
-- Test 1: lectura cruzada debe devolver 0 filas
SET app.tenant_id = 1;
SELECT COUNT(*) FROM contactos;  -- solo ve contactos de tenant 1
SET app.tenant_id = 2;
SELECT COUNT(*) FROM contactos;  -- solo ve contactos de tenant 2

-- Test 2: INSERT cruzado debe fallar
SET app.tenant_id = 1;
INSERT INTO contactos (tenant_id, ...) VALUES (2, ...);
-- ERROR: violates row-level security policy
```

---

### [ESTRATEGIA DE AUDITORÍA]

- **Columnas:** `created_at`, `created_by`, `updated_at`, `updated_by`
  en todas las tablas de negocio.
- **Tabla global:** `auditoria_global` (particionada por mes) con trigger
  genérico `fn_auditar_trigger` aplicado a todas las tablas críticas.
- **Inmutabilidad:** `REVOKE UPDATE, DELETE` + partición mensual que
  se "congela" después de N días.
- **Retención:** 3 años en producción, después archive a S3 cold storage.

---

### [ESTRATEGIA DE HISTORIZACIÓN]

- **`oportunidad_etapa_historial` — SCD Tipo 2** con `tenant_id`
  redundante para queries cross-tenant del SaaS provider.
- **Índice único parcial** garantiza una sola etapa vigente por oportunidad.
- **Reporte típico:** "tiempo promedio en cada etapa por tenant" requiere
  este historial.

---

### [ADVERTENCIAS DE RENDIMIENTO E INTEGRIDAD]

- **RLS overhead:** 5-10% en queries simples, más en joins. Cachear
  `current_tenant_id()` si es cuello de botella.
- **Trigger de validación cruzada:** hace 1 query extra por INSERT.
  Aceptable hasta 10K inserts/s. Sobre eso, mover a app.
- **Índices con `tenant_id` primero:** todos los índices deben
  empezar con `tenant_id` o las queries multi-tenant se vuelven lentas.
- **JSONB en auditoría:** crece rápido. Particionar por mes y comprimir
  particiones antiguas con `pg_compress`.
- **Catálogos globales:** sin RLS, accesibles por todos los tenants.
  Si un tenant quiere catálogos custom, agregar `tenant_id` y cambiar
  a estrategia "shared + override".
- **Email único global vs. por tenant:** usuarios tienen email único
  global; contactos tienen email único por tenant (un contacto "juan@x.com"
  puede existir en tenant 1 y tenant 2 sin colisión).

---

## Ejemplo 3 — Sistema de Salud (HIS/HCE)

**Contexto del usuario (mensaje original):**
> "Necesito modelar una Historia Clínica Electrónica (HCE) en PostgreSQL.
> Tengo pacientes, médicos, consultas, diagnósticos (CIE-10), medicamentos,
> recetas, resultados de laboratorio y estudios de imagen. HIPAA exige que
> NADA se borre, que TODO cambio quede auditado, y que el acceso a datos
> de un paciente quede registrado. El sistema debe funcionar en un hospital
> grande con ~500K consultas/año."

**Supuestos documentados:**
- Motor: PostgreSQL
- Carga: OLTP
- Tolerancia: **Crítica extrema** (salud, regulado)
- PK: BIGINT
- Multitenancy: No (un solo hospital o red de centros con tenant_id opcional)
- **Soft delete: NUNCA** — todo cambio es append-only + corrección con nuevo evento
- **Auditoría: TOTAL** (toda tabla, no selectiva)
- **Historización: SCD2 + append-only** (eventos clínicos nunca se modifican)
- **Retención legal: 20 años mínimo** (normativa sanitaria típica)
- **Particionamiento: OBLIGATORIO** por fecha (volumen alto + archivado)
- **HIPAA: encriptación, accesos auditados, integridad verificable**

**Diferencia clave con e-commerce:** en salud **no se corrige un registro,
se añade un nuevo evento que lo anula o complementa**. Ej: "medicación X
prescrita el 10/01" + "medicación X suspendida el 15/01 por reacción".
Ambas filas viven para siempre.

---

### [DIAGNÓSTICO ARQUITECTÓNICO]

- **Catálogo de diagnósticos (CIE-10):** jerárquico (capítulos > categorías
  > subcategorías) → auto-referencial.
- **Eventos clínicos inmutables:** consultas, prescripciones, resultados
  de laboratorio → no se UPDATE nunca. Cualquier cambio es un nuevo evento
  con FK al original.
- **Recetas:** una receta tiene N medicamentos. Medicamentos son catálogos
  (con VADEMECUM, dosis, vía).
- **Resultados de laboratorio:** una orden genera N items, cada uno con
  valor numérico, unidad, rango de referencia.
- **Accesos a HCE:** TODA lectura de datos de paciente debe quedar registrada
  (HIPAA Security Rule).
- **Soft delete NO aplica** — registros clínicos son evidencia legal.
- **Particionamiento obligatorio** en `consultas`, `resultados_lab`,
  `auditoria_acceso` — esperado > 5M filas/año.

---

### [ÁRBOL DE DEPENDENCIAS FUNCIONALES]

```
pacientes:
  id → {mrn, nombre, fecha_nacimiento, sexo, ...}
  mrn → {id, ...}   -- UNIQUE: Medical Record Number

medicos:
  id → {nombre, especialidad_id, numero_licencia, ...}
  numero_licencia → {id, ...}   -- UNIQUE global

consultas (append-only, inmutable):
  id → {paciente_id, medico_id, fecha, motivo, notas, tipo_id}
  -- NUNCA se hace UPDATE

diagnosticos (inmutables, append-only):
  id → {consulta_id, codigo_cie10_id, es_principal, ...}

prescripciones (inmutables, append-only):
  id → {consulta_id, medicamento_id, dosis, frecuencia, duracion_dias}
  -- suspensión = NUEVA prescripción con estado SUSPENDIDA + referencia

resultados_lab (inmutables):
  id → {orden_id, item_id, valor, unidad, rango_ref_min, rango_ref_max,
        fecha_resultado, validado_por}
```

---

### [ESQUEMA DDL FINAL]

```sql
-- ============================================================
-- HISTORIA CLÍNICA ELECTRÓNICA - PostgreSQL - v1.0
-- HIPAA + append-only + particionamiento + auditoría total
-- ============================================================

-- ============================================================
-- CATÁLOGOS CLÍNICOS
-- ============================================================

CREATE TABLE catalogo_cie10 (                            -- [CATÁLOGO] jerárquico
    id           VARCHAR(10) PRIMARY KEY,                 -- "J45.0", "E11.9"
    codigo       VARCHAR(10) NOT NULL UNIQUE,
    descripcion  VARCHAR(500) NOT NULL,
    capitulo     VARCHAR(5) NOT NULL,                     -- "J", "E", "I"
    padre_id     VARCHAR(10) REFERENCES catalogo_cie10(id),
    activo       BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE INDEX idx_cie10_padre ON catalogo_cie10(padre_id);
CREATE INDEX idx_cie10_capitulo ON catalogo_cie10(capitulo);

CREATE TABLE catalogo_especialidad (                     -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(20) NOT NULL UNIQUE,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE catalogo_tipo_consulta (                    -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(30) NOT NULL UNIQUE,                   -- PRIMERA_VEZ, SEGUIMIENTO, URGENCIA
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE catalogo_medicamento (                      -- [CATÁLOGO] VADEMECUM
    id           BIGINT PRIMARY KEY,
    codigo_atc   VARCHAR(20) NOT NULL UNIQUE,             -- WHO ATC classification
    nombre_generico VARCHAR(200) NOT NULL,
    nombre_comercial VARCHAR(200),
    forma_farmaceutica VARCHAR(50) NOT NULL,             -- TABLETA, JARABE, INYECTABLE
    requiere_receta BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE TABLE catalogo_estado_receta (                    -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(20) NOT NULL UNIQUE,                   -- ACTIVA, SUSPENDIDA, COMPLETADA, CANCELADA
    nombre VARCHAR(100) NOT NULL
);

-- ============================================================
-- MAESTROS
-- ============================================================

CREATE TABLE pacientes (                                 -- [MAESTRO]
    id                  BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    mrn                 VARCHAR(20) NOT NULL UNIQUE,      -- Medical Record Number
    nombre              VARCHAR(200) NOT NULL,
    apellido            VARCHAR(200) NOT NULL,
    fecha_nacimiento    DATE NOT NULL,
    sexo                CHAR(1) NOT NULL CHECK (sexo IN ('M','F','O')),
    documento_id        VARCHAR(50),                       -- encriptado en app
    telefono_encrypted  BYTEA,                            -- pgcrypto
    email_encrypted     BYTEA,
    direccion_encrypted BYTEA,
    fecha_fallecimiento DATE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by          BIGINT NOT NULL,
    updated_at          TIMESTAMPTZ,
    updated_by          BIGINT,
    CONSTRAINT chk_pacientes_edad CHECK (
        fecha_nacimiento <= CURRENT_DATE
        AND (fecha_fallecimiento IS NULL OR fecha_fallecimiento >= fecha_nacimiento)
    )
);

CREATE INDEX idx_pacientes_apellido_nombre ON pacientes(apellido, nombre);

CREATE TABLE medicos (                                   -- [MAESTRO]
    id                BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre            VARCHAR(200) NOT NULL,
    numero_licencia   VARCHAR(50) NOT NULL UNIQUE,
    especialidad_id   SMALLINT NOT NULL REFERENCES catalogo_especialidad(id),
    email             VARCHAR(200) NOT NULL UNIQUE,
    activo            BOOLEAN NOT NULL DEFAULT TRUE,
    created_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- ============================================================
-- EVENTOS CLÍNICOS (APPEND-ONLY, inmutables)
-- ============================================================

-- *** Consultas: NUNCA se UPDATE ni DELETE ***
CREATE TABLE consultas (                                 -- [TRANSACCIONAL] inmutable
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    paciente_id     BIGINT NOT NULL REFERENCES pacientes(id),
    medico_id       BIGINT NOT NULL REFERENCES medicos(id),
    fecha           TIMESTAMPTZ NOT NULL,
    tipo_id         SMALLINT NOT NULL REFERENCES catalogo_tipo_consulta(id),
    motivo          TEXT NOT NULL,
    notas           TEXT,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by      BIGINT NOT NULL                       -- médico que registra
) PARTITION BY RANGE (fecha);

-- Particiones mensuales
CREATE TABLE consultas_2026_q1 PARTITION OF consultas
    FOR VALUES FROM ('2026-01-01') TO ('2026-04-01');
CREATE TABLE consultas_2026_q2 PARTITION OF consultas
    FOR VALUES FROM ('2026-04-01') TO ('2026-07-01');
CREATE TABLE consultas_2026_q3 PARTITION OF consultas
    FOR VALUES FROM ('2026-07-01') TO ('2026-10-01');
CREATE TABLE consultas_2026_q4 PARTITION OF consultas
    FOR VALUES FROM ('2026-10-01') TO ('2027-01-01');
CREATE TABLE consultas_default PARTITION OF consultas DEFAULT;

CREATE INDEX idx_consultas_paciente_fecha
    ON consultas(paciente_id, fecha DESC);
CREATE INDEX idx_consultas_medico_fecha
    ON consultas(medico_id, fecha DESC);

-- Diagnósticos: parte de la consulta, inmutables
CREATE TABLE consulta_diagnosticos (                     -- [TRANSACCIONAL] inmutable
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    consulta_id     BIGINT NOT NULL REFERENCES consultas(id),
    codigo_cie10_id VARCHAR(10) NOT NULL REFERENCES catalogo_cie10(id),
    es_principal    BOOLEAN NOT NULL DEFAULT FALSE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by      BIGINT NOT NULL,
    CONSTRAINT chk_un_diagnostico_principal_por_consulta
        EXCLUDE USING btree (consulta_id WITH =)
        WHERE (es_principal = TRUE)                       -- solo UN principal por consulta
);

CREATE INDEX idx_dx_consulta ON consulta_diagnosticos(consulta_id);
CREATE INDEX idx_dx_cie10 ON consulta_diagnosticos(codigo_cie10_id);

-- Prescripciones: append-only. Suspender = nueva fila con estado SUSPENDIDA
CREATE TABLE prescripciones (                            -- [TRANSACCIONAL] inmutable
    id                BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    consulta_id       BIGINT NOT NULL REFERENCES consultas(id),
    medicamento_id    BIGINT NOT NULL REFERENCES catalogo_medicamento(id),
    dosis             VARCHAR(100) NOT NULL,              -- "500mg"
    frecuencia        VARCHAR(100) NOT NULL,              -- "cada 8h"
    duracion_dias     SMALLINT,
    cantidad          INTEGER,
    estado_id         SMALLINT NOT NULL REFERENCES catalogo_estado_receta(id),
    prescripcion_padre_id BIGINT REFERENCES prescripciones(id),  -- suspensión referencia al original
    fecha_inicio      TIMESTAMPTZ NOT NULL,
    fecha_fin         TIMESTAMPTZ,
    created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by        BIGINT NOT NULL,
    CONSTRAINT chk_prescripciones_duracion CHECK (duracion_dias IS NULL OR duracion_dias > 0),
    CONSTRAINT chk_prescripciones_vigencia
        CHECK (fecha_fin IS NULL OR fecha_fin >= fecha_inicio)
) PARTITION BY RANGE (created_at);

CREATE INDEX idx_prescripciones_consulta ON prescripciones(consulta_id);
CREATE INDEX idx_prescripciones_padre ON prescripciones(prescripcion_padre_id);
CREATE INDEX idx_prescripciones_medicamento ON prescripciones(medicamento_id);

-- ============================================================
-- LABORATORIO
-- ============================================================

CREATE TABLE ordenes_lab (                               -- [TRANSACCIONAL]
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    paciente_id     BIGINT NOT NULL REFERENCES pacientes(id),
    medico_solicita BIGINT NOT NULL REFERENCES medicos(id),
    fecha_orden     TIMESTAMPTZ NOT NULL DEFAULT now(),
    estado_id       SMALLINT NOT NULL,
    created_by      BIGINT NOT NULL
) PARTITION BY RANGE (fecha_orden);

CREATE TABLE catalogo_estado_orden_lab (                 -- [CATÁLOGO]
    id     SMALLINT PRIMARY KEY,
    codigo VARCHAR(20) NOT NULL UNIQUE,                   -- PENDIENTE, EN_PROCESO, COMPLETADA, CANCELADA
    nombre VARCHAR(100) NOT NULL
);

ALTER TABLE ordenes_lab
    ADD CONSTRAINT fk_ordenes_lab_estado
    FOREIGN KEY (estado_id) REFERENCES catalogo_estado_orden_lab(id);

CREATE TABLE catalogo_examen_lab (                       -- [CATÁLOGO]
    id            BIGINT PRIMARY KEY,
    codigo        VARCHAR(20) NOT NULL UNIQUE,            -- LOINC: 2345-7 (Glucosa)
    nombre        VARCHAR(200) NOT NULL,
    unidad        VARCHAR(50),                            -- mg/dL, mmol/L
    rango_ref_min NUMERIC(12,4),
    rango_ref_max NUMERIC(12,4)
);

CREATE TABLE resultados_lab (                            -- [TRANSACCIONAL] inmutable
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    orden_id        BIGINT NOT NULL REFERENCES ordenes_lab(id),
    examen_id       BIGINT NOT NULL REFERENCES catalogo_examen_lab(id),
    valor           NUMERIC(14,4),
    valor_texto     TEXT,                                 -- para resultados no numéricos
    unidad          VARCHAR(50),                          -- snapshot al momento
    rango_ref_min   NUMERIC(12,4),                        -- snapshot
    rango_ref_max   NUMERIC(12,4),
    es_anormal      BOOLEAN,                              -- calculado por la app al insertar
    fecha_resultado TIMESTAMPTZ NOT NULL DEFAULT now(),
    validado_por    BIGINT REFERENCES medicos(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by      BIGINT NOT NULL
) PARTITION BY RANGE (fecha_resultado);

CREATE INDEX idx_resultados_orden ON resultados_lab(orden_id);
CREATE INDEX idx_resultados_paciente_examen
    ON resultados_lab(orden_id, examen_id);

-- ============================================================
-- AUDITORÍA DE ACCESOS (HIPAA Security Rule)
-- TODA lectura de datos de paciente se registra
-- ============================================================

CREATE TABLE auditoria_acceso (                          -- [AUDITORÍA]
    id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    usuario_id      BIGINT NOT NULL REFERENCES medicos(id),
    paciente_id     BIGINT NOT NULL REFERENCES pacientes(id),
    tabla           VARCHAR(100) NOT NULL,
    registro_id     BIGINT,
    operacion       VARCHAR(10) NOT NULL CHECK (operacion IN ('SELECT','INSERT','UPDATE','DELETE','PRINT','EXPORT')),
    ip_origen       INET,
    proposito       TEXT,                                 -- "tratamiento", "investigación"
    timestamp_audit TIMESTAMPTZ NOT NULL DEFAULT now()
) PARTITION BY RANGE (timestamp_audit);

-- Particiones mensuales
CREATE TABLE auditoria_acceso_2026_06 PARTITION OF auditoria_acceso
    FOR VALUES FROM ('2026-06-01') TO ('2026-07-01');
-- (crear particiones futuras con job mensual)

CREATE INDEX idx_auditoria_paciente_fecha
    ON auditoria_acceso(paciente_id, timestamp_audit DESC);
CREATE INDEX idx_auditoria_usuario_fecha
    ON auditoria_acceso(usuario_id, timestamp_audit DESC);

-- Inmutabilidad estricta
REVOKE UPDATE, DELETE ON auditoria_acceso FROM app_user;

-- ============================================================
-- INTEGRIDAD: inmutabilidad de eventos clínicos
-- ============================================================

-- Triggers que rechazan UPDATE/DELETE en tablas clínicas
CREATE OR REPLACE FUNCTION fn_rechazar_modificacion() RETURNS TRIGGER AS $$
BEGIN
    RAISE EXCEPTION 'Tabla clínica inmutable (%): operación % no permitida. '
                    'Use un nuevo evento con referencia al original.',
                    TG_TABLE_NAME, TG_OP;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_inmutable_consultas
    BEFORE UPDATE OR DELETE ON consultas
    FOR EACH ROW EXECUTE FUNCTION fn_rechazar_modificacion();

CREATE TRIGGER trg_inmutable_prescripciones
    BEFORE UPDATE OR DELETE ON prescripciones
    FOR EACH ROW EXECUTE FUNCTION fn_rechazar_modificacion();

CREATE TRIGGER trg_inmutable_resultados
    BEFORE UPDATE OR DELETE ON resultados_lab
    FOR EACH ROW EXECUTE FUNCTION fn_rechazar_modificacion();
```

---

### [DIAGRAMA DE RELACIONES]

```
pacientes (1) ────── (N) consultas (1) ────── (N) consulta_diagnosticos
pacientes (1) ────── (N) consultas (1) ────── (N) prescripciones
pacientes (1) ────── (N) ordenes_lab (1) ────── (N) resultados_lab
medicos   (1) ────── (N) consultas
medicos   (1) ────── (N) ordenes_lab (solicita)
medicos   (1) ────── (N) resultados_lab (valida)
medicamentos (cat) ─── (N) prescripciones
catalogo_cie10 (cat, jerárquico) ─── (N) consulta_diagnosticos
prescripciones (auto-referencial: prescripcion_padre_id para suspensiones)
pacientes (1) ────── (N) auditoria_acceso
```

---

### [ANÁLISIS DE ESCALABILIDAD]

- **Crecimiento esperado:** 500K consultas/año → 2.5M en 5 años.
  Resultados de lab: 5M/año.
- **Particionamiento aplicado:** `consultas`, `prescripciones`,
  `ordenes_lab`, `resultados_lab`, `auditoria_acceso` — todas por rango
  de fecha, particiones trimestrales/mensuales.
- **Job de mantenimiento:** crear particiones futuras con 3 meses de
  anticipación (cron + `CREATE TABLE ... PARTITION OF`).
- **Archivo:** particiones > 5 años se mueven a esquema `archivo_` con
  tablespace en almacenamiento frío. Permanecen consultables.
- **Riesgo de bloqueo:** las inserciones constantes en `consultas` podrían
  generar contención en el índice por `paciente_id`. Considerar índice
  hash por `paciente_id` con `FILLFACTOR=70`.

---

### [ANÁLISIS DE SEGURIDAD]

| Categoría | Datos | Recomendación |
|---|---|---|
| **PHI (Protected Health Information)** | TODOS los datos de pacientes | HIPAA Security Rule: cifrado en tránsito (TLS) y reposo (pgcrypto). Acceso registrado. |
| PII + datos clínicos | nombre, fecha_nacimiento, diagnósticos, medicamentos, resultados | Cifrado a nivel de columna con `pgcrypto` o `pgcrypto` con claves por registro. |
| Credenciales | password de médicos | bcrypt/argon2, MFA obligatorio. |
| Auditoría de accesos | HIPAA requiere | `auditoria_acceso` con cada SELECT/PRINT/EXPORT. |
| Integridad | datos clínicos | Triggers de inmutabilidad + hash chain opcional. |
| Catálogos globales | CIE-10, medicamentos, exámenes | Sin cifrado (públicos). |

**HIPAA Security Rule checklist:**
- ✅ Confidencialidad: cifrado + acceso restringido
- ✅ Integridad: append-only + hash chain
- ✅ Disponibilidad: backups + particionamiento
- ✅ Audit trail: `auditoria_acceso` con cada operación

**GDPR adicional (si aplica):** el derecho al olvido entra en tensión
con retención legal de 20 años. La base legal de HIPAA prevalece sobre
GDPR en este caso, pero documentar la base legal.

---

### [PLAN DE MIGRACIÓN]

Esquema desde cero, sin producción. Plan:

1. **Mes 1:** catálogos clínicos (CIE-10, medicamentos, exámenes).
2. **Mes 2:** pacientes y médicos con cifrado.
3. **Mes 3:** consultas inmutables + particionamiento.
4. **Mes 4:** prescripciones, diagnósticos, resultados.
5. **Mes 5:** auditoría de accesos + triggers de inmutabilidad.
6. **Mes 6:** tests E2E que validen que un UPDATE a `consultas` falle
   y que toda lectura quede registrada.

**Tests HIPAA obligatorios (CI/CD):**
```sql
-- Test 1: UPDATE en tabla inmutable debe fallar
UPDATE consultas SET notas = 'modificado' WHERE id = 1;
-- ERROR: Tabla clínica inmutable (consultas)

-- Test 2: DELETE debe fallar
DELETE FROM prescripciones WHERE id = 1;
-- ERROR: Tabla clínica inmutable (prescripciones)

-- Test 3: Cada SELECT genera fila en auditoria_acceso
SELECT * FROM pacientes WHERE id = 1;
SELECT COUNT(*) FROM auditoria_acceso WHERE paciente_id = 1;
-- Debe ser >= 1
```

---

### [ESTRATEGIA DE AUDITORÍA]

- **Inmutabilidad clínica:** triggers `BEFORE UPDATE OR DELETE` que
  lanzan excepción en `consultas`, `prescripciones`, `resultados_lab`.
  Corrección = nuevo evento con FK al original.
- **Auditoría de accesos (HIPAA):** `auditoria_acceso` con cada
  SELECT/INSERT/UPDATE/DELETE/PRINT/EXPORT. Particionada por mes.
- **Inmutabilidad de la auditoría:** `REVOKE UPDATE, DELETE`.
- **Retención:** 20 años (normativa sanitaria típica) en producción,
  después archivo a S3 con Glacier.

---

### [ESTRATEGIA DE HISTORIZACIÓN]

- **Eventos append-only:** no se necesita SCD2 clásico. Cada cambio
  es un nuevo evento (`prescripcion_padre_id` para suspensiones).
- **Corrección de errores:** se hace con un nuevo evento de tipo
  "anulación" o "corrección" que referencia al original. Ambos
  registros coexisten.
- **Catálogo CIE-10:** histórico se mantiene, nuevos códigos se
  añaden con `activo = TRUE`. Códigos obsoletos se marcan `activo = FALSE`
  pero no se borran (los diagnósticos ya emitidos los referencian).

---

### [ADVERTENCIAS DE RENDIMIENTO E INTEGRIDAD]

- **Triggers de inmutabilidad:** overhead en cada INSERT/UPDATE. El
  costo es despreciable (solo verifica TG_OP). Pero verificar que
  ningún proceso intente hacer UPDATE en estas tablas.
- **Cifrado con pgcrypto:** las queries sobre columnas cifradas
  (Búsqueda por nombre, etc.) son imposibles sin descifrar antes.
  Estrategia: índice de búsqueda separado con hash del valor
  (ej: `nombre_hash = md5(nombre)` para búsquedas case-insensitive
  sin descifrar).
- **EXCLUDE constraint** en `consulta_diagnosticos`: PostgreSQL es
  el único motor que soporta esto. MySQL/SQL Server requieren trigger.
- **Particiones:** cualquier `ALTER TABLE` sobre la tabla padre se
  propaga a todas las particiones. Crear particiones futuras con
  suficiente anticipación.
- **Auditoría de accesos de alto volumen:** puede escribir 10K+ filas/s
  en hospital grande. Considerar envío asíncrono a Kafka antes de
  insertar en PostgreSQL.
- **EXCLUDE constraint en MySQL/SQL Server:** no existe, usar trigger
  + UNIQUE index con `es_principal` derivado (no recomendado, perder
  semántica).

---

# ⚠️ Anti-Patrones Avanzados

Los anti-patrones de la Fase 0 cubren errores de diseño básicos. Esta
sección cubre errores de **arquitectura y proceso** que solo se ven
después de años de experiencia o en proyectos a gran escala.

---

## 1. Sobre-Ingeniería (Over-Engineering)

### 1.1 Normalización excesiva prematura

**Síntoma:** aplicar 4FN/5FN o modelar cada dependencia funcional
como tabla separada en un proyecto pequeño.

```sql
-- ❌ Sobre-ingeniería: proyecto de 200 empleados
CREATE TABLE empleados_nombres (id, nombre, apellido);
CREATE TABLE empleados_contactos (id, telefono, email);
CREATE TABLE empleado_telefonos (empleado_id, telefono, tipo);
CREATE TABLE empleado_emails (empleado_id, email, tipo);
CREATE TABLE empleado_nombre_historial (...);  -- nadie lo pidió
```

**Por qué es malo:** el costo de JOINs, mantenimiento, migraciones y
capacitación del equipo no se compensa con el beneficio.

**Regla:** normaliza hasta la forma que el **caso de uso actual** exija.
Empieza en 3FN. Solo sube a BCNF/4FN/5FN cuando el volumen o la
criticidad lo justifiquen.

**Test de justificación antes de añadir complejidad:**
- ¿Hay hoy un caso de uso real que requiera esta separación?
- ¿La complejidad añadida (más tablas, más JOINs) es compensada por
  un beneficio concreto (rendimiento, integridad, auditoría)?
- ¿El equipo puede mantenerlo sin errores?

### 1.2 Auditoría total prematura

**Síntoma:** activar triggers de auditoría en TODAS las tablas
("por si acaso") desde el día 1.

**Por qué es malo:**
- Cada INSERT/UPDATE escribe también en `auditoria_*` → 2x almacenamiento
  y latencia.
- La tabla de auditoría se vuelve un cuello de botella en sí misma.
- Datos auditados que nadie consulta = desperdicio puro.

**Regla:** auditar selectivamente. Pregunta: "¿quién va a consultar
este log y para qué?" Si no hay respuesta clara, no audites.

**Matriz de decisión:**
| Tabla | ¿Auditar? | Razón |
|---|---|---|
| `pedidos`, `pagos` | ✅ SÍ | financieras, críticas |
| `productos`, `categorias` | ⚠️ SELECTIVO | solo cambios de precio, activación |
| `direcciones_envio` | ❌ NO | datos operativos sin valor histórico |
| `logs_app` | ❌ NO | ya es un log por diseño |
| `temp_xxx` (temporales) | ❌ NO | se borran igual |

### 1.3 Soft delete universal

**Síntoma:** agregar `deleted_at` a TODAS las tablas "por seguridad".

**Por qué es malo:**
- Toda query operativa debe añadir `WHERE deleted_at IS NULL` y
  olvidarlo = fuga de datos "eliminados".
- Los índices deben incluir `deleted_at` o el rendimiento se hunde.
- Dificulta restores de backups y replica.
- Crea ambigüedad: ¿una fila `deleted_at` no-NULL sigue contando
  para el stock? ¿para el saldo?

**Regla:** soft delete solo en datos transaccionales que el usuario
puede querer recuperar (carrito, borrador, pedido cancelable). En
catálogos y maestros usar `activo BOOLEAN` (estado lógico, no eliminación).
En datos clínicos/legales NO usar — son evidencia, deben permanecer
visibles con un campo de estado, no "eliminados".

### 1.4 Multitenancy sin necesidad real

**Síntoma:** añadir `tenant_id`, RLS y triggers de validación desde
el día 1 en un sistema que aún no tiene 2 clientes.

**Por qué es malo:**
- 30-50% de overhead en queries y mantenimiento.
- RLS + tests de fuga + triggers de validación = mucha complejidad
  para un problema que aún no existe.
- Dificulta iteración rápida en MVP.

**Regla:** empieza con single-tenant. Cuando llegue el **segundo
cliente real con datos separados**, refactoriza a multi-tenant. El
coste de la migración es finito y conocido; el coste de la complejidad
prematura es permanente.

**Sin embargo:** si el producto se vende COMO SaaS multi-tenant desde
el inicio, la decisión es diferente. Es un trade-off consciente.

---

## 2. Normalización Incorrecta

### 2.1 "JSON para evitar crear tablas"

**Síntoma:**
```sql
CREATE TABLE pedidos (
    id BIGINT PRIMARY KEY,
    datos JSONB  -- {productos: [...], cliente: {...}, pagos: [...]}
);
```

**Por qué es malo:** pierde todas las garantías relacionales:
- Imposible aplicar FK
- Imposible validar tipos con CHECK
- Imposible JOINear eficientemente
- Imposible auditar cambios a nivel de subdocumento
- Replicación de datos "anidados" sin integridad

**Cuándo SÍ es válido JSONB:**
- Atributos realmente semi-estructurados (configuración de usuario,
  respuesta de API externa que no se normaliza).
- Catálogo de extensiones/plugins con schema dinámico.
- **Nunca** como sustituto de entidades del dominio.

### 2.2 "Todo en una sola tabla"

**Síntoma:** una tabla con 50+ columnas que mezcla entidades.

```sql
CREATE TABLE users_everything (
    id, email, password, name, address, phone, role, plan,
    company_name, company_tax_id, company_address, company_phone,
    billing_card_last4, billing_card_exp, billing_card_brand,
    -- ... y 30 columnas más
);
```

**Por qué es malo:** ya cubierto en Fase 0 (Tabla Dios), pero el
problema operacional es que cada query carga 50 columnas y los
índices son enormes.

### 2.3 "PKs naturales con significado"

**Síntoma:** usar email, número de documento, o código de producto
como PK.

**Por qué es malo:**
- Si el email cambia, hay que cambiar la PK → propagación en cascada.
- Los JOINs son más lentos (VARCHAR vs INT).
- Expone información de negocio en la PK (filtrable en logs, errores).
- UUIDs/INTs permiten mejor fragmentación y sharding.

**Regla:** PKs surrogadas SIEMPRE. Las claves naturales van como
`UNIQUE` separado.

### 2.4 "Herencia de un solo ID para todo"

**Síntoma:** un campo `entity_type` + `entity_id` como FK polimórfica.

```sql
CREATE TABLE comentarios (
    id BIGINT PRIMARY KEY,
    entity_type VARCHAR(20),  -- 'pedido', 'producto', 'cliente'
    entity_id BIGINT,
    texto TEXT
);
```

**Por qué es malo:** imposible aplicar FK, imposible JOIN directo,
índices ineficientes.

**Solución:** una tabla de comentarios por entidad, o una tabla
única con FKs NULL + CHECK que solo una esté poblada.

### 2.5 "Catálogos con códigos en lugar de IDs"

**Síntoma:**
```sql
CREATE TABLE pedidos (
    id BIGINT PRIMARY KEY,
    estado VARCHAR(20)  -- 'PENDIENTE', 'PAGADO', ...
);
```

**Por qué es malo:**
- No hay catálogo asociado (no se puede añadir descripción, orden, color).
- No se puede desactivar un valor sin migrar datos.
- Cambiar el nombre visible requiere UPDATE masivo.

**Solución:** tabla catálogo con PK propia y FK.

---

## 3. Anti-Patrones de Migración

### 3.1 "DROP COLUMN directo en producción"

**Síntoma:** añadir columna, popularla, y luego `ALTER TABLE ... DROP COLUMN`
de la columna vieja en horas pico.

**Por qué es malo:** en motores como MySQL o SQL Server, DROP COLUMN
adquiere lock de tabla durante la operación (puede ser minutos con
millones de filas). Lecturas y escrituras se bloquean.

**Solución:** dejar la columna vieja indefinidamente (no cuesta
mucho espacio en una BD moderna). DROP solo en ventanas de
mantenimiento o con herramientas de migración online (pt-online-schema-change,
gh-ost, pg_repack).

### 3.2 "Borrar y re-insertar"

**Síntoma:** para "limpiar" una tabla, hacer `TRUNCATE` y re-importar
desde CSV.

**Por qué es malo:** pierde la historia, los IDs autogenerados, las
FKs, los triggers, los índices. Y no resuelve el problema subyacente.

### 3.3 "ALTER TABLE ADD COLUMN sin valor DEFAULT"

**Síntoma:**
```sql
ALTER TABLE pedidos ADD COLUMN procesado BOOLEAN;
-- En MySQL/InnoDB antiguo: el ALTER espera reescribir TODA la tabla.
```

**Solución:** siempre con `DEFAULT` o `NOT NULL DEFAULT` para que
el ALTER sea metadata-only.

---

## 4. Anti-Patrones de ORM (vs. DDL Manual)

Los ORMs (Hibernate, Entity Framework, SQLAlchemy, ActiveRecord,
Prisma) son herramientas poderosas que introducen anti-patrones
específicos. Saber cuándo escribir DDL a mano es crítico.

### 4.1 "Migrations del ORM como fuente de verdad"

**Síntoma:** el equipo nunca escribe DDL, todo viene de migraciones
del ORM (Rails, Django migrations, etc.).

**Cuándo es OK:** MVP, equipos de 1-3 personas, sistema pequeño.
**Cuándo NO:** sistemas con DBAs, sistemas regulados, multi-tenant
con RLS (muchos ORMs no manejan RLS bien), sistemas con particionamiento
(los ORMs rara vez generan particiones).

**Recomendación:** el ORM maneja migraciones triviales (add column,
create table simple). El DBA/especialista escribe migraciones para:
- RLS, triggers, funciones, políticas
- Particionamiento
- Índices especializados
- Auditoría inmutable

### 4.2 "Eager loading al infinito"

**Síntoma:** el ORM genera `SELECT ... FROM a JOIN b JOIN c JOIN d JOIN e`
porque "siempre cargo todo".

**Por qué es malo:** query gigante, latencia alta, columnas no usadas.

**Solución:** usar lazy loading o `select_related`/`includes` deliberadamente.

### 4.3 "N+1 queries invisible"

**Síntoma:**
```python
# Código ORM
for pedido in pedidos:  # 1 query
    print(pedido.cliente.nombre)  # N queries más (una por pedido)
```

**Por qué es malo:** 1 + N queries cuando debería ser 2. Se vuelve
catastrófico con 10K pedidos.

**Solución:** eager loading explícito (`joinedload`, `selectinload`).

### 4.4 "ORM genera SQL, no lo revisa nadie"

**Síntoma:** los índices no existen porque el ORM "se encarga".

**Solución:** el ORM no crea índices especializados (compuestos,
parciales, cubridores). El DBA los añade con migraciones manuales.

### 4.5 "Tipos del ORM vs. tipos del motor"

**Síntoma:** el ORM usa `String(255)` para todo, ignorando
`VARCHAR(50)`, `CHAR(2)`, `TEXT`, `ENUM` del motor.

**Por qué es malo:** pierde optimización de almacenamiento, semántica
del dato, y CHECK constraints nativos.

**Solución:** mapear explícitamente o usar columnas personalizadas
del ORM.

---

## 5. Anti-Patrones de Proceso

### 5.1 "Diseñar la BD al final"

**Síntoma:** el equipo diseña la API, los endpoints, la UI, y al final
"ahorita vemos cómo queda la BD".

**Por qué es malo:** la BD es el contrato más difícil de cambiar
(migraciones costosas, downtime). Una API mal diseñada se puede
versionar; una BD mal diseñada se reescribe.

**Regla:** diseñar la BD ANTES de la API. La API es una vista sobre
la BD; la BD es el modelo del dominio.

### 5.2 "Cambios de esquema sin revisar por el equipo"

**Síntoma:** cada developer hace migraciones en su branch sin
revisión, y al integrar hay conflictos y migraciones que se pisan.

**Solución:** migrations con revisión obligatoria, ejecutadas en
orden con timestamps, probadas en staging antes de producción.

### 5.3 "Producción y desarrollo divergentes"

**Síntoma:** la BD local del developer tiene cambios que nunca
llegaron a producción. La BD de producción tiene cambios manuales
que nadie más conoce.

**Solución:** migrations como código, ejecutadas en cada entorno,
mismo DDL en local/staging/prod (con datos distintos).

### 5.4 "No documentar decisiones de diseño"

**Síntoma:** un campo `tipo` con valores 'A', 'B', 'C' que nadie
sabe qué significan. Una columna `metadata JSONB` que se llena con
lo que cada developer quiere.

**Solución:** el esquema ES documentación. Catálogos, CHECK constraints
y comentarios inline son la primera línea. ADRs (Architecture Decision
Records) para decisiones de fondo.

---

## 6. Anti-Patrones de Rendimiento Disfrazados

### 6.1 "Más índices = más rápido"

**Falso.** Cada índice acelera lecturas pero **ralentiza escrituras**
(insert/update/delete mantienen todos los índices). En tablas con
muchas escrituras, tener 15 índices puede ser peor que tener 3.

**Regla:** crear índices justificados por queries reales. Medir antes
y después. Eliminar índices no usados (`pg_stat_user_indexes`).

### 6.2 "SELECT * en producción"

**Síntoma:** la app hace `SELECT * FROM pedidos` y solo usa 3 columnas.

**Por qué es malo:** transfiere todas las columnas por la red, satura
el buffer pool, no se beneficia de índices cubridores (covering indexes).

### 6.3 "COUNT(*) en tablas grandes"

**Síntoma:** mostrar "total de pedidos" con `SELECT COUNT(*) FROM pedidos`
en cada carga de página.

**Por qué es malo:** en tablas de millones de filas, COUNT(*) es O(n).
Solución: tabla de contadores materializados, `pg_class.reltuples` (aproximado),
o cache en Redis.

### 6.4 "OFFSET grande para paginación"

**Síntoma:** `LIMIT 20 OFFSET 100000` para "página 5000".

**Por qué es malo:** el motor tiene que leer y descartar 100K filas.
Solución: paginación por cursor (`WHERE id > last_id LIMIT 20`).

---

## Resumen: Regla de los 3 Criterios

Antes de añadir cualquier complejidad (auditoría, soft delete,
multitenancy, RLS, historización, particionamiento), valida:

1. **¿Hay caso de uso real hoy?** Si no, no lo hagas.
2. **¿El beneficio supera el costo?** Mide con números, no intuiciones.
3. **¿El equipo puede mantenerlo?** La complejidad que nadie entiende
   se convierte en bugs y deuda.

Si los 3 son SÍ, adelante. Si alguno es NO, espera.

---

# 🛠️ Herramientas de Migración

Las migraciones de esquema son el momento de mayor riesgo en la vida
de un sistema. Esta sección documenta las herramientas líderes por
ecosistema, sus trade-offs y cuándo elegir cada una.

---

## Visión General

| Herramienta | Ecosistema | Tipo | Multi-tenant friendly | Transaccional | Idempotente | Open Source |
|---|---|---|---|---|---|---|
| **Flyway** | Java/JVM | SQL + Java | ✅ | ✅ (per migration) | ✅ | ✅ (Community) |
| **Liquibase** | Java/JVM, polyglot | XML/YAML/JSON/SQL | ✅ | ✅ | ✅ | ✅ |
| **Alembic** | Python | Python + SQL | ✅ | ✅ | ⚠️ parcial | ✅ |
| **sqitch** | Perl (multi-lang) | SQL puro | ✅ | ✅ (verificación) | ✅ | ✅ |
| **Django migrations** | Django/Python | Python | ⚠️ | ✅ | ❌ | ✅ |
| **Rails migrations** | Rails/Ruby | Ruby | ⚠️ | ✅ | ❌ | ✅ |
| **Prisma migrate** | TypeScript/Node | Prisma schema | ⚠️ | ❌ | ❌ | ✅ |
| **knex.js migrate** | Node.js | JavaScript | ⚠️ | ❌ | ❌ | ✅ |
| **golang-migrate** | Go | SQL | ✅ | ❌ | ❌ | ✅ |
| **dbmate** | Multi-lang | SQL puro | ✅ | ❌ | ❌ | ✅ |
| **Atlas** | Multi-lang | HCL/SQL | ✅ | ⚠️ | ✅ | ✅ |

**Leyenda:**
- ✅ Soporte nativo / muy bueno
- ⚠️ Soporte parcial / requiere trabajo extra
- ❌ No soportado

---

## Comparación Detallada

### Flyway

**Fortalezas:**
- SQL puro (los developers de DB lo aman).
- Versionado numérico claro (`V1__init.sql`, `V2__add_pedidos.sql`).
- Validación de checksum (no se puede editar una migración aplicada).
- Repeatable migrations para views, procedures, datos seed.
- Callbacks en Java/Kotlin para migraciones complejas.

**Debilidades:**
- Edición de una migración aplicada requiere `flyway:repair` (quema
  el historial).
- No tiene "merge" automático de migraciones concurrentes.
- Licencia Community vs. Teams (algunas features son pagas).

**Cuándo elegirlo:**
- Equipo backend en Java/Kotlin/Spring.
- Quieres SQL puro sin lógica de aplicación.
- Sistema regulado (necesitas auditoría de cambios de esquema).

**Ejemplo de uso:**
```
db/migration/
  V1__schema_inicial.sql
  V2__create_pedidos.sql
  V3__add_soft_delete_pedidos.sql
  R__vistas_reportes.sql         -- repeatable
```

```sql
-- V3__add_soft_delete_pedidos.sql
ALTER TABLE pedidos
    ADD COLUMN deleted_at TIMESTAMPTZ,
    ADD COLUMN deleted_by BIGINT REFERENCES usuarios(id);

CREATE INDEX idx_pedidos_activos ON pedidos(id) WHERE deleted_at IS NULL;
```

**Integración con multi-tenancy:**
Flyway no entiende de tenants; ejecuta el mismo script en todos
los schemas. Para multi-schema (schema-per-tenant) usar Flyway
`placeholders` con `flyway.schemas=tenant1,tenant2,tenant3`. Para
pool compartido con `tenant_id`, Flyway aplica el DDL una vez
y la lógica de filtrado se delega a la app/RLS.

**Patrón expand-contract con Flyway:**
```sql
-- V10__add_email_new.sql
ALTER TABLE clientes ADD COLUMN email_new VARCHAR(200);
CREATE INDEX idx_clientes_email_new ON clientes(email_new);

-- V11__backfill_email.sql (puede tardar, run fuera de horario)
UPDATE clientes SET email_new = email WHERE email_new IS NULL;

-- V12__switch_reads.sql (la app cambia a email_new, no DROP aún)

-- V13__drop_email_old.sql (meses después, ventana de mantenimiento)
ALTER TABLE clientes DROP COLUMN email;
```

---

### Liquibase

**Fortalezas:**
- Formato XML/YAML/JSON además de SQL → programático, con lógica
  condicional.
- Rollback automático por cada cambio (no solo expansión).
- Contextos y labels para aplicar cambios por entorno.
- Preconditions (ejecutar migración X solo si la tabla Y existe).
- Policies (multi-DB: mismo cambio genera SQL para distintos motores).

**Debilidades:**
- XML verboso. YAML/JSON más limpio pero menos popular.
- Curva de aprendizaje mayor que Flyway.
- Cambios programáticos (groovy/java) requieren equipo con skill.

**Cuándo elegirlo:**
- Necesitas rollback automático.
- Cambios de esquema que dependen del estado actual (conditional DDL).
- Multi-motor (PostgreSQL para prod, MySQL para staging).

**Ejemplo (changelog YAML):**
```yaml
databaseChangeLog:
  - changeSet:
      id: 1
      author: juan
      changes:
        - createTable:
            tableName: pedidos
            columns:
              - column:
                  name: id
                  type: bigint
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: total
                  type: numeric(12,2)
                  constraints:
                    nullable: false
```

**Rollback:**
```yaml
rollback:
  - dropTable:
      tableName: pedidos
```

---

### Alembic

**Fortalezas:**
- Estándar de facto en Python (FastAPI, Flask, Django-no-migrations).
- Migraciones escritas en Python puro → lógica programática rica.
- Autogenerate de migraciones detectando diffs del modelo SQLAlchemy.
- Branching y merge de migraciones.

**Debilidades:**
- Autogenerate no es perfecto (revisar siempre el output).
- Migraciones en Python = más complejo que SQL puro.
- Transaccionalidad depende del motor y de la migración individual.

**Cuándo elegirlo:**
- Stack Python (FastAPI, SQLAlchemy, Flask).
- Quieres migraciones que hagan transformaciones de datos en Python.

**Ejemplo:**
```python
# alembic/versions/abc123_add_soft_delete.py
def upgrade():
    op.add_column('pedidos',
        sa.Column('deleted_at', sa.DateTime(timezone=True), nullable=True))
    op.create_index('idx_pedidos_activos', 'pedidos', ['id'],
                    postgresql_where=sa.text('deleted_at IS NULL'))

def downgrade():
    op.drop_index('idx_pedidos_activos', table_name='pedidos')
    op.drop_column('pedidos', 'deleted_at')
```

**Con SQLAlchemy + multi-tenancy:**
```python
# event listener para inyectar tenant_id en todas las queries
from sqlalchemy import event

@event.listens_for(Session, 'do_orm_execute'):
    def filter_by_tenant(state):
        if not state.is_select:
            return
        tenant_id = get_current_tenant()
        # Añadir WHERE tenant_id = X a cada query
        ...
```

---

### sqitch

**Fortalezas:**
- **SQL puro** (sin dependencia de lenguaje de la app).
- **Verificación**: cada migración tiene scripts `verify.sql` que
  confirman que el cambio se aplicó (ej: contar filas, ver estructura).
- Modelo de "deploy" y "revert" explícitos y separados.
- Funciona con cualquier BD y cualquier lenguaje.

**Debilidades:**
- No tan popular como Flyway/Liquibase en comunidad.
- Requiere disciplina de escribir los scripts verify.
- CLI menos pulido que la competencia.

**Cuándo elegirlo:**
- Quieres SQL puro sin acoplar a un lenguaje/framework.
- Sistema crítico donde necesitas verificación de cada cambio.
- Equipos DBAs (no developers) manejan el esquema.

**Ejemplo de estructura:**
```
sqitch/
  deploy/
    01_schema_inicial.sql
    02_create_pedidos.sql
    03_add_soft_delete.sql
  revert/
    03_add_soft_delete.sql
    02_create_pedidos.sql
    01_schema_inicial.sql
  verify/
    01_schema_inicial.sql   -- psql -c "\dt" | wc -l
    02_create_pedidos.sql   -- SELECT count(*) FROM pedidos
```

---

### Django Migrations

**Fortalezas:**
- Integrado con el ORM de Django.
- `makemigrations` autogenera migraciones desde los modelos.
- Dependencias automáticas entre migraciones.

**Debilidades:**
- Acoplado a Django.
- Mal manejado en multi-tenant (cada tenant necesita su schema y
  Django no lo hace nativamente — se usan packages como
  `django-tenants`).
- No tiene "merge" elegante: dos developers con migraciones paralelas
  generan conflictos.

**Cuándo elegirlo:** ya estás 100% en Django y tu multi-tenancy
es schema-per-tenant con `django-tenants`.

---

### Rails ActiveRecord Migrations

**Similar a Django:** cómodo dentro de Rails, doloroso fuera.
Schema_migration table es simple. Limitado para casos avanzados
(Rollback no es atómico por default en todas las versiones).

---

### Prisma Migrate

**Fortalezas:**
- Schema declarativo en TypeScript.
- Genera SQL desde el schema.
- Type-safety end-to-end.

**Debilidades:**
- Migraciones generadas, no editables a mano (pierde flexibilidad).
- No soporta features avanzados: RLS, particiones, triggers nativos.
- Para esos, hay que usar `prisma migrate --create-only` y editar
  el SQL manualmente.

**Cuándo elegirlo:** stack TypeScript/Node, equipo que prefiere
schema declarativo.

---

### golang-migrate

**Fortalezas:**
- Binario simple, SQL puro.
- Funciona con cualquier framework Go (Gin, Echo, Chi).
- Buena librería `database/sql`.

**Debilidades:**
- Sin transaccionalidad garantizada por migration.
- Sin autogenerate.

**Cuándo elegirlo:** backend Go, quieres SQL puro sin acoplar a framework.

---

## Decisión: ¿Cuál Elegir?

```
¿Stack principal?
├── Java/Kotlin/Spring
│     → Flyway (SQL puro) o Liquibase (rollback + condicionales)
├── Python (Django)
│     → Django migrations (si no usas multi-schema) o Alembic (si usas)
├── Python (FastAPI/Flask/SQLAlchemy)
│     → Alembic
├── TypeScript/Node
│     → Prisma migrate (si todo es declarativo)
│     → knex.js migrate (si prefieres control manual)
│     → golang-migrate style con node-pg-migrate (si SQL puro)
├── Go
│     → golang-migrate
├── Ruby
│     → Rails migrations
├── Equipo DBA/DB-centric
│     → sqitch o Flyway
└── Multi-motor (PostgreSQL + MySQL + SQL Server)
      → Liquibase (políticas cross-DB)
```

---

## Patrones Universales (Cualquier Herramienta)

### 1. Migrations con Timestamp + Secuencia

```sql
-- V20260615_1200__create_pedidos.sql  (Flyway style)
-- 20260615_120003_create_pedidos.sql  (timestamp style)
```

Cada migration tiene timestamp que indica orden. La herramienta
las ejecuta en orden lexicográfico.

### 2. Nunca Editar Migrations Aplicadas

Si una migration ya está en producción, NO la edites. Crea una
nueva que corrija. Razones:
- El checksum queda invalidado.
- Environments divergen (local: nueva, prod: vieja).
- Backups de BD no incluyen las migrations corregidas.

**Excepción:** si el cambio NO se aplicó a NINGÚN environment (ni
local, ni staging), se puede editar.

### 3. Expand-Contract para Cambios Incompatibles

```sql
-- EXPAND: añadir nueva columna/tabla
ALTER TABLE clientes ADD COLUMN email_new VARCHAR(200);

-- BACKFILL: copiar datos (puede tardar)
UPDATE clientes SET email_new = email WHERE id BETWEEN 1 AND 10000;

-- DUAL WRITE: app escribe a ambos
-- (cambio en código, no en SQL)

-- SWITCH: app lee de email_new
-- (cambio en código)

-- CONTRACT: eliminar columna vieja (meses después)
ALTER TABLE clientes DROP COLUMN email;
```

### 4. Validar Antes de Aplicar

En CI/CD, antes de aplicar a producción:
1. Aplicar migrations a una BD staging con datos sintéticos.
2. Ejecutar suite de tests E2E.
3. Medir tiempo de cada migration.
4. Si alguna migration tarda > 5 minutos, replantear.

### 5. Locking Awareness

En MySQL, `ALTER TABLE` con cambios no-online requiere lock de
escritura. PostgreSQL es más amigable (MVCC). SQL Server varía
por versión.

**Para MySQL con `ALTER TABLE` pesados:** usar `pt-online-schema-change`
(Percona) o `gh-ost` (GitHub). Crean una tabla sombra, copian datos,
y hacen swap. Cero downtime.

### 6. Test de Rollback

Cada migration debe tener un `down` o `revert` definido.
Aunque rara vez se ejecute, documenta la intención y
descubre dependencias ocultas.

---

## Workflow Recomendado (Cualquier Herramienta)

```
1. Developer crea migration en su branch
2. PR con revisión de DBAs/senior
3. CI aplica migration a BD temporal + corre tests
4. Merge a main
5. CI despliega a staging automáticamente
6. QA valida en staging
7. CD aplica a producción (en horario de bajo tráfico)
8. Monitoreo post-deploy: latencia, errores, locks
9. Plan B listo: rollback script en mano
```

---

## Multi-Tenant y Migrations

**Desafío:** las migrations deben aplicarse a TODOS los tenants, no
solo a uno. Las herramientas que se integran con multi-tenancy:

| Herramienta | Schema-per-tenant | Pool compartido + tenant_id | BD-por-tenant |
|---|---|---|---|
| Flyway | `flyway.schemas=tenant1,tenant2,tenant3` | Una sola BD, `tenant_id` se añade en cada tabla | Una BD por tenant, ejecución manual o script |
| Liquibase | `schemas` tag en changelog | Igual que Flyway | Igual que Flyway |
| Alembic | `op.execute()` con SQL dinámico por schema | Migrations estándar, RLS se aplica manualmente | Manual o via script |
| Django + django-tenants | Nativo (TENANT_APPS) | Soporte parcial | Manual |

**Recomendación para SaaS multi-tenant:**
- **Pool compartido:** cualquier herramienta estándar. Las migrations
  no cambian; la lógica de tenant_id está en la app/RLS.
- **Schema-per-tenant:** Flyway o Liquibase con `placeholders`
  parametrizados. La migration `V1__init.sql` se aplica a cada schema
  de tenant nuevo automáticamente.
- **BD-por-tenant:** script propio de provisioning que crea la BD,
  aplica migrations estándar, y registra el tenant en una BD de control.

---

## Tono y Estilo de Interacción

Explica cada movimiento como si el usuario fuera a defender el diseño ante
su equipo de ingeniería: razona con precisión, no ejecutes mecánicamente.

Cuando detectes una decisión de diseño ambigua, presenta las dos opciones
con sus trade-offs explícitos en lugar de elegir arbitrariamente.

Nunca presentes el esquema final sin haber mostrado primero el diagnóstico.
El usuario necesita entender el problema antes de ver la solución.

Si el esquema recibido ya está en 3FN/BCNF, confírmalo explícitamente y
ofrece solo optimizaciones adicionales: índices, restricciones CHECK/UNIQUE
faltantes, ajustes de tipos de dato específicos del motor, oportunidades
de auditoría/historización, particionamiento o seguridad.

Si el usuario menciona una migración desde un esquema existente (no un diseño
desde cero), pregunta si tiene datos en producción — las restricciones
de migración sin downtime cambian el orden de aplicación de los cambios DDL.
