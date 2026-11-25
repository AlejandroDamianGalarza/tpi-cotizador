# Contrato de cores precio y confirmación de Movilidad

Punto 2.5 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Este punto desarrolla qué envía el backend a cada core, qué respuestas debe distinguir y qué cambia al pasar de cotizar a confirmar. Continúa los borradores de [configuración de la cotización](01-02-datos-y-configuracion-de-cotizacion.md), [comparación de ofertas](01-03-comparacion-de-ofertas.md) y [resultados parciales](01-04-resultados-parciales-y-ofertas-tardias.md).

El acta y el anexo 2.2 incluyen dos cores simulados y una venta simulada idempotente. Excluyen pagos, cobranzas y emisión legal. La detección de póliza vigente, el bloqueo por cobertura previa y una vista de vigencias quedan planteados en esta vuelta como propuestas de refinamiento.

## Precio y condiciones de cada plan

Cada plan devuelto por un core es una oferta. Debe poder identificarse y mostrar su premio a pagar, moneda, base de pago y condiciones. El desglose puede incluir prima pura, impuestos, sellados y otros componentes que informe el core. El descuento puede estar incluido en el precio, aparecer en el desglose o estar informado solo como condición de la solicitud.

El perfil comercial puede resolver un porcentaje de descuento explícito o códigos de productor, perfil, plan o producto, según el core. Esos valores explican **qué se solicitó**. El total y el desglose reportados por el core explican **qué se ofreció**. Un código comercial no es un componente monetario. Para la PoC se propone mostrar el premio devuelto, conservar el nivel de detalle disponible y dejar señalada cualquier inconsistencia según la regla que se apruebe. Falta definir qué componentes ya incluyen descuentos, cómo se verifica el desglose cuando corresponde y qué pasa si el total falta o no coincide. La plataforma no deduce componentes ausentes ni aplica un descuento por segunda vez.

## Configuración inicial y contrato común

Como diseño candidato, la API de experiencia entrega una configuración inicial de cotización con la modalidad, los campos, las opciones permitidas y una referencia a la configuración aplicada. El frontend usa el mismo contrato para todos los perfiles y envía el riesgo y las opciones elegidas con identificadores de la plataforma. En el backend se resuelven el perfil efectivo, los mapeos y los parámetros de cada core. Al recibir la cotización, el backend comprueba que la selección siga habilitada. Falta decidir cuándo se fija ese contexto y qué ocurre si cambia entre abrir el formulario, cotizar y confirmar.

