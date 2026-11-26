# Decisiones del DER sobre cotización, venta, vigencia, asignación y contratos

Segunda entrega del TPI. **Estado: borrador para aprobación.** Este registro reúne lo que falta resolver para dibujar el DER, completar el diccionario de datos y cerrar el listado de módulos del Nivel 1. Continúa las [decisiones de perfiles y vigencia](decisiones-der-02-perfil-y-vigencia.md) y se apoya en [versiones en curso](03-09-versiones-en-curso-y-recuperacion.md), [vencimiento](04-10-vencimiento-y-recotizacion-de-ofertas.md), [seguimiento e instancias](04-11-referencia-publica-e-instancias-de-cotizacion.md), [atribución de la venta](04-12-atribucion-de-venta-y-estado-de-solicitud.md), [cupos y exclusividad](04-12-cupos-y-exclusividad-de-venta.md), [inicialización local](inicializacion-local-db-setup.md) y los [contratos de los cores](05-15-segunda-lectura-de-fichas-y-cores.md).

Las secciones 1 a 5 describen reglas de negocio. La sección 6 reúne las propuestas de implementación en PostgreSQL. La sección 8 lista las decisiones que hay que aprobar.

| Marca | Significado |
| --- | --- |
| **Aprobado** | Ya está registrado en un documento aprobado o se acordó en esta revisión. El DER lo usa sin otra lectura. |
| **Propuesta N1** | Se propone para el Nivel 1. Hay que aprobarlo antes de dibujar el DER. |
| **Diferido** | Queda fuera del Nivel 1. Se indica qué dato o relación se conserva desde ahora. |

## 1 Cotización e instancia de cálculo

La cotización es lo que la persona sigue con su número público. La instancia de cálculo registra lo que efectivamente se usó en cada cálculo. Esto precisa la [decisión 2](decisiones-der-02-perfil-y-vigencia.md): la revisión de perfil aplicada se conserva en la **instancia**, que pertenece a la cotización. Así una recotización puede aplicar otra revisión sin cambiar el número de seguimiento.

| Entidad | Qué conserva | Marca |
| --- | --- | --- |
| Cotización | Número público de seguimiento, cliente cuando está identificado, actor que la inició, canal y fecha de creación. | Propuesta N1 |
| Instancia de cálculo | Datos de riesgo usados en ese cálculo, versión de producto, revisión de perfil, fecha del pedido y `correlationId`. El motivo y las referencias de origen solo sirven para recotizar y se agregan con la recotización. | Propuesta N1 |
| Intento de core | Core, revisión de mapeo, versión del adaptador, estado, tiempos y referencia del core, si la hay. | Propuesta N1 |
| Oferta | Plan, premio, moneda, base de pago, desglose disponible, coberturas y vigencia según la sección 3. | Aprobado en lo que respecta a la vigencia por oferta |

Reglas de negocio propuestas para el Nivel 1:

- **Una instancia por cotización en el Nivel 1.** La recotización guiada sigue siendo candidata, según el 4.10. El modelo admite varias instancias bajo el mismo número, aunque el primer recorrido cree una sola.
- **Un intento por core en el Nivel 1.** Cada instancia registra un intento por core habilitado y su resultado, aunque sea sin oferta, rechazo, timeout o falla. Los reintentos de cotización pasan al Nivel 2: ante una falla, la persona inicia una cotización nueva. Cuando se incorporen, un reintento agregará un intento a la misma instancia y una recotización creará otra instancia bajo el mismo número.
- **Respuesta sincrónica de la cotización.** En el Nivel 1 el backend llama a los cores en paralelo y responde cuando todos los intentos terminaron o vencieron. Se propone un timeout de cotización de 2 segundos por core, ajustable después de medir. La publicación progresiva de ofertas y los avisos por ofertas tardías pasan al Nivel 2.
- **Revisión de mapeo validada antes del intento.** Antes de registrar cada intento, el backend resuelve la revisión vigente del mapeo del producto y el core, y verifica que exista y pertenezca a ese par. Si falta, ese core no se invoca y se registra el motivo. El intento conserva la revisión que usó. Esta validación no agrega tablas ni restricciones.
- **Formulario abierto con configuración cambiada.** La configuración pública que recibe el frontend identifica la versión de producto y la revisión de perfil con las que se armó. Al crear la cotización, el backend vuelve a resolver **ambas** para el contexto del actor. Si cualquiera de las dos cambió, no crea la cotización: devuelve la configuración vigente y el frontend pide revisar los datos, conservando los que sigan siendo válidos. La configuración queda fijada al crear la instancia. Esto responde la primera pregunta del 3.9.
- **Cliente opcional al cotizar.** Una cotización puede empezar sin cliente identificado. La solicitud de venta exige un cliente (sección 2).

