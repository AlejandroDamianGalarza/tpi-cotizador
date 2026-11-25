# Vencimiento y recotización de ofertas de Movilidad

Punto 4.10 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Cada oferta tiene un plazo de aceptación. Cuando vence se mantiene disponible para consulta y se ofrece volver a cotizar. El backend puede reutilizar los datos que ya conserva cuando la persona solicita la recotización. Este punto desarrolla la [vigencia de ofertas del 2.6](02-06-huella-historial-y-notificaciones.md) y se relaciona con la [confirmación del 2.5](02-05-contrato-de-cores-precio-y-confirmacion.md), el [premio del 2.7](02-07-moneda-y-desglose-del-premio.md) y la [compatibilidad de versiones del 3.9](03-09-versiones-en-curso-y-recuperacion.md).

## Plazo de aceptación por oferta

El vencimiento se aplica a cada oferta. Dos planes de la misma cotización pueden tener plazos distintos según el core o el perfil aplicado. Los plazos de **una hora** y **24 horas** son ejemplos para revisar. El plazo para aceptar el premio se distingue de la fecha de inicio y fin de la cobertura, y de la fecha que se envía al core en el pedido.

Como regla candidata, registrar la vigencia informada por el core, el límite interno que corresponda al perfil y el vencimiento efectivo con su origen. Si existen ambos límites, tomar el más temprano. Si el core no informa la vigencia, el plazo interno indica hasta cuándo la plataforma acepta usar la oferta. La confirmación puede requerir que el core revalide el premio según su contrato. Queda por definir desde qué momento se cuenta el plazo interno y cómo se versiona esa configuración.

El frontend muestra hasta cuándo se puede usar la oferta y, al vencer, la marca como **Vencida** y ofrece **Volver a cotizar**. El backend valida el plazo al intentar confirmar, incluso si el formulario quedó abierto y todavía muestra el precio anterior. Si una confirmación ya se envió y su resultado es desconocido, se resuelve por el flujo del 2.5 antes de generar otra venta.

## Volver a cotizar sin repetir el formulario

1. La persona pulsa **Volver a cotizar** sobre la oferta vencida. El frontend referencia la cotización y la oferta existentes. Esta acción autoriza calcular otra oferta.
2. El backend valida el acceso a la operación, recupera los datos que se pueden reutilizar y comprueba que sigan siendo válidos para el perfil, el producto y la versión aplicable. Si falta un dato o quedó desactualizado, pide solo lo necesario.
3. El adaptador prepara un pedido nuevo según el contrato del core. Puede reenviar los datos de riesgo requeridos y actualizar la fecha del pedido si el core la exige. La fecha de inicio de cobertura solo se cambia cuando lo requiere el caso y con la validación correspondiente.
4. La respuesta genera una oferta nueva, con premio, desglose disponible, condiciones y nuevo plazo. La persona la revisa y decide si quiere confirmarla. Si el plan dejó de estar disponible o el core falla, se informa el resultado y se conserva la oferta anterior en el historial con su estado.

Queda por decidir si el primer flujo recotiza solo la oferta seleccionada, vuelve a pedir planes al mismo core o consulta a todos los cores habilitados. Para acotar la demo, propongo empezar por el core de la oferta seleccionada.

## Trazabilidad del origen

La recotización por vencimiento se registra como una cotización nueva vinculada a la anterior. Conserva la referencia a la oferta vencida, el motivo **vencimiento**, sus propios intentos y los datos vigentes que se aplicaron. Una cotización inicial tiene origen **nuevo**. Si hay varias recotizaciones, se puede recorrer la relación con la anterior hasta llegar a la original. Para la PoC alcanza con los identificadores internos relacionados. Un código visible distinto queda abierto si lo necesita el cliente o soporte.

El historial debe permitir distinguir una consulta de la oferta vencida, un reintento técnico de la misma operación y una recotización iniciada por la persona. La huella del 2.6 puede ayudar a comparar los criterios entre ambas cotizaciones, aun si varían el precio, el plazo o los parámetros del core.

## Prioridad y preguntas para la segunda lectura

Como prioridad para el Nivel 1, propongo mostrar el vencimiento y evitar que se confirme una oferta vencida. La recotización guiada para un core simulado queda como escenario candidato si aporta a la demo y entra en el tiempo disponible. Se debe revisar junto con el alcance de recotización de ofertas volátiles previsto en el anexo, sin dar por resuelta la elegibilidad de venta de Core B.

1. ¿Desde cuándo se cuentan los plazos por core y perfil? ¿Qué muestra la pantalla cuando el plazo proviene solo de una regla interna?
2. ¿Qué datos, fechas y contexto comercial se pueden reutilizar y cuáles debe reconfirmar la persona?
3. ¿La recotización recupera el mismo plan o agrega alternativas? ¿Qué pasa cuando ese plan ya no está disponible?
4. ¿Cuándo tomamos por válida una confirmación que llega cerca del vencimiento y qué evidencia conserva el backend?