| Pieza | Responsabilidad candidata |
| --- | --- |
| Contrato canónico | Definir el significado común del riesgo, los planes, el precio y los resultados. El backend invoca a los cores por una interfaz estable. |
| OpenAPI y JSON Schema | Describir las operaciones HTTP y validar la estructura de datos donde haga falta. La estructura y la traducción tienen responsabilidades distintas. [OpenAPI](https://spec.openapis.org/oas/v3.1.0.html) · [JSON Schema](https://json-schema.org/understanding-json-schema/about) |
| Adaptador por core | Traducir al contrato externo, resolver códigos, normalizar respuestas y declarar qué operaciones y referencias admite. Una capacidad que el core no soporta debe quedar distinguida de una llamada fallida. |
| Versiones y configuración interna | Registrar qué versión de producto y qué configuración comercial se usaron, junto con el adaptador, su mapeo y la versión de API del core cuando corresponda. El contrato con el frontend y la conexión al core pueden evolucionar por separado si siguen siendo compatibles. |

La configuración inicial puede salir del catálogo del backend. Si un core futuro requiere una preconsulta técnica para conocer planes o montos, se especificará su vigencia, dependencia y respuesta ante fallas. El adaptador encapsula esa consulta y el frontend sigue leyendo la configuración pública de la plataforma.

## Resultados y reintentos por operación

| Operación | Qué se debe distinguir | Criterio candidato |
| --- | --- | --- |
| Cotizar | Oferta o varias ofertas, sin oferta para el riesgo, rechazo de datos, capacidad no disponible, timeout o falla técnica. | En un caso transitorio definido por contrato, repetir el mismo pedido puede conservar el ID de cotización y crear un intento nuevo. Si cambió el riesgo, tratarlo como una cotización nueva vinculada a la anterior. Si se recalculó el premio, identificar la nueva oferta y conservar la anterior. Consultar el estado y volver a cotizar son acciones distintas. |
| Solicitar o confirmar | Rechazo definitivo, pendiente, confirmada, o resultado desconocido cuando falta respuesta después de enviar la solicitud. | Cada operación que puede generar un efecto tiene identidad propia. Al repetir la misma confirmación, devolver o consultar el resultado asociado a esa identidad. Una clave reutilizada con datos distintos produce un conflicto. Si se perdió la respuesta, resolver el estado antes de reenviar o cambiar a otro core, salvo que el contrato garantice un reenvío idempotente dentro de sus condiciones. |

Cotizar se trata como un cálculo en la PoC. Para una integración futura hay que confirmar si una cotización externa también crea registros o reservas. Un timeout de confirmación no permite saber si el core aplicó la operación. Si un contrato externo no permite consultar ni deduplicar, el caso debe permanecer pendiente de resolución. Eso evita reenviar a ciegas una solicitud que podría duplicar una póliza o un pago en un sistema real. [Semántica de reintentos HTTP, RFC 9110 §9.2.2](https://www.rfc-editor.org/rfc/rfc9110.html#section-9.2.2)

La PoC puede demostrar una sola venta simulada aun si se pierde la primera respuesta, con un registro idempotente bajo control de la plataforma. La traza debe poder unir cotización, oferta elegida, venta o solicitud, intentos, core, referencia externa si existe, versiones, estado y tiempos. La deduplicación de una misma confirmación y el control de dos ventas distintas sobre el mismo bien son reglas separadas.

## Póliza vigente y elegibilidad de venta

La regla de negocio planteada permite **cotizar y comparar aunque exista una póliza vigente**. Cuando se confirma que hay una póliza activa en conflicto, la selección queda informativa y se bloquea la venta simulada hasta que la póliza anterior deje de estar vigente según la fuente acordada. El prototipo puede mostrar el motivo y un resumen autorizado de la cobertura vigente. La gestión de baja queda fuera de este flujo.

El bloqueo requiere definir qué significa «en conflicto». Como punto de partida, propongo **mismo cliente y mismo bien asegurable**, dentro de las modalidades y cores incluidos, para permitir que una persona asegure una segunda bicicleta. Falta decidir cómo se identifica el bien, qué coberturas o fechas se solapan y si una póliza de un core bloquea la confirmación en el otro. Detectarla en un core solo informa sobre la fuente consultada. La evidencia que hace falta para bloquear entre cores es una decisión de dominio.

Si el visitante todavía es anónimo, se pueden mostrar precios y dejar la elegibilidad de venta por verificar. Con identidad disponible, una póliza encontrada durante la cotización puede mostrarse o registrarse en una consulta de vigencias autorizada. La confirmación debe basarse en un estado suficientemente reciente según la regla aprobada. «Sin póliza en la fuente consultada» y «No pudimos verificar» son resultados distintos. Si la verificación se vuelve obligatoria, propongo mantener el resultado como pendiente cuando la fuente falle y permitir seguir cotizando.

## Prioridad y preguntas para la segunda lectura

Para el Nivel 1, propongo demostrar primero dos respuestas de cotización con distinto detalle de precio y una confirmación simulada repetida que conserva la misma venta. Si se aprueba incorporar el bloqueo, probarlo con un registro sintético y tres casos: cobertura vigente para el mismo bien, ausencia de cobertura en la fuente consultada, y verificación no disponible. Una vista de cartera, consultas a varios cores y la recuperación de una confirmación externa quedan como ampliaciones a estimar.

1. ¿Qué diferencia a «solicitar» de «confirmar» en el flujo? ¿Cuál de esos pasos registra la venta simulada y qué estado devuelve cada uno?
2. ¿Qué componentes y descuentos informa cada simulador? ¿Qué respuesta espera el usuario si el premio es incompleto o inconsistente?
3. ¿Qué fallas de cotización se pueden reintentar? Para confirmar, ¿qué referencias y capacidades tendría que ofrecer un core para consultar o resolver un resultado desconocido?
4. ¿Cómo identificamos el mismo bien? ¿Qué alcance de coberturas, vigencias y cores aplicaría al bloqueo por póliza vigente?
5. ¿En qué pasos se consulta la vigencia, qué ocurre si no se puede verificar y qué detalle pueden ver el cliente y el operador?