## 2 Venta simulada

### 2.1 Reglas para aprobar

| Regla | Contenido propuesto | Marca |
| --- | --- | --- |
| Una tabla de solicitudes | La venta simulada es una **solicitud de venta** que recibió acuse. No existe una entidad de venta separada. Los reportes cuentan las solicitudes en estado `RECIBIDA`. | Propuesta N1 |
| Una bicicleta por cotización | Todas las instancias de una cotización se refieren al mismo bien. Cotizar otra bicicleta o cambiar el bien abre otra cotización con otro número. | Propuesta N1 |
| Una venta exitosa por seguimiento | Mientras una solicitud del seguimiento esté recibida o sin resolver, no se acepta otra, aunque sea para otra oferta del mismo número. Una solicitud rechazada deja confirmar de nuevo. | Propuesta N1 |
| A vende, B compara | Las ofertas de B se muestran y comparan. Confirmar una oferta de B responde que esa oferta no está habilitada para venta en el Nivel 1. Es una limitación declarada: el anexo exige recotizar B antes de una operación posterior (§9) y ubica esa recotización en el Nivel 3 (§19). | Propuesta N1 |
| Cliente identificado | Confirmar exige un cliente identificado. En el Nivel 1 el visitante compara pero no confirma; para vender se usa una identidad de cliente de la botonera. | Propuesta N1 |
| Oferta vigente | La solicitud solo se registra si la oferta sigue vigente según su vencimiento efectivo (sección 3) en el momento en que el backend la registra. | Aprobado (4.10) |

### 2.2 Acuse que registra la venta

Como propuesta para el Nivel 1, el simulador de Core A expone una operación de **solicitud** que recibe el identificador de la solicitud como clave de idempotencia. La respuesta `RECIBIDA`, con un número de solicitud del core, es el acuse. Al recibirlo, el backend pasa la solicitud a `RECIBIDA` y guarda el instante en que registró el acuse. Ese instante es el que cuenta en los reportes de venta. La respuesta `RECHAZADA`, con su motivo, cierra la solicitud sin venta. La cobertura provisional del 4.12 queda **diferida**.

### 2.3 Estados de la solicitud

| Estado | Cuándo se aplica | Cuenta como venta | Bloquea otra solicitud del seguimiento | Lo que ve la persona |
| --- | --- | --- | --- | --- |
| `PENDIENTE_ENVIO` | La solicitud quedó registrada y todavía no hay acuse ni respuesta. | No | Sí | «Estamos procesando tu solicitud» |
| `RECIBIDA` | El core devolvió el acuse. | Sí | Sí | «Solicitud recibida» y el número de solicitud |
| `RECHAZADA` | El core rechazó la solicitud de forma definitiva. | No | No | «Solicitud rechazada» y el motivo apto para el cliente |
| `DESCONOCIDA` | Se envió la solicitud y no hubo una respuesta concluyente. La solicitud pasa a este estado en el momento en que vence el timeout de envío a A o se corta la conexión. | No | Sí | «Estamos verificando el estado de tu solicitud» |

**`RECIBIDA` y `RECHAZADA` son estados finales.** Una vez alcanzados, ningún desenlace posterior los cambia. Un timeout o un error que llega después solo queda registrado en el historial del envío. Las transiciones permitidas son: de `PENDIENTE_ENVIO` o `DESCONOCIDA` a `RECIBIDA`, `RECHAZADA` o `DESCONOCIDA`. **Propuesta N1.**

### 2.4 Caída del proceso después de guardar `PENDIENTE_ENVIO`

Si el proceso cae después de registrar `PENDIENTE_ENVIO`, la plataforma no sabe si la solicitud llegó al core. Propuesta para el Nivel 1:

| Parámetro | Valor inicial de la demo | Efecto | Marca |
| --- | --- | --- | --- |
| Timeout de envío a A | 3 segundos | Al vencer, la solicitud pasa a `DESCONOCIDA` en ese momento. | Propuesta N1 |
| Umbral de «envío sin confirmar» | 5 segundos desde el último envío o, si no hubo envío, desde la creación | Hace visible una solicitud que quedó en `PENDIENTE_ENVIO` después de una caída. | Propuesta N1 |

