# Resultados parciales y ofertas tardías de Movilidad

Punto 1.4 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Cada entidad puede presentar sus ofertas para la misma solicitud y la persona puede verlas a medida que estén disponibles. Con una oferta válida y elegible ya puede avanzar. Las respuestas pendientes, tardías o fallidas se conservan como parte de la trazabilidad de la cotización. Este punto continúa el [refinamiento de la comparación de ofertas](01-03-comparacion-de-ofertas.md).

## Qué ve la persona durante la cotización

| Situación | Comportamiento candidato |
| --- | --- |
| Todavía no hay ofertas y hay respuestas pendientes | Mostrar qué entidades siguen calculando y permitir consultar el avance. El estado indica que todavía faltan respuestas. |
| Hay una oferta y al menos una entidad sigue calculando | Mostrar la oferta disponible junto con un aviso discreto, por ejemplo «La aseguradora X sigue calculando». La persona puede esperar, revisar la oferta o avanzar si es válida y elegible. |
| Hay ofertas de ambas entidades | Mostrar las opciones de ambas con los criterios del punto 1.3. Si una entidad devuelve varios planes, su estado de resolución se distingue de la cantidad de planes disponibles. |
| Una entidad rechaza el riesgo, no puede cotizarlo o falla | Mantener las ofertas válidas de la otra entidad. Distinguir «sin oferta para este riesgo» de «no pudimos obtener una respuesta». El motivo técnico detallado queda en la trazabilidad interna. |
| Terminaron los intentos o se cerró la espera sin ofertas válidas | Informar que no hay ofertas disponibles para continuar en esa cotización, con los motivos que puedan mostrarse. Queda pendiente si la persona puede reintentar un core o debe pedir una nueva cotización. |

La velocidad determina cuándo aparece una oferta. El cuadro de comparación puede ordenarlas por la misma base de premio y las condiciones del punto 1.3. Una respuesta publicada después se incorpora al mismo cuadro con una indicación de nueva oferta. La latencia se puede medir por entidad. Su causa, como saturación por volumen, requiere evidencia independiente.

## Elección y confirmación de la oferta

- **Antes de elegir.** Una oferta nueva se incorpora sin bloquear el recorrido. Si la persona está mirando un plan, esa vista no debería saltar a otro por un cambio en el orden de la lista.
- **Con una oferta elegida y antes de confirmar.** Se puede mostrar «Llegó una nueva oferta» y permitir verla, volver al comparador, cambiar explícitamente de opción o ignorar el aviso. La opción elegida queda seleccionada hasta una nueva elección. La confirmación identifica la oferta concreta y comprueba que siga vigente y elegible.
- **Tras confirmar la venta simulada.** El resultado posterior puede quedar asociado a la cotización para reconstruir el caso. La venta conserva la oferta efectivamente confirmada. Se debe definir con la especificación de venta el instante que cuenta como confirmación y cómo se resuelve una respuesta que llega al mismo tiempo.

Un aviso contextual podría decir «Llegó una oferta con menor premio mensual» si ambas usan la misma base de pago. También podría señalar una diferencia verificable, como un límite mayor de responsabilidad civil al mismo premio. El mensaje debe conservar visibles las contrapartidas, como una franquicia mayor o prestaciones de otro alcance. Para el primer escenario, propongo un aviso neutral de nueva oferta. Un aviso sobre ventajas específicas requiere reglas de comparación aprobadas.

## Identificadores y continuidad

La plataforma conserva el **ID interno de cotización** y registra un **ID interno por intento de core** desde el envío de la solicitud. Cada intento guarda la entidad, tiempos, estado y resultado cuando exista. La referencia externa se agrega al recibirla, si el core la proporciona. Si un intento vence o termina con error, conserva su identidad y su último estado. Un reintento, cuando corresponda, se debe distinguir del intento anterior.

La sesión sirve para regresar a la cotización y recuperar las ofertas y estados que le corresponden. Como diseño candidato, la sesión referencia el ID interno y el backend mantiene el registro de intentos. Queda por definir cuánto tiempo se puede retomar, cómo se protege el acceso desde una sesión anónima y cuándo un operador autorizado puede consultar la misma operación.

## Ventana de espera y respuestas tardías

Hay dos momentos que conviene definir por separado. La **disponibilidad para mostrar y elegir** comienza cuando llega la primera oferta válida. La **ventana de espera de cada core** determina hasta cuándo el sistema mantiene pendiente su intento. El primer momento permite seguir con la compra aunque el segundo todavía no haya llegado.

- Si un core responde después de que ya se mostró otra oferta y dentro de su ventana, la respuesta puede agregarse a la misma cotización y mostrarse al volver o actualizar la vista.
- Si la llamada terminó por timeout, se registra ese desenlace. Incorporar una respuesta después de ese límite depende de que el contrato del core permita recuperarla o recibirla y correlacionarla de forma confiable. Para la PoC queda por definir si algún simulador representará ese caso y si esa respuesta puede volver a ofrecerse o solo quedará como evidencia.
- La vigencia de una oferta y su elegibilidad para venta siguen las reglas de cotización y venta. La llegada tardía no determina por sí sola que una oferta esté vigente ni que sea apta para vender, en especial en el core volátil.

## Prioridad propuesta para el Nivel 1

El caso candidato usa los dos cores del acta con demoras sintéticas distintas. El primero devuelve una oferta elegible. Mientras el segundo sigue calculando, la persona puede continuar. Si vuelve antes de confirmar y el segundo ya respondió dentro de la ventana, puede comparar las dos opciones o seguir con la elegida. La publicación de la segunda oferta se puede demostrar con una actualización de la vista durante la sesión. Los avisos fuera de la sesión, la recuperación después del timeout y los avisos de ventajas específicas quedan para estimar y aprobar como incrementos. Esta propuesta requiere revisar el esfuerzo junto con la distinción del anexo entre el Nivel 1 y la tolerancia a fallas del Nivel 2.

Escenarios candidatos para verificar el comportamiento:

1. Con un core pendiente y una oferta elegible, el flujo permite avanzar y conserva ambos intentos.
2. Si el segundo responde antes de confirmar, la persona puede ver la oferta nueva y cambiar explícitamente su elección. Si la ignora, se mantiene la selección que había hecho.
3. Si se confirma mientras el otro core sigue pendiente, la venta queda vinculada solo a la oferta elegida. Una respuesta posterior conserva su relación con la cotización cuando se puede correlacionar.
4. Si ninguna entidad devuelve una oferta válida y se cierra la espera, el flujo muestra el resultado y conserva los desenlaces para distinguir rechazo, timeout y fallo técnico.

## Preguntas para la segunda lectura

1. ¿Cuánto puede quedar pendiente cada core y cuándo se cierra la cotización para nuevas respuestas? ¿Qué casos simulamos dentro y fuera de ese límite?
2. ¿Qué forma de actualización alcanza para la demo? ¿Ver las novedades al regresar o refrescar, o mostrar también un aviso mientras la persona permanece en el flujo?
3. ¿Durante cuánto tiempo se puede retomar la cotización? ¿Cómo cambia el acceso si empezó como visitante, después se identificó o la consulta luego un operador?
4. ¿Qué hace elegible una oferta para confirmar? ¿La oferta del core volátil permite una venta simulada o queda como alternativa informativa en el Nivel 1?
5. ¿Cuándo se ofrece reintentar y cuándo pedir una nueva cotización? ¿Qué estados deben verse y cuáles quedan para diagnóstico?
