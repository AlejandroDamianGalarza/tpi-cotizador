# Diccionario de datos del backend intermediario

Segunda entrega del TPI. **Estado: borrador para revisión.** Este diccionario documenta cada tabla, campo y vista del [esquema y DER](01-esquema-y-der.md). Los tipos son los previstos para PostgreSQL 16. Las restricciones que involucran varias columnas se describen al final de cada tabla.

## Convenciones

| Abreviatura | Significado |
| --- | --- |
| **A**, **P**, **D**, **PN-xx**, **CP** | Aprobado, Propuesta N1, Diferido, propuesta nueva de este borrador y dependencia de un contrato de core preliminar, igual que en el [esquema](01-esquema-y-der.md). |
| ACTA | [Acta de constitución](../../fuentes/entrega-01/acta-constitucion.md) |
| ANX §n | [Anexo técnico 2.2](../../fuentes/entrega-01/anexo-tecnico-v2-2.md), sección n |
| ALC | [Alcance por niveles](../../refinamiento/alcance-por-niveles-y-bases-de-extension.md) |
| R n.m | Refinamiento n.m de [`docs/refinamiento`](../../refinamiento/) |
| D02 | [Decisiones DER-02](../../refinamiento/decisiones-der-02-perfil-y-vigencia.md) |
| D03 §n | [Decisiones DER-03](../../refinamiento/decisiones-der-03.md), sección n |
| INI | [Inicialización local](../../refinamiento/inicializacion-local-db-setup.md) |
| Oblig. | `Sí` significa `NOT NULL`. |
| FK → `t` | Clave foránea hacia la tabla `t`. Si es compuesta, se indican sus columnas entre paréntesis y la otra columna figura como «parte de FK compuesta». Todas las FK usan `ON DELETE RESTRICT`. |

Tipos comunes: identificadores `uuid` con `gen_random_uuid()`; importes `numeric(14,2)`; moneda `char(3)` con `CHECK (moneda = 'ARS')`; instantes `timestamptz`; hashes `char(64)` en hexadecimal SHA-256 sobre JSON canonicalizado según RFC 8785.

## Contexto

### `entidad_comercial`

Entidad ficticia que aporta clientes, operadores o un convenio. Se carga con `db-setup`. En el Nivel 1: `BANCO_A`; `BANCO_B` es opcional. Las entidades comerciales son distintas de los cores. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador interno. | — |
| `codigo` | `text` | Sí | UNIQUE; `CHECK (codigo ~ '^[A-Z0-9_]+$')` | Código estable usado en los archivos de configuración. | R 1.1 · P |
| `nombre` | `text` | Sí | — | Nombre visible, sintético. | — |
| `creada_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

### `actor`

Identidad sintética que elige la botonera de la demo. El backend toma sus permisos de esta tabla y no de un rol enviado por el frontend. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador interno del actor. | — |
| `codigo` | `text` | Sí | UNIQUE | Código de la botonera, por ejemplo `VISITANTE`, `CLIENTE_BANCO_A`, `OPERADOR_BANCO_A`, `RESPONSABLE_CONFIG`. Es el seudónimo que exponen las consultas. | R 3.8 · P; ANX §14.2 |
| `tipo` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Papel del actor en el recorrido. | R 3.8 · P; PN-09 |
| `nombre_visible` | `text` | Sí | — | Etiqueta del botón de la demo. | — |
| `entidad_id` | `uuid` | No | FK → `entidad_comercial` | Entidad a la que pertenece un operador o un cliente de convenio. | R 1.1 · P |
| `cliente_id` | `uuid` | No | FK → `cliente`; UNIQUE | Cliente que representa un actor de tipo `CLIENTE`. | R 3.8 · P |
| `acceso` | `text` | Sí | `CHECK (acceso = 'SIMULADO')` | Cómo se obtuvo la identidad. En el Nivel 1 siempre es simulada. | R 2.6 · P; ANX §17 |
| `activo` | `boolean` | Sí | `DEFAULT true` | Si el actor puede usarse en la demo. | — |
| `creado_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

Restricciones de tabla: `CHECK ((tipo = 'CLIENTE') = (cliente_id IS NOT NULL))`; `CHECK (tipo <> 'OPERADOR' OR entidad_id IS NOT NULL)`.

### `cliente`