Los dos valores son iniciales y se ajustan después de medir. El timeout explícito ya lleva a `DESCONOCIDA`, así que el umbral solo cubre el caso en que el proceso cayó antes de registrar un desenlace.

1. **Sigue bloqueando.** La solicitud se mantiene en `PENDIENTE_ENVIO` y cuenta como sin resolver. No se puede crear otra solicitud para el mismo seguimiento.
2. **Se vuelve visible.** Si sigue en `PENDIENTE_ENVIO` más allá del umbral, la consulta de solicitudes la muestra como **envío sin confirmar**, junto con las `DESCONOCIDA`. La persona ve el mensaje de verificación. No hace falta otro estado: la condición surge del estado y del tiempo transcurrido. **El umbral no libera el seguimiento ni dispara un reenvío automático.**
3. **Se resuelve reenviando la misma solicitud.** Una solicitud `DESCONOCIDA`, o una en `PENDIENTE_ENVIO` que superó el umbral, se resuelve volviendo a enviar **la misma solicitud con la misma clave y los mismos datos**. Solo es seguro porque el contrato del simulador A garantiza que una clave repetida devuelve el resultado original sin crear otra solicitud (sección 5). El reenvío se dispara al repetir la confirmación con la misma clave de idempotencia o con una acción **Verificar estado** disponible para el operador o el responsable. Cada reenvío queda registrado. Nunca se crea otra solicitud.
4. **El reenvío no vuelve a validar la vigencia en la plataforma.** La vigencia se validó al registrar la solicitud. Al reenviar, decide A: si ya había aceptado la clave, devuelve el acuse original; si nunca la recibió, valida el plazo con el límite que viaja en la solicitud (sección 5). Así una oferta vencida después del primer envío no impide recuperar una venta que A aceptó a tiempo, y tampoco permite registrar una venta nueva fuera de plazo.
5. **Reenvíos concurrentes.** Si coinciden dos reenvíos de la misma solicitud, por ejemplo la persona repite la confirmación mientras el operador pulsa **Verificar estado**, los dos quedan en el historial de envíos. Si uno devuelve `RECIBIDA` y el otro termina en timeout, la solicitud queda `RECIBIDA`, el timeout se registra solo en su envío y la venta se cuenta una vez, sin importar el orden en que terminen.
6. **El pedido se reconstruye con datos inmutables.** Cada envío arma el mismo pedido a partir de: el identificador de la solicitud (clave de idempotencia) y `aceptar_hasta`; el código de plan de la oferta; el `cotizacionId` de A guardado en el intento; y los datos sintéticos del cliente. Solicitud, oferta e intento no cambian después de registrados. **Condición de la demo:** los datos sintéticos del cliente se mantienen fijos; ningún módulo los edita en el Nivel 1 y repetir `db-setup` no los sobrescribe. Si esa condición se rompiera, A respondería conflicto y la solicitud quedaría para diagnóstico.
7. **Diferido.** Consultar el estado en el core sin reenviar y resolver automáticamente las solicitudes sin respuesta quedan para el Nivel 2, como indica el [alcance por niveles](alcance-por-niveles-y-bases-de-extension.md).

## 3 Vigencia de la oferta y aviso al responsable

| Regla | Marca |
| --- | --- |
| El plazo de aceptación pertenece a cada oferta y es distinto del período de cobertura. | Aprobado ([decisión 3](decisiones-der-02-perfil-y-vigencia.md)) |
| Se aplica el menor límite disponible entre el de la plataforma y el del core. | Aprobado |
| Si la regla de la plataforma supera el límite del core, se aplica el del core y se avisa al responsable de configuración. | Aprobado |
| La ficha o su anexo respaldan la regla de negocio del plazo. El mapeo de cada core puede concretarla por core y plan, porque los planes son códigos propios de cada core. | Aprobado |
| La oferta conserva el límite de la plataforma, el límite del core, el vencimiento efectivo y su origen. Como el límite propio puede venir de la ficha a través del mapeo, el diccionario lo llama **límite de plataforma**. | Aprobado; el nombre es una propuesta |

Propuestas para el Nivel 1:

