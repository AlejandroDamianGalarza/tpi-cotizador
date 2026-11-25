# Referencia pública e instancias de cotización

Punto 4.11 del refinamiento del TPI. **Estado: borrador para segunda lectura.** La persona conserva el mismo número de seguimiento cuando retoma una cotización y pide calcularla de nuevo. El backend registra por separado las instancias de cálculo y los intentos contra los cores. Esto precisa el vínculo de origen del [4.10](04-10-vencimiento-y-recotizacion-de-ofertas.md) y retoma los [contratos de core del 2.5](02-05-contrato-de-cores-precio-y-confirmacion.md) y las [referencias del 2.6](02-06-huella-historial-y-notificaciones.md).

## Qué conserva cada identificador

| Identificador | Qué permite reconstruir |
| --- | --- |
| Referencia pública de seguimiento | La cotización que la persona consulta o retoma. Se mantiene al recotizar dentro del mismo seguimiento. La interfaz muestra su estado y las ofertas que correspondan. |
| ID interno de instancia de cálculo | Cada cálculo iniciado, incluido el original y los que surjan al recotizar. Conserva la instancia y la oferta de origen, el motivo, el contexto y las fechas. |
| ID interno de intento | Cada llamado a un core dentro de una instancia, con su estado y resultado. Una instancia puede tener uno o varios intentos, contra uno o más cores según el recorrido. |
| Referencia externa | El ID de transacción o referencia que entrega un core, si lo hace. El adaptador registra su relación con el intento y aplica la regla de consulta o reenvío de ese contrato. |

El ID nuevo que se plantea para una recotización en el 2.6 y el 4.10 corresponde a una **instancia interna**. El número de seguimiento visible se conserva. Una consulta del historial usa los permisos de la persona, aunque conozca ese número.

## Consulta, reintento y recotización

- **Consultar o retomar.** El backend recupera la información guardada bajo la misma referencia. Abrir la pantalla no inicia otro cálculo por sí solo.
- **Reintentar una llamada.** Si el contrato permite repetir el mismo pedido ante una falla transitoria, el backend registra otro intento dentro de la instancia. Si el core ya asignó una referencia y permite consultar su estado, queda por decidir si se consulta esa referencia antes de repetir la llamada.
- **Volver a cotizar.** Cuando corresponde iniciar otro cálculo, por ejemplo porque venció una oferta, se crea una instancia vinculada al origen. El adaptador traduce el nuevo pedido según el core. Un core transaccional puede iniciar otra transacción y devolver otra referencia. En un core volátil, los IDs de la plataforma conservan la traza aunque el core carezca de un ID recuperable. Si llega una oferta nueva, se guarda por separado para que la persona la revise antes de confirmar.

El contrato de cada core debe aclarar qué significa su referencia, cuándo puede reutilizarse y cuándo corresponde iniciar otra transacción. La confirmación de venta conserva su identificador y sus propias reglas de idempotencia. El mismo número de seguimiento puede agrupar varias ofertas. La confirmación identifica la oferta exacta elegida, y una respuesta tardía se vincula con la instancia e intento que la originaron.

## Caso candidato para la PoC

Una persona consulta la cotización `M-1042`. La oferta original vence y la persona pulsa **Volver a cotizar**. La pantalla sigue mostrando `M-1042` y, cuando llega el resultado, presenta el nuevo premio y su vigencia. Internamente, el backend registra dos instancias, los intentos realizados y las referencias externas que cada core haya informado. Así se puede distinguir qué oferta venció, cuál se volvió a cotizar y cuál se seleccionó para confirmar.

## Preguntas para la segunda lectura

1. ¿Qué cambios se consideran continuación bajo el mismo seguimiento, y cuáles inician una cotización con una referencia pública nueva? Por ejemplo, vencimiento frente a un cambio del bien asegurable.
2. ¿Qué puede hacer cada core con una referencia previa? ¿Consultar, repetir el mismo pedido o iniciar otra transacción?
3. Si dos instancias siguen abiertas o una respuesta llega tarde, ¿qué ofertas se muestran como vigentes en el seguimiento y cuáles aparecen en el historial?
4. ¿Qué parte de la traza necesita ver soporte u operador y qué información alcanza con mostrarle al cliente?