Persona sintética que puede ser tomador y asegurado. **Condición de la demo:** sus datos no cambian. Ningún módulo los edita en el Nivel 1 y repetir `db-setup` no los sobrescribe, porque el pedido de venta a A se reconstruye con ellos en cada reenvío ([esquema, sección 7.3](01-esquema-y-der.md#73-datos-con-que-se-reconstruye-el-pedido-a-a)). **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador interno del cliente. | — |
| `entidad_id` | `uuid` | No | FK → `entidad_comercial` | Entidad que aporta el cliente y su convenio. Define el perfil aplicable. | R 1.1 · P; PN-06 |
| `tipo_documento` | `text` | Sí | `CHECK (tipo_documento = 'DNI')` | Tipo de documento sintético. | R 1.1 · P |
| `numero_documento` | `text` | Sí | `CHECK (numero_documento ~ '^[0-9]{7,8}$')`; UNIQUE con `tipo_documento` | Número sintético. No se expone en logs ni vistas. | R 2.6 · P |
| `nombre` | `text` | Sí | — | Nombre sintético. | — |
| `apellido` | `text` | Sí | — | Apellido sintético. | — |
| `fecha_nacimiento` | `date` | No | — | Dato que puede aportar un banco ficticio. | R 1.1 · P |
| `creado_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

## Catálogo y versiones

### `producto`

Familia de producto. En el Nivel 1, una sola fila: `MOVILIDAD`. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador interno. | — |
| `codigo` | `text` | Sí | UNIQUE | Código estable de la familia. | ANX §4 |
| `nombre` | `text` | Sí | — | Nombre visible. | — |
| `creado_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

### `producto_version`

Versión funcional instalada de un producto. Contiene la ficha, la experiencia y el esquema de riesgo. Movilidad v1 y v2 se cargan con `db-setup`. **A** (inmutable)

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; UNIQUE con `producto_id` | Identificador de la versión. | — |
| `producto_id` | `uuid` | Sí | FK → `producto` | Producto al que pertenece. | — |
| `numero` | `int` | Sí | `CHECK (numero > 0)`; UNIQUE con `producto_id` | Número de versión dentro del producto. | INI · A |
| `etiqueta` | `text` | Sí | — | Nombre visible, como `v1`. | — |
| `estado_publicacion` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Estado de publicación. Es distinto de la activación, que vive en `asignacion_version`. Es el único campo modificable. | D03 §4 · P |
| `contenido` | `jsonb` | Sí | Inmutable (trigger) | Ficha de la versión: modalidades, campos, esquema de riesgo y reglas de negocio de plazo. Ver «Contenido de los campos jsonb». | ANX PR-05 · A |
| `hash_contenido` | `char(64)` | Sí | Inmutable | Hash del contenido. Si `db-setup` recibe el mismo número con otro hash, falla. | INI · A |
| `origen_archivo` | `text` | Sí | — | Ruta del archivo en Git del que se cargó. | ANX §5.1 |
| `cargada_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

### `asignacion_version`

Período durante el cual una versión atiende las nuevas cotizaciones de un producto en un canal. Activar o revertir cierra la fila activa y crea otra. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador del período. | — |
| `producto_id` | `uuid` | Sí | FK → `producto`; parte de FK compuesta | Producto asignado. | — |
| `canal` | `text` | Sí | `CHECK` (valores de canal); UNIQUE parcial con `producto_id` `WHERE activa` | Canal al que se aplica. | D03 §4 · P |
| `producto_version_id` | `uuid` | Sí | FK → `producto_version` (`producto_version_id`, `producto_id`) | Versión asignada. | ACTA · A |
| `activa` | `boolean` | Sí | — | Si el período está vigente. | D03 §4 · P |
| `activada_en` | `timestamptz` | Sí | — | Inicio del período. | — |
| `desactivada_en` | `timestamptz` | No | — | Fin del período. | — |

Restricción de tabla: `CHECK (activa = (desactivada_en IS NULL))`. La versión asignada debe estar en `APROBADA`; lo valida el módulo de versiones antes de activar.

### `asignacion_historial`

Registro de cada cambio de asignación. Solo admite inserciones. **A**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador del evento. | — |
| `producto_id` | `uuid` | Sí | FK → `producto`; parte de las dos FK compuestas | Producto afectado. | — |
| `canal` | `text` | Sí | `CHECK` (valores de canal) | Canal afectado. | — |
| `evento` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Tipo de cambio. | ACTA · A |
| `version_anterior_id` | `uuid` | No | FK → `producto_version` (`version_anterior_id`, `producto_id`) | Versión que dejó de estar asignada. | — |
| `version_nueva_id` | `uuid` | Sí | FK → `producto_version` (`version_nueva_id`, `producto_id`) | Versión que quedó asignada. | — |
| `actor_id` | `uuid` | No | FK → `actor` | Quién hizo el cambio. Vacío en la carga inicial. | ANX §18 |
| `motivo` | `text` | No | — | Motivo informado. | — |
| `ocurrido_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento del cambio. | — |

Restricciones de tabla: `CHECK ((evento = 'CARGA_INICIAL') = (actor_id IS NULL AND version_anterior_id IS NULL))`. Trigger que rechaza `UPDATE` y `DELETE`.

## Configuración comercial e integración

### `perfil_comercial`

Perfil comercial preparado. En el Nivel 1: `ESTANDAR` y `BANCO_A_CONVENIO`. **A**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; parte de FK compuesta | Identificador del perfil. | — |
| `codigo` | `text` | Sí | UNIQUE | Código estable del perfil. | D02 · A |
| `nombre` | `text` | Sí | — | Nombre visible. | — |
| `tipo` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Estándar o convenio. | D02 · A |
| `entidad_id` | `uuid` | No | FK → `entidad_comercial` | Entidad del convenio. | R 3.8 · P |
| `revision_vigente_id` | `uuid` | No | FK → `perfil_comercial_revision` (`revision_vigente_id`, `id`) | Revisión que se aplica a nuevas instancias. `db-setup` la completa en la misma transacción en que crea el perfil y su revisión, y solo si está vacía. | PN-01 |
| `creado_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

Restricción de tabla: `CHECK ((tipo = 'CONVENIO') = (entidad_id IS NOT NULL))`.

### `perfil_comercial_revision`

Revisión inmutable de un perfil. Contiene condiciones neutrales respecto del core. **A**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; UNIQUE con `perfil_id` | Identificador de la revisión. | — |
| `perfil_id` | `uuid` | Sí | FK → `perfil_comercial` | Perfil al que pertenece. | D03 §4 · P |
| `numero_revision` | `int` | Sí | `CHECK (numero_revision > 0)`; UNIQUE con `perfil_id` | Número de la revisión dentro del perfil. | D03 §4 · P; INI · A |
| `contenido` | `jsonb` | Sí | Inmutable (trigger) | Cores habilitados, opciones de suma habilitadas y canales. | D03 §4 · P |
| `hash_contenido` | `char(64)` | Sí | Inmutable | Hash del contenido, para detectar el conflicto de carga. | INI · A |
| `origen_archivo` | `text` | Sí | — | Ruta del archivo en Git. | — |
| `cargada_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

### `core`

Catálogo de cores. Agregar un core es agregar una fila, un mapeo y un adaptador. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador del core. | — |
| `codigo` | `text` | Sí | UNIQUE | `CORE_A`, `CORE_B`. Coincide con la clave del registro de adaptadores. | ANX §9 |
| `nombre` | `text` | Sí | — | Nombre visible. | — |
| `tipo_referencia` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Qué referencia de cotización entrega el core. | ANX §9 |
| `admite_venta` | `boolean` | Sí | — | Si sus ofertas se pueden confirmar. En el Nivel 1: A `true`, B `false`. | D03 §2.1 · P |
| `activo` | `boolean` | Sí | `DEFAULT true` | Si se invoca en nuevas instancias. | — |
| `creado_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

### `mapeo_core`

Mapeo de un producto hacia un core. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; parte de FK compuesta | Identificador del mapeo. | — |
| `core_id` | `uuid` | Sí | FK → `core`; UNIQUE con `producto_id` | Core destino. | D03 §4 · P |
| `producto_id` | `uuid` | Sí | FK → `producto` | Producto que se traduce. | D03 §4 · P |
| `revision_vigente_id` | `uuid` | No | FK → `mapeo_core_revision` (`revision_vigente_id`, `id`) | Revisión que el backend toma antes de registrar un intento para este producto y core. `db-setup` la completa en la misma transacción en que crea el mapeo y su revisión, y solo si está vacía. | PN-01; D03 §1 · P |
| `creado_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

### `mapeo_core_revision`

Revisión inmutable de las traducciones de un core y de sus plazos por plan. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; UNIQUE con `mapeo_id` | Identificador de la revisión. | — |
| `mapeo_id` | `uuid` | Sí | FK → `mapeo_core` | Mapeo al que pertenece. | D03 §4 · P |
| `numero_revision` | `int` | Sí | `CHECK (numero_revision > 0)`; UNIQUE con `mapeo_id` | Número de la revisión dentro del mapeo. | D03 §4 · P; INI · A |
| `contenido` | `jsonb` | Sí | Inmutable (trigger) | Traducciones de perfil, suma, provincia, modalidad y coberturas, más plazos de aceptación por plan. | D03 §3 · A; D03 §4 · P |
| `hash_contenido` | `char(64)` | Sí | Inmutable | Hash del contenido. | INI · A |
| `origen_archivo` | `text` | Sí | — | Ruta del archivo en Git. | — |
| `cargada_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de la carga. | — |

## Cotización y ofertas

### `cotizacion`

Seguimiento que ve la persona. Una cotización sigue un solo bien. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; UNIQUE con `cliente_id` | Identidad interna (`quotationId` del anexo). | ANX §9 · PR-01 |
| `numero_seguimiento` | `text` | Sí | UNIQUE; `CHECK (numero_seguimiento ~ '^M-[0-9]+$')`; secuencia `seq_numero_seguimiento` | Número público, como `M-1042`. Se conserva al recotizar. | R 4.11 · P |
| `producto_id` | `uuid` | Sí | FK → `producto` | Producto cotizado. | ACTA |
| `canal` | `text` | Sí | `CHECK` (valores de canal) | Canal en que se inició. | ACTA; R 4.12 · P |
| `actor_iniciador_id` | `uuid` | Sí | FK → `actor` | Actor que inició la cotización. | R 4.12 · P |
| `cliente_id` | `uuid` | No | FK → `cliente` | Cliente identificado. Vacío si cotiza un visitante. | D03 §1 · P |
| `creada_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento de creación. | — |

### `instancia_calculo`

Cálculo de una cotización. Conserva el riesgo y las versiones que se usaron. En el Nivel 1 hay exactamente una por cotización, con `numero = 1`. El motivo y las referencias de origen se agregan con la recotización. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; UNIQUE con `cotizacion_id` | Identidad interna del cálculo. | R 4.11 · P |
| `cotizacion_id` | `uuid` | Sí | FK → `cotizacion` | Seguimiento al que pertenece. | D03 §1 · P |
| `numero` | `int` | Sí | `CHECK (numero > 0)`; UNIQUE con `cotizacion_id` | Orden de la instancia dentro del seguimiento. Siempre 1 en el Nivel 1. | D03 §1 · P |
| `producto_version_id` | `uuid` | Sí | FK → `producto_version` | Versión de producto usada. | D03 §1 · P; ACTA |
| `perfil_revision_id` | `uuid` | Sí | FK → `perfil_comercial_revision` | Revisión de perfil usada. | D02 · A; D03 §1 · P |
| `modalidad` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Modalidad cotizada. Debe estar habilitada en la versión. | ACTA; PN-05 |
| `provincia` | `text` | Sí | — | Código de provincia de la plataforma, de la lista local de prueba. Cada adaptador lo traduce. | R 1.2 · P; PN-05 |
| `codigo_postal` | `char(4)` | Sí | `CHECK (codigo_postal ~ '^[0-9]{4}$')` | CP de cuatro cifras. La elección entre CP y CPA sigue abierta. | R 1.2 · P; R 5.15 |
| `suma_solicitada` | `numeric(14,2)` | Sí | `CHECK (suma_solicitada > 0)` | Suma asegurada pedida para daños al bien. | R 1.2 · P; PN-05 |
| `moneda` | `char(3)` | Sí | `CHECK (moneda = 'ARS')` | Moneda de la suma. | R 2.7 |
| `riesgo` | `jsonb` | Sí | Validado con el esquema de riesgo de la versión | Atributos propios de la modalidad. Ver «Contenido de los campos jsonb». | ANX PR-03; D03 §1 · P |
| `fecha_pedido` | `date` | Sí | — | Fecha que se informa a los cores que la requieren. | R 2.6 · P |
| `correlation_id` | `uuid` | Sí | — | Identificador de trazabilidad transversal. | ANX §9 |
| `creada_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento en que se fijó la configuración. | D03 §1 · P |

### `intento_core`

Llamada a un core dentro de una instancia. La estructura es la misma para cualquier core. En el Nivel 1 hay un intento por core e instancia y no hay reintentos. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; UNIQUE con `cotizacion_id` | Identidad del intento (`attemptId`). | ANX §9 |
| `instancia_id` | `uuid` | Sí | FK → `instancia_calculo` (`instancia_id`, `cotizacion_id`) | Instancia que generó el intento. | D03 §1 · P |
| `cotizacion_id` | `uuid` | Sí | Parte de FK compuesta | Copia de la cotización para las FK compuestas. | D03 §6 · P |
| `core_id` | `uuid` | Sí | FK → `core`; UNIQUE con `instancia_id` | Core invocado. Un intento por core en cada instancia. | ANX §9; D03 §1 · P |
| `mapeo_revision_id` | `uuid` | Sí | FK → `mapeo_core_revision` | Revisión de mapeo que usó el intento. El backend la toma del mapeo del producto y el core, y verifica que exista, antes de insertar el intento. La base no exige esa pertenencia. | D03 §1 · P |
| `adaptador_version` | `text` | Sí | — | Versión del código del adaptador (VER-04). | ANX §11 |
| `estado` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Estado canónico del intento. | D03 §5 · P |
| `codigo_resultado_core` | `text` | No | — | Código original devuelto por el core, como `COTIZADA` o `BUSY`. | D03 §5 · P |
| `motivo` | `text` | No | — | Motivo apto para mostrar, si el core lo informa. | R 1.4 |
| `enviado_en` | `timestamptz` | Sí | — | Inicio de la llamada. | R 5.13 |
| `finalizado_en` | `timestamptz` | No | — | Fin de la llamada o del timeout. | R 5.13 |
| `duracion_ms` | `int` | No | `CHECK (duracion_ms >= 0)` | Duración medida. | ANX §14.1 |
| `tipo_referencia` | `text` | No | `CHECK` (valores en «Valores permitidos») | Tipo de referencia obtenida. | ANX §9 |
| `referencia_externa` | `text` | No | Se completa una sola vez | Identificador del core, como el `cotizacionId` de A. Se usa para reconstruir el pedido de venta. | ANX §9; D03 §2.4 · P · CP |
| `parametros_referencia` | `jsonb` | No | — | Parámetros autorizados para volver a cotizar en un core volátil. | ANX §9; R 2.6 |
| `huella` | `char(64)` | No | — | Huella de los parámetros. No reemplaza el identificador interno. | ANX §9; R 2.6 |
| `huella_version` | `text` | No | — | Versión de la regla de normalización usada para la huella. | R 2.6 · P |
| `detalle_diagnostico` | `jsonb` | No | Sin datos personales | Respuesta inválida o error técnico, saneado. | R 5.15 · P |

Restricciones de tabla:
- `CHECK ((estado = 'PENDIENTE') = (finalizado_en IS NULL))`.
- `CHECK (tipo_referencia <> 'ID_EXTERNO' OR referencia_externa IS NOT NULL)`.
- `CHECK (tipo_referencia <> 'PARAMETROS' OR (parametros_referencia IS NOT NULL AND huella IS NOT NULL AND huella_version IS NOT NULL))`.
- `CHECK (estado <> 'CON_OFERTAS' OR tipo_referencia IS NOT NULL)`.

### `oferta`

Cada plan devuelto por un core. Es inmutable una vez registrada. **A** (vigencia por oferta) / **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK; UNIQUE con `cotizacion_id` | Identidad de la oferta (`offerId`). | ANX §9 |
| `intento_id` | `uuid` | Sí | FK → `intento_core` (`intento_id`, `cotizacion_id`) | Intento que la devolvió. | D03 §1 · P |
| `cotizacion_id` | `uuid` | Sí | Parte de FK compuesta | Copia de la cotización para las FK compuestas. | D03 §6 · P |
| `codigo_plan_core` | `text` | Sí | UNIQUE con `intento_id` | Código de plan del core, como `SA100-BASE` o `BASIC`. No identifica la cotización. | R 5.15 |
| `denominacion` | `text` | Sí | — | Nombre del plan según el core, como «Esencial». | R 1.3 |
| `premio_total` | `numeric(14,2)` | Sí | `CHECK (premio_total > 0)` | Premio a pagar informado por el core. | ANX §10; R 2.7 |
| `moneda` | `char(3)` | Sí | `CHECK (moneda = 'ARS')` | Moneda del premio. | R 2.7 |
| `base_pago` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Base sobre la que se expresa el premio. | R 1.3 |
| `periodo_cobertura_dias` | `int` | Sí | `CHECK (periodo_cobertura_dias > 0)` | Período de cobertura. Es distinto del plazo de aceptación. | R 4.10 · A |
| `estado_desglose` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Nivel de detalle del desglose recibido. | ANX §10 |
| `desglose_consistente` | `boolean` | No | — | Si la suma de componentes coincide con el total. Vacío si no se puede verificar. | R 2.7 · P |
| `recibida_en` | `timestamptz` | Sí | — | Instante en que el backend registró la respuesta del core. Desde aquí se cuentan los plazos expresados como duración. | D03 §3 · P |
| `estado_limite_core` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Si el core informó un límite válido, ninguno o uno inválido. Es inválido si no se puede interpretar o no es posterior a `recibida_en`. | D02 · A; PN-04 · CP |
| `limite_core` | `timestamptz` | No | — | Límite de aceptación válido informado por el core para este plan. Vacío si no se informó o era inválido; el valor inválido queda en el diagnóstico del intento. | D02 · A; CP |
| `limite_plataforma` | `timestamptz` | No | — | Límite calculado por la plataforma con la regla de la ficha concretada en el mapeo. | D03 §3 · A |
| `vence_efectivo` | `timestamptz` | No | Obligatorio si el origen es conocido; vacío si es `DESCONOCIDO` | Límite que se aplica: el menor límite válido. | D02 · A; D03 §3 · P |
| `origen_vencimiento` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | De dónde sale el vencimiento efectivo. | D03 §3 · A/P |

Restricciones de tabla: `CHECK ((origen_vencimiento = 'DESCONOCIDO') = (vence_efectivo IS NULL))`, las condiciones por origen de la [sección 6.3 del esquema](01-esquema-y-der.md#63-check-de-vigencia-de-la-oferta) y `CHECK ((estado_limite_core = 'INFORMADO') = (limite_core IS NOT NULL))`. La oferta no guarda si está vigente o vencida: lo calcula `v_oferta_estado`.

### `oferta_componente_premio`

Componentes del desglose informados por el core. Un componente ausente no tiene fila; uno informado en cero tiene fila con importe 0. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `oferta_id` | `uuid` | Sí | PK; FK → `oferta` | Oferta desglosada. | ANX §10 |
| `componente` | `text` | Sí | PK; `CHECK` (valores en «Valores permitidos») | Componente del premio. | ANX §10 |
| `importe` | `numeric(14,2)` | Sí | `CHECK (importe >= 0)` | Importe informado, convertido a pesos si el core usa centavos. | ANX §10; R 2.7 |
| `moneda` | `char(3)` | Sí | `CHECK (moneda = 'ARS')` | Moneda del importe. | R 2.7 |

Una oferta `SOLO_TOTAL` no tiene filas en esta tabla. Lo valida el módulo de ofertas.

### `oferta_prestacion`

Coberturas y asistencias de una oferta, con el detalle que informó el core. **PN-10**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador de la prestación. | — |
| `oferta_id` | `uuid` | Sí | FK → `oferta` | Oferta a la que pertenece. | R 1.3 |
| `orden` | `int` | Sí | UNIQUE con `oferta_id` | Orden en que el core la informó. | — |
| `tipo` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Cobertura o asistencia. | PN-10 |
| `codigo_core` | `text` | No | — | Código del core, como `DAN-BIC` o `CYCLE_DAMAGE`. Las asistencias de A no tienen código. | R 5.15 |
| `categoria_codigo` | `text` | No | Validado contra la revisión de mapeo | Categoría del comparador, como `DANOS_BIEN`, `AP_FRACTURAS`, `RESPONSABILIDAD_CIVIL` o `ASISTENCIA`. Vacía si no hay mapeo aprobado: queda como detalle sin agrupar. | R 1.3; R 5.15 |
| `descripcion` | `text` | No | — | Descripción informada. | R 1.3 |
| `estado_inclusion` | `text` | Sí | `CHECK` (valores en «Valores permitidos») | Si está incluida, excluida explícitamente o sin informar. | R 1.3; R 5.15 |
| `limite` | `numeric(14,2)` | No | `CHECK (limite > 0)` | Suma de la cobertura. | R 1.2 |
| `franquicia_tipo` | `text` | No | `CHECK` (valores en «Valores permitidos») | Forma de la franquicia. Vacío si no se informó. | R 1.3; PN-10 |
| `franquicia_importe` | `numeric(14,2)` | No | `CHECK (franquicia_importe >= 0)` | Franquicia fija. | R 1.3 |
| `franquicia_bps` | `int` | No | `CHECK (franquicia_bps BETWEEN 0 AND 10000)` | Franquicia porcentual en puntos básicos. | R 1.3; D03 §5 |
| `franquicia_base` | `text` | No | `CHECK` (valores en «Valores permitidos») | Sobre qué se aplica el porcentaje. | R 5.15 |
| `moneda` | `char(3)` | No | `CHECK (moneda = 'ARS')` | Moneda de `limite` y `franquicia_importe`. | R 2.7 |

Restricciones de tabla:
- `CHECK (estado_inclusion = 'INCLUIDA' OR (limite IS NULL AND franquicia_tipo IS NULL))`.
- `CHECK (estado_inclusion <> 'INCLUIDA' OR descripcion IS NOT NULL)`.
- `CHECK (franquicia_tipo <> 'FIJA' OR (franquicia_importe IS NOT NULL AND franquicia_bps IS NULL AND franquicia_base IS NULL))`.
- `CHECK (franquicia_tipo <> 'PORCENTUAL' OR (franquicia_bps IS NOT NULL AND franquicia_base IS NOT NULL AND franquicia_importe IS NULL))`.
- `CHECK (franquicia_tipo IS NOT NULL AND franquicia_tipo <> 'NINGUNA' OR (franquicia_importe IS NULL AND franquicia_bps IS NULL AND franquicia_base IS NULL))`.
- `CHECK ((limite IS NULL AND franquicia_importe IS NULL) OR moneda IS NOT NULL)`.
- `CHECK (tipo <> 'ASISTENCIA' OR (limite IS NULL AND franquicia_tipo IS NULL))`.

## Venta simulada

### `solicitud_venta`

Solicitud de venta simulada sobre la oferta exacta elegida. Una solicitud en `RECIBIDA` es una venta. No existe otra tabla de venta. **P**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identidad de la solicitud (`saleId` cuando está `RECIBIDA`). Es la clave de idempotencia que se envía a A. | D03 §2.2 · P |
| `cotizacion_id` | `uuid` | Sí | FK → `cotizacion` (`cotizacion_id`, `cliente_id`); parte de la FK compuesta a `oferta`; UNIQUE parcial `WHERE estado IN ('PENDIENTE_ENVIO', 'RECIBIDA', 'DESCONOCIDA')` | Seguimiento vendido. Una sola solicitud vigente por seguimiento. | D03 §2.1 · P; PN-03 |
| `oferta_id` | `uuid` | Sí | FK → `oferta` (`oferta_id`, `cotizacion_id`) | Oferta exacta elegida. | R 4.12; D03 §2.1 · P |
| `cliente_id` | `uuid` | Sí | FK → `cliente`; parte de la FK compuesta a `cotizacion` | Tomador y asegurado. Debe ser el cliente de la cotización. | R 4.12; D03 §2.1 · P |
| `actor_id` | `uuid` | Sí | FK → `actor` | Quién envió la solicitud: el cliente o el operador. | R 4.12 · P |
| `canal` | `text` | Sí | `CHECK` (valores de canal) | Canal de quien envió la solicitud. | R 4.12; PN-08 |
| `idempotency_key` | `text` | Sí | UNIQUE | Clave de idempotencia que envía el frontend. | ANX §17 · A |
| `hash_pedido` | `char(64)` | Sí | — | Hash de oferta, cliente, actor y `aceptar_hasta`. La misma clave con otro hash responde 409. | D03 §6 · P |
| `aceptar_hasta` | `timestamptz` | Sí | Inmutable | Copia del vencimiento efectivo de la oferta. Viaja a A como `aceptarHasta`. | D03 §5 · P |
| `premio_aceptado` | `numeric(14,2)` | Sí | `CHECK (premio_aceptado > 0)` | Premio que aceptó la persona. | R 2.7; PN-07 |
| `moneda` | `char(3)` | Sí | `CHECK (moneda = 'ARS')` | Moneda del premio aceptado. | R 2.7 |
| `estado` | `text` | Sí | `CHECK` (valores en «Valores permitidos»); `RECIBIDA` y `RECHAZADA` son finales | Estado de la solicitud. Cada desenlace se aplica solo si el estado actual es `PENDIENTE_ENVIO` o `DESCONOCIDA`. | D03 §2.3 · P · CP |
| `motivo_rechazo` | `text` | No | — | Motivo apto para el cliente, cuando A rechaza. | D03 §2.3 · P |
| `referencia_core` | `text` | No | — | Número de solicitud de A, como `SOL-A-0007`. | D03 §2.2 · P |
| `acuse_en` | `timestamptz` | No | — | Instante en que el backend registró el acuse. Es el instante de la venta. | D03 §2.2 · P |
| `creada_en` | `timestamptz` | Sí | `DEFAULT now()` | Momento del registro en `PENDIENTE_ENVIO`. | D03 §2.4 · P |
| `actualizada_en` | `timestamptz` | Sí | `DEFAULT now()` | Último cambio de estado. | — |

Restricciones de tabla: `CHECK ((estado = 'RECIBIDA') = (acuse_en IS NOT NULL AND referencia_core IS NOT NULL))`; `CHECK (estado <> 'RECHAZADA' OR motivo_rechazo IS NOT NULL)`. La validación de vigencia, capacidad de venta del core y coincidencia de cliente ocurre en la transacción de registro ([sección 6.7 del esquema](01-esquema-y-der.md#67-orden-transaccional-de-la-solicitud)). Los datos de la solicitud no cambian después del registro, salvo `estado`, `motivo_rechazo`, `referencia_core`, `acuse_en` y `actualizada_en`.

### `envio_solicitud`

Historial mínimo de cada envío o reenvío de una solicitud a A. Si coinciden dos reenvíos, los dos quedan registrados; un timeout posterior a la venta queda solo aquí. **PN-02**

| Campo | Tipo | Oblig. | Clave o restricción | Significado | Regla |
| --- | --- | --- | --- | --- | --- |
| `id` | `uuid` | Sí | PK | Identificador del envío. | — |
| `solicitud_id` | `uuid` | Sí | FK → `solicitud_venta` | Solicitud enviada. | D03 §2.4 · P |
| `numero` | `int` | Sí | `CHECK (numero > 0)`; UNIQUE con `solicitud_id` | Orden del envío. El primero es 1. Se asigna con la solicitud bloqueada un instante. | D03 §2.4 · P |
| `enviado_en` | `timestamptz` | Sí | — | Inicio del envío. Referencia del umbral de 5 s. | D03 §2.4 · P |
| `finalizado_en` | `timestamptz` | No | Se completa una sola vez, con `UPDATE … WHERE finalizado_en IS NULL` | Fin del envío. Vacío si el proceso cayó durante el envío. | D03 §2.4 · P |
| `resultado` | `text` | No | `CHECK` (valores en «Valores permitidos») | Desenlace de este envío, aunque la solicitud ya esté en un estado final. | D03 §2.4 · P · CP |

Restricción de tabla: `CHECK ((resultado IS NULL) = (finalizado_en IS NULL))`.

## Valores permitidos

| Campo | Valores | Significado | Regla |
| --- | --- | --- | --- |
| `actor.tipo` | `VISITANTE`, `CLIENTE`, `OPERADOR`, `RESPONSABLE_CONFIGURACION` | Visitante anónimo, cliente identificado, operador de una entidad y responsable de configuración (`product-manager` del anexo). | R 3.8; PN-09 |
| `actor.acceso` | `SIMULADO` | Identidad de demo. `OIDC` se agrega en el Nivel 2. | ANX §17 |
| `canal` (en `asignacion_version`, `asignacion_historial`, `cotizacion`, `solicitud_venta`) | `AUTOASISTIDO`, `ASISTIDO` | Cliente por cuenta propia u operador por cuenta del cliente. Equivalen a `customer` y `operator` del anexo §8. | R 4.12; D03 §4 |
| `cliente.tipo_documento` | `DNI` | Documento sintético. | R 1.1 |
| `producto_version.estado_publicacion` | `APROBADA`, `RETIRADA`; `BORRADOR` desde el Nivel 2 | Publicación. La activación no es un estado de la versión. | D03 §4 |
| `asignacion_historial.evento` | `CARGA_INICIAL`, `ACTIVACION`, `ROLLBACK` | Carga desde `db-setup`, activación de otra versión y vuelta a una versión anterior. | ACTA; INI |
| `perfil_comercial.tipo` | `ESTANDAR`, `CONVENIO` | Perfil general o de una entidad. | D02 |
| `core.tipo_referencia`, `intento_core.tipo_referencia` | `ID_EXTERNO`, `PARAMETROS`, `NO_RECUPERABLE` | Equivalen a `external-id`, `parameters` y `not-recoverable` del anexo §9. | ANX §9 |
| `instancia_calculo.modalidad` | `BICICLETA`, `BICICLETA_ELECTRICA` | v1 admite solo `BICICLETA`; v2 admite las dos. | ANX §4 |
| `intento_core.estado` | `PENDIENTE`, `CON_OFERTAS`, `SIN_OFERTA`, `RECHAZO_NEGOCIO`, `DATOS_INVALIDOS`, `NO_SOPORTADO`, `FALLA_TEMPORAL`, `TIMEOUT`, `RESPUESTA_INVALIDA`, `ERROR_TECNICO` | Estado canónico. `PENDIENTE` dura mientras la llamada está en curso, hasta 2 s; queda solo si el proceso cae. `TIMEOUT` se registra al vencer los 2 s. Las equivalencias con los códigos de A y B están en D03 §5. | D03 §1; D03 §5 · CP |
| `oferta.base_pago` | `MENSUAL`, `POR_PERIODO` | A usa los mismos nombres; B usa `MONTH` y `TERM`. | R 1.3; R 5.15 |
| `oferta.estado_desglose` | `COMPLETO`, `PARCIAL`, `SOLO_TOTAL` | `BreakdownStatus` del anexo. | ANX §10 |
| `oferta.estado_limite_core` | `INFORMADO`, `AUSENTE`, `INVALIDO` | El core informó un límite válido, no informó o informó un dato inválido. | D02; PN-04 |
| `oferta.origen_vencimiento` | `CORE`, `PLATAFORMA`, `AJUSTADO_A_CORE`, `DESCONOCIDO` | Límite del core, límite de plataforma igual o anterior al del core, plataforma recortada al core (genera aviso) y sin límite válido (no confirmable). | D03 §3 |
| `oferta_componente_premio.componente` | `PRIMA_PURA`, `PRIMA_TARIFA`, `GASTOS_PRODUCCION`, `GASTOS_EXPLOTACION`, `RECARGOS`, `IMPUESTOS`, `SELLADOS`, `PERCEPCIONES`, `CARGOS_FINANCIEROS` | Componentes del anexo §10. El premio total está en `oferta.premio_total`. | ANX §10 |
| `oferta_prestacion.tipo` | `COBERTURA`, `ASISTENCIA` | Cobertura con suma o servicio de asistencia. | PN-10 |
| `oferta_prestacion.estado_inclusion` | `INCLUIDA`, `NO_INCLUIDA`, `NO_INFORMADA` | Se muestran como incluida, «No incluida» y «No informado». | R 1.3; R 5.15 |
| `oferta_prestacion.franquicia_tipo` | `FIJA`, `PORCENTUAL`, `NINGUNA` | Importe fijo, porcentaje o «Sin franquicia» explícito. Vacío si no se informó. | R 1.3; PN-10 |
| `oferta_prestacion.franquicia_base` | `DANO`, `LIMITE_COBERTURA` | Equivalen a `LOSS` y `COVERAGE_LIMIT` de B. | R 5.15 |
| `solicitud_venta.estado` | `PENDIENTE_ENVIO`, `RECIBIDA`, `RECHAZADA`, `DESCONOCIDA` | Ver D03 §2.3. Solo `RECIBIDA` cuenta como venta. `RECIBIDA` y `RECHAZADA` son finales. `RECHAZADA` no bloquea otra solicitud. | D03 §2.3 |
| `envio_solicitud.resultado` | `RECIBIDA`, `RECHAZADA`, `CONFLICTO`, `TIMEOUT`, `ERROR_TECNICO` | Respuesta de A, conflicto de clave, timeout de 3 s o corte de conexión. | D03 §2.4; D03 §5 |

## Contenido de los campos jsonb

Los campos `jsonb` se validan con JSON Schema al cargarse o al recibirse. Su estructura exacta se aprueba con la ficha de cada versión.

| Campo | Contenido previsto | Validación | Regla |
| --- | --- | --- | --- |
| `producto_version.contenido` | Modalidades habilitadas, campos y etiquetas del formulario, esquema de riesgo (JSON Schema) y reglas de negocio de plazo de la ficha. | `db-setup`, contra el esquema de fichas. | ANX PR-05; D03 §3 |
| `perfil_comercial_revision.contenido` | Cores habilitados, opciones de suma habilitadas y canales en que aplica el perfil. | `db-setup`. | D03 §4 |
| `mapeo_core_revision.contenido` | Perfil a productor o descuento; opción de suma a código de plan o monto; provincia, modalidad y coberturas a códigos del core; categoría de cada cobertura; plazo de aceptación por plan. | `db-setup`. | D03 §3, §4 |
| `instancia_calculo.riesgo` | v1: marca, modelo opcional, valor declarado e identificador sintético opcional del bien. v2 agrega, para `BICICLETA_ELECTRICA`, la potencia del motor y los datos de batería. El nombre, la unidad y el umbral de la potencia se aprueban con la ficha. | Backend, con el esquema de riesgo de la versión usada. | ANX §4; D03 §5; R 5.14 |
| `intento_core.parametros_referencia` | Parámetros autorizados que hacen falta para volver a cotizar en un core volátil, sin datos personales. | Adaptador del core. | ANX §9; R 2.6 |
| `intento_core.detalle_diagnostico` | Código, mensaje y fragmento saneado de una respuesta inválida o de un error técnico. | Adaptador del core. | R 5.15 |

## Vistas

### `v_oferta_estado`

Estado actual de cada oferta. **P**

| Columna | Tipo | Significado |
| --- | --- | --- |
| `oferta_id` | `uuid` | Oferta. |
| `cotizacion_id` | `uuid` | Seguimiento. |
| `vigente` | `boolean` | `vence_efectivo > now()`. Falso si el origen es `DESCONOCIDO`. |
| `vendible` | `boolean` | Vigente, con origen distinto de `DESCONOCIDO` y core con `admite_venta`. |

### `v_solicitud_sin_resolver`

Solicitudes que necesitan verificación. El umbral se lee de configuración (5 s). **P**

| Columna | Tipo | Significado |
| --- | --- | --- |
| `solicitud_id` | `uuid` | Solicitud. |
| `numero_seguimiento` | `text` | Seguimiento bloqueado. |
| `estado` | `text` | `DESCONOCIDA` o `PENDIENTE_ENVIO`. |
| `motivo_visibilidad` | `text` | `DESCONOCIDA` o `ENVIO_SIN_CONFIRMAR`. |
| `desde` | `timestamptz` | Último `enviado_en` o, si no hubo envíos, `creada_en`. |
| `envios` | `int` | Cantidad de envíos registrados. |

### `v_aviso_plazo_ajustado`

Ofertas cuyo límite de plataforma superó al del core. Es la consulta del responsable de configuración. **P**

| Columna | Tipo | Significado |
| --- | --- | --- |
| `oferta_id` | `uuid` | Oferta afectada. |
| `numero_seguimiento` | `text` | Seguimiento. |
| `core_codigo` | `text` | Core. |
| `codigo_plan_core` | `text` | Plan. |
| `mapeo_revision_id` | `uuid` | Revisión de mapeo que concretó la regla. |
| `producto_version_id` | `uuid` | Versión de producto con la ficha que respalda la regla. |
| `perfil_revision_id` | `uuid` | Revisión de perfil aplicada. |
| `limite_plataforma` | `timestamptz` | Límite según la regla de la plataforma. |
| `limite_core` | `timestamptz` | Límite informado por el core. |
| `vence_efectivo` | `timestamptz` | Límite aplicado. |
| `recibida_en` | `timestamptz` | Momento del hecho. |

### `v_evento_negocio`

Una fila por hecho de negocio, con los campos del `BusinessEvent` del anexo §14.2. Reemplaza la tabla de eventos en el Nivel 1. **P**

| Columna | Tipo | Significado |
| --- | --- | --- |
| `tipo_evento` | `text` | `COTIZACION_CREADA`, `OFERTA_RECIBIDA`, `INTENTO_FALLIDO`, `SOLICITUD_REGISTRADA`, `VENTA_RECIBIDA` o `SOLICITUD_RECHAZADA`. |
| `ocurrido_en` | `timestamptz` | Momento del hecho. Para `VENTA_RECIBIDA`, `acuse_en`. |
| `producto_codigo` | `text` | Producto. |
| `producto_version` | `text` | Etiqueta de la versión usada. |
| `modalidad` | `text` | Modalidad. |
| `core_codigo` | `text` | Core, cuando aplica. |
| `adaptador_version` | `text` | Versión del adaptador, cuando aplica. |
| `canal` | `text` | Canal. |
| `actor_tipo` | `text` | Tipo de actor. |
| `actor_codigo` | `text` | Código seudónimo del actor. |
| `cotizacion_id` | `uuid` | Cotización. |
| `oferta_id` | `uuid` | Oferta, cuando aplica. |
| `solicitud_id` | `uuid` | Solicitud, cuando aplica. |
| `importe` | `numeric(14,2)` | Premio de la oferta o premio aceptado. |
| `moneda` | `char(3)` | Moneda. |
| `resultado` | `text` | Estado del intento o de la solicitud. |