- **Instante desde el que se cuenta una duración.** Un plazo expresado como duración se cuenta desde el instante en que el backend registra la respuesta del core que trae la oferta. Es el primer momento en que la plataforma conoce la oferta, no depende del reloj del core y se aplica igual a A y a B. Si el core informa un instante absoluto, se usa tal cual.
- **Origen del vencimiento.** Los valores posibles son: informado por el core, límite de plataforma, ajustado al core (dispara el aviso) y desconocido.
- **Sin un límite válido.** Si faltan los dos límites, el origen queda como desconocido y el vencimiento efectivo queda vacío. La oferta se puede ver y comparar, pero no confirmar. Esto cierra el caso que la decisión 3 dejaba pendiente.
- **Límite del core inválido aunque exista uno de plataforma.** Un límite del core es inválido si no se puede interpretar como instante o no es posterior al momento en que el backend registró la respuesta. Como el límite del core es el máximo que la plataforma puede reconocer, un dato inválido impide saber si el límite de plataforma lo respeta. Se propone tratarlo como origen desconocido: la oferta se compara, no se confirma y el intento conserva el detalle para diagnóstico. Depende del contrato preliminar de A, que todavía no tiene la vigencia por plan.
- **Cómo ve el aviso el responsable.** En el Nivel 1, el aviso es una **consulta** disponible para la identidad de responsable de configuración de la botonera. Lista las ofertas cuyo límite se ajustó al del core, con oferta, plan, core, revisión de mapeo, revisión de perfil, ambos límites, valor efectivo y momento del hecho. No se envía por ningún canal externo. Se verifica con un caso sintético en el que la regla de la plataforma supera el límite que informa A. El envío por correo u otro canal y el seguimiento del aviso (visto o resuelto) quedan **diferidos**.

## 4 Asignación de versiones y revisiones de configuración

| Regla | Marca |
| --- | --- |
| Hay una asignación activa por **producto y canal**. Los canales son autoasistido y asistido, que corresponden a `customer` y `operator` del anexo. «Activar v2 para operador» (DEMO-07) es activarla en el canal asistido. | Propuesta N1 |
| La audiencia como parte del alcance de la asignación (§13.2 del anexo) queda para el Nivel 3, junto con las reglas por rol y porcentaje. | Diferido |
| La activación y el rollback cambian la asignación y dejan historial. No modifican ni borran versiones. | Aprobado (acta y anexo §11) |
| El estado de publicación de una versión es distinto de su activación. «Activa» es una propiedad de la asignación, no de la versión. El ciclo de publicación completo es del Nivel 2. | Propuesta N1 |
| Una versión o revisión publicada no se modifica. Un cambio genera otra revisión. | Aprobado (anexo PR-06) |
| Cada revisión se identifica por su número **dentro del perfil o del mapeo al que pertenece**. Por ejemplo, perfil `ESTANDAR` revisión 1 o mapeo `CORE_A` de Movilidad revisión 2. Cada mapeo pertenece a un core y a un producto. | Propuesta N1 |
| La revisión de perfil contiene condiciones neutrales respecto del core: cores habilitados, opciones de suma habilitadas y canal. La revisión de mapeo contiene las traducciones de cada core: perfil a productor o descuento, suma a código de plan o monto, provincia, modalidad, cobertura a categoría y plazos por plan. | Propuesta N1 |
| `db-setup`: si llega el mismo contenido, se reutiliza la revisión existente. Si llega otro contenido con la misma identificación, la carga falla con un conflicto y el cambio requiere otra revisión. | Aprobado ([inicialización local](inicializacion-local-db-setup.md)) |
| `db-setup` crea la asignación inicial solo si no existe ninguna para ese producto y canal. Repetir la carga nunca reactiva una versión ni modifica la asignación vigente. | Propuesta N1; precisa «donde hagan falta» |

## 5 Ajustes de los contratos de los cores

Los campos, las unidades y los umbrales concretos se aprueban junto con la ficha. Esta sección registra qué hay que cambiar en los [esquemas preliminares](contratos/). Todas las reglas que dependen de estos ajustes quedan condicionadas a que se cierren los contratos; la entrega 2 las marca como dependientes de un contrato preliminar.

| Ajuste | Core | Cambio | Marca |
| --- | --- | --- | --- |
| Vigencia por plan | A | Mover `validaHasta` de la respuesta a cada plan. Se propone eliminar el límite general. Si se conserva, el límite de un plan no puede superarlo. | Propuesta N1 |
| Códigos de cotización y de consulta separados | A | Separar la respuesta de cotización de la respuesta de consulta. `COTIZACION_NO_ENCONTRADA` y `COTIZACION_VENCIDA` se usan solo en la consulta. | Propuesta N1 |
| Rechazo de negocio distinto de «sin oferta» | A | Agregar `RECHAZADA` para un riesgo no elegible según una regla documentada. `SIN_OFERTA` queda para cuando no hay planes para esos parámetros. | Propuesta N1 |
| Rechazo de negocio distinto de «sin oferta» | B | B ya tiene `NOT_ELIGIBLE`, pero no puede expresar «sin oferta» sin rechazar, porque `options` exige al menos un elemento. Agregar un resultado `NO_OPTIONS`. | Propuesta N1 |
| Dato de bicicleta eléctrica que un core usa | B | Para `E_CYCLE`, B recibe la potencia del motor y responde no elegible o no soportado por encima de un umbral. A sigue recibiendo solo `BIC-E01`. El campo, la unidad y el umbral quedan para aprobar con la ficha de v2. | Propuesta N1 |
| Operación de solicitud | A | Nueva operación con clave de idempotencia, referencia a la oferta (`cotizacionId` e `idPlan`), límite de aceptación (`aceptarHasta`, igual al vencimiento efectivo de la oferta) y datos sintéticos del tomador. Responde `RECIBIDA` con número de solicitud o `RECHAZADA` con motivo, incluido «oferta vencida». Garantías: una clave repetida con los mismos datos devuelve la respuesta original, aunque A se haya reiniciado; una clave repetida con otros datos responde conflicto. B no tiene operación de solicitud en el Nivel 1. | Propuesta N1 |
| Franquicia porcentual en enteros | B | `deductiblePct` es `number`. Pasarlo a puntos básicos enteros, igual que `discountBps`. | Diferido |
| Desglose parcial | B | Ningún ejemplo usa todavía el estado parcial. | Diferido |

### Casos de aceptación de la operación de solicitud de A

Estos casos deben pasar antes de cerrar el contrato de A. Se verifican contra el simulador y contra el backend. **Propuesta N1.**

| Caso | Situación | Resultado esperado |
| --- | --- | --- |
| 1 · Acuse perdido | A recibió la solicitud y la aceptó, pero el acuse no llegó al backend. La solicitud queda en `DESCONOCIDA`, o en `PENDIENTE_ENVIO` si el backend cayó. | Al reenviar la misma clave con los mismos datos, A devuelve el **mismo número de solicitud**. El backend pasa la solicitud a `RECIBIDA` y se cuenta una sola venta. |
| 1b · Acuse perdido con reinicio de A | Igual al caso 1, pero A se reinicia entre el primer envío y el reenvío. Se aplica si la demo incluye reiniciar A. | El resultado del caso 1 sobrevive al reinicio: A devuelve el mismo número de solicitud. |
| 2a · Nunca recibida y oferta vencida | A nunca recibió la solicitud. Cuando llega el reenvío, ya pasó `aceptarHasta`. | A responde `RECHAZADA` con motivo «oferta vencida». El backend pasa la solicitud a `RECHAZADA` y **no registra una venta**. El seguimiento queda libre para cotizar de nuevo. |
| 2b · Aceptada a tiempo y vencida después | A aceptó la solicitud antes de `aceptarHasta` y el acuse se perdió. Cuando llega el reenvío, la oferta ya venció. | A devuelve el **acuse original**, sin volver a validar el plazo. El backend pasa la solicitud a `RECIBIDA` con el número original. |
| 3 · Reenvíos concurrentes | La solicitud está en `DESCONOCIDA` y coinciden dos reenvíos con la misma clave. Uno recibe `RECIBIDA` y el otro termina en timeout, en cualquier orden. | La solicitud queda `RECIBIDA` con el número original. El timeout queda registrado solo en su envío. La venta se cuenta una sola vez. |

Consecuencias para el simulador y los módulos:

- **Registro de claves persistente.** Para el caso 1b, el simulador de A guarda sus claves de idempotencia y sus respuestas en un almacenamiento propio que sobrevive al reinicio, separado de la base del intermediario. Se propone un esquema propio `core_a` en el mismo servidor PostgreSQL de la demo, o un volumen de Docker. En el Nivel 1 las claves no vencen.
- **A decide el plazo al recibir.** A compara el instante de recepción con el menor entre su propio límite del plan y `aceptarHasta`. La plataforma no vuelve a validar la vigencia al reenviar (2.4).
- **Huella del pedido.** El hash de la solicitud en el backend incluye `aceptarHasta`, y el pedido a A se reconstruye siempre con los datos inmutables de 2.4, así el reenvío lleva los mismos datos que el primer envío.

Equivalencias propuestas entre los resultados de cada core y el estado canónico del intento. El diccionario usa la columna canónica.

| Estado canónico | Core A | Core B | Significado |
| --- | --- | --- | --- |
| `PENDIENTE` | — | — | El intento se envió y todavía no hay respuesta. |
| `CON_OFERTAS` | `COTIZADA` | `OK` | Llegó al menos una oferta válida. |
| `SIN_OFERTA` | `SIN_OFERTA` | `NO_OPTIONS` (nuevo) | No hay planes para esos parámetros. |
| `RECHAZO_NEGOCIO` | `RECHAZADA` (nuevo) | `NOT_ELIGIBLE` | El riesgo no es elegible según una regla documentada. |
| `DATOS_INVALIDOS` | `DATOS_INVALIDOS` | `BAD_RISK_DATA` | El core rechazó los datos enviados. |
| `NO_SOPORTADO` | `PRODUCTO_NO_SOPORTADO` | `ASSET_UNSUPPORTED` | El core no cotiza esa modalidad o producto. |
| `FALLA_TEMPORAL` | `FALLA_TEMPORAL` | `BUSY` | Falla transitoria informada por el core. |
| `TIMEOUT` | — | — | El backend cortó la espera sin respuesta. |
| `RESPUESTA_INVALIDA` | Código desconocido o premio inválido | Código desconocido o premio inválido | Se conserva para diagnóstico y no genera oferta. |
| `ERROR_TECNICO` | Conexión o error del servidor | Conexión o error del servidor | Falla técnica sin respuesta interpretable. |

## 6 Propuestas de implementación en PostgreSQL

Todo lo de esta sección es **Propuesta N1**. Implementa las reglas de las secciones 1 a 5 y se puede cambiar sin alterarlas.

- **Versión y tipos.** PostgreSQL 16. Los estados se modelan como `text` con `CHECK` en lugar de tipos `enum`, para simplificar las migraciones. Los importes usan `numeric(14,2)` con moneda `char(3)` y `CHECK (moneda = 'ARS')`.
- **Claves de revisión.** `UNIQUE (perfil_id, numero_revision)` y `UNIQUE (mapeo_id, numero_revision)`. Cada revisión guarda `hash_contenido`, un SHA-256 del JSON canonicalizado según [RFC 8785](https://www.rfc-editor.org/info/rfc8785/). El seed usa `INSERT … ON CONFLICT DO NOTHING` y, si la fila ya existía, compara el hash. Si difiere, `db-setup` termina con error.
- **Inmutabilidad.** Un trigger `BEFORE UPDATE OR DELETE` rechaza cambios en las revisiones y en el contenido de `producto_version`. Las FK que apuntan a revisiones y versiones usan `ON DELETE RESTRICT`.
- **Asignación.** `UNIQUE (producto_id, canal) WHERE activa`. Activar o revertir ocurre en una transacción que desactiva la fila anterior, activa la nueva y agrega una fila a `asignacion_historial`, que solo admite inserciones. Una activación concurrente choca con el índice y se responde 409. Si en el Nivel 3 se agrega una audiencia nullable, el índice necesita `NULLS NOT DISTINCT`.
- **Coherencia del seguimiento.** FK compuestas en cadena para que una solicitud solo pueda apuntar a una oferta de su misma cotización:
  - `instancia_calculo` declara `UNIQUE (id, cotizacion_id)`;
  - `intento_core` referencia `(instancia_id, cotizacion_id)`;
  - `oferta` referencia `(intento_id, cotizacion_id)`;
  - `solicitud_venta` referencia `(oferta_id, cotizacion_id)`.
- **Solicitud de venta.**
  - `UNIQUE (idempotency_key)` más `hash_pedido`. La misma clave con el mismo hash devuelve la solicitud existente; con otro hash responde 409.
  - Índice único parcial sobre `(cotizacion_id) WHERE estado IN ('PENDIENTE_ENVIO', 'RECIBIDA', 'DESCONOCIDA')`. Con dos confirmaciones concurrentes de ofertas distintas, una sola fila entra; la otra recibe la violación de unicidad y se responde 409.
  - La fila en `PENDIENTE_ENVIO` se confirma en la base **antes** de llamar al core. En esa misma transacción se valida que la oferta siga vigente y que su core admita venta.
  - `cliente_id NOT NULL`.
  - `aceptar_hasta` se copia del vencimiento efectivo de la oferta al registrar la solicitud, no cambia después y forma parte de `hash_pedido`.
  - La solicitud guarda `acuse_en` y `referencia_core`. Cada envío queda en una tabla mínima de historial de envíos (número, inicio, fin y resultado), en lugar de las columnas `envios` y `ultimo_envio_en` propuestas antes.
  - El timeout de 3 segundos es configuración del adaptador de A, no un dato de la base.
  - Cada desenlace se aplica con una actualización condicional: `UPDATE … WHERE estado IN ('PENDIENTE_ENVIO', 'DESCONOCIDA')`. Un desenlace que llega cuando la solicitud ya es `RECIBIDA` o `RECHAZADA` no la modifica; solo completa su envío.
  - El reenvío actualiza la misma fila. No hay ningún proceso que libere el seguimiento ni reenvíe por tiempo.
- **Vigencia en la oferta.** Columnas `limite_core`, `limite_plataforma`, `vence_efectivo` y `origen_vencimiento`. Un `CHECK` exige `vence_efectivo IS NOT NULL` cuando el origen es conocido y `vence_efectivo IS NULL` cuando es desconocido; además exige que el vencimiento coincida con el menor de los límites válidos y que el origen sea coherente con los valores guardados.
- **Mapeo por intento.** La resolución y validación de la revisión de mapeo ocurre en el backend antes de insertar el intento. No se agrega una FK compuesta entre intento, core y mapeo.
- **Vistas.**
  - `v_solicitud_sin_resolver`: `DESCONOCIDA`, más `PENDIENTE_ENVIO` cuyo último envío o creación superó el umbral de 5 segundos. El umbral se toma de configuración para poder ajustarlo después de medir.
  - `v_aviso_plazo_ajustado`: ofertas con origen «ajustado al core», para la consulta del responsable.
  - `v_evento_negocio`: reemplaza la tabla `BusinessEvent` del anexo en el Nivel 1, armada a partir de cotización, instancia, intento, oferta y solicitud.

## 7 Diferido y qué se conserva desde el Nivel 1

| Tema | Nivel | Dato o relación que se conserva en el Nivel 1 |
| --- | --- | --- |
| Tabla de bien asegurable | N3, con cupos | Identificador sintético opcional del bien dentro del riesgo y cliente en la solicitud. |
| Vincular una cotización anónima a un cliente | N2 | `cliente_id` opcional en la cotización. |
| Recotización guiada, con motivo e instancia y oferta de origen | Candidata, N3 según el anexo | La instancia de cálculo y su número dentro de la cotización. |
| Reintentos de cotización | N2 | Un intento por core con su resultado. |
| Publicación progresiva de ofertas y avisos por ofertas tardías | N2 | Estado y tiempos de cada intento. |
| Resolver solicitudes sin respuesta consultando el core | N2 | Estados `DESCONOCIDA` y `PENDIENTE_ENVIO`, referencia del core y registro de envíos. |
| Venta de ofertas de B | N3, con la recotización de volátiles | Parámetros de referencia y huella de cada intento de B. |
| Cupos y exclusividad entre cores | N3 | Cliente, core, estado y fecha en la solicitud. |
| Audiencia en la asignación | N3 | Historial de asignaciones. |
| Ciclo de publicación completo | N2 | Estado de publicación separado de la activación. |
| Plazo de aceptación distinto por perfil | N3 | Regla de plazo dentro de la revisión de mapeo. |
| Envío del aviso por canal y su seguimiento | N2 o N3 | Vista del aviso para el responsable. |
| Tabla `BusinessEvent` y auditoría administrativa completa | N2 | Vista de eventos e historial de asignaciones. |
| Tablas de categorías, modalidades y opciones de suma | N2 o N3 | Su contenido vive en las revisiones de perfil y de mapeo. |
| Aviso de venta al cliente | N2 | Identificador de solicitud. |
| Idempotencia al crear cotizaciones | N2 | Sin cambios en el Nivel 1. |
| Cobertura provisional | N3 | Estado de la solicitud separado de un futuro estado de cobertura. |

## 8 Decisiones para aprobar

### Reglas de negocio

1. La cotización guarda el número público. La instancia guarda los datos de riesgo y las revisiones que se usaron. En el Nivel 1 hay una instancia por cotización y un intento por core, sin reintentos ni referencias de origen (sección 1).
2. La cotización responde de forma sincrónica cuando todos los intentos terminaron, con un timeout de 2 segundos por core. La publicación progresiva y los avisos por ofertas tardías pasan al Nivel 2 (sección 1).
3. El backend resuelve y valida la revisión de mapeo por producto y core antes de registrar cada intento (sección 1).
4. Con el formulario abierto, el backend vuelve a resolver la versión de producto y la revisión de perfil. Si alguna cambió, se recarga la configuración antes de crear la cotización (sección 1).
5. Una bicicleta por cotización. Cambiar de bien abre otra cotización (2.1).
6. Para confirmar hace falta un cliente identificado. El visitante no confirma en el Nivel 1 (2.1).
7. A vende y B compara, como limitación declarada del Nivel 1 (2.1).
8. Una venta exitosa por seguimiento. Las solicitudes pendientes o desconocidas bloquean otra solicitud (2.1 y 2.3).
9. La respuesta `RECIBIDA` de A registra la venta, con el instante en que el backend guarda el acuse (2.2).
10. Timeout de envío a A de 3 segundos, que lleva la solicitud a `DESCONOCIDA`, y umbral de 5 segundos para mostrar un `PENDIENTE_ENVIO` como envío sin confirmar. El umbral no libera el seguimiento ni reenvía. La resolución es reenviar la misma solicitud con la misma clave y los mismos datos, sin volver a validar la vigencia en la plataforma (2.4).
11. `RECIBIDA` y `RECHAZADA` son estados finales; los reenvíos concurrentes quedan en el historial de envíos y la venta se cuenta una vez. El pedido se reconstruye con datos inmutables, con la condición de que los datos sintéticos del cliente no cambien durante la demo (2.3 y 2.4).
12. Contrato de solicitud de A: lleva `aceptarHasta`, conserva sus claves después de un reinicio y cumple los casos de aceptación 1, 1b, 2a, 2b y 3 (sección 5).
13. La duración de un plazo se cuenta desde que el backend registra la respuesta del core (sección 3).
14. Una oferta sin ningún límite válido se puede comparar, pero no confirmar. Un límite del core inválido deja la oferta en ese caso aunque exista un límite de plataforma (sección 3).
15. El responsable ve el aviso de ajuste en una consulta, sin envío externo (sección 3).
16. Una asignación activa por producto y canal (sección 4).
17. La revisión de perfil guarda condiciones neutrales y la revisión de mapeo guarda las traducciones y los plazos por core y plan (sección 4).
18. Los ajustes de contrato de la sección 5 y la tabla de equivalencias de estados. El campo, la unidad y el umbral del dato de bicicleta eléctrica se aprueban con la ficha.

### Implementación en PostgreSQL

1. PostgreSQL 16, estados como `text` con `CHECK` e importes `numeric(14,2)` en ARS.
2. Claves de revisión por perfil o mapeo, hash de contenido según RFC 8785 y conflicto en `db-setup` cuando el contenido difiere.
3. Triggers de inmutabilidad y `ON DELETE RESTRICT` sobre revisiones y versiones.
4. Índice único parcial de asignación por producto y canal, más un historial que solo admite inserciones.
5. FK compuestas entre instancia, intento, oferta y solicitud.
6. Unicidad de la clave de idempotencia, índice parcial de una solicitud vigente por seguimiento y registro de `PENDIENTE_ENVIO` antes de llamar al core, con `aceptar_hasta` incluido en el hash del pedido.
7. Columnas y `CHECK` de vigencia en la oferta, con `vence_efectivo` obligatorio cuando el origen es conocido.
8. Vistas de solicitudes sin resolver, aviso de plazo ajustado y eventos de negocio.
9. Historial mínimo de envíos y desenlaces aplicados con actualización condicional sobre estados no finales.

## 9 Propuestas surgidas al redactar la entrega 2

El [esquema de la entrega 2](../entregas/entrega-02/01-esquema-y-der.md#11-propuestas-nuevas-de-este-borrador) detalla diez propuestas (PN-01 a PN-10) que este registro no resolvía. Todas son **Propuesta N1**. Las que cambian el diseño son:

- **PN-01.** Cada perfil y cada mapeo apuntan a su revisión vigente. `db-setup` crea el perfil o mapeo, inserta su revisión y completa el puntero en la misma transacción, sin claves diferidas. Repetir la carga no cambia el puntero.
- **PN-02.** Historial mínimo de envíos de cada solicitud.
- **PN-03.** Una cotización sin cliente identificado no se puede vender, y el cliente de la solicitud es el de la cotización.
- **PN-04.** Un límite del core inválido deja la oferta con origen desconocido (sección 3).
