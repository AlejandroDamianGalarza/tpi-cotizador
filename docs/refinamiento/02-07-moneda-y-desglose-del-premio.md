# Moneda y desglose del premio de Movilidad

Punto 2.7 del refinamiento del TPI. **Estado: borrador para segunda lectura.** La PoC opera en pesos argentinos. Cada importe conserva su moneda y el historial distingue el premio de la cotización del que informa el core al procesar la solicitud. Continúa el [contrato de cores y confirmación](02-05-contrato-de-cores-precio-y-confirmacion.md) y el [refinamiento del historial y los avisos](02-06-huella-historial-y-notificaciones.md).

## Moneda de cada importe

El alcance contempla una sola moneda operativa, `ARS`. El contrato y el registro deben asociar un código de moneda a los importes, incluidas las sumas aseguradas, los premios y los componentes monetarios del desglose. Para la PoC se excluyen la conversión, los tipos de cambio, la selección de moneda y la operación en moneda extranjera.

En una futura integración, el core podría recibir la moneda como parámetro explícito o derivarla de un código de productor, un producto o un perfil comercial. El adaptador resolverá esa diferencia. Como criterio candidato, si el simulador omite la moneda, el adaptador solo puede completar `ARS` cuando su contrato lo garantice. Una moneda incompatible o indeterminada se distingue de una oferta en pesos apta para comparar y confirmar.

## Premio en cotización y en solicitud

| Momento | Qué conserva la operación |
| --- | --- |
| Cotización | Oferta y plan, premio y moneda, período o base de pago, vigencia y desglose informado en ese momento. Si puede haber variaciones al solicitar, comunicarlo junto al importe cotizado. |
| Solicitud | Referencia a la oferta seleccionada y registro separado del premio y desglose devueltos por el core. Si la respuesta todavía no determina el importe final, conservar ese estado. |
| Confirmación | Referencia a la versión de los importes y condiciones que quedaron aceptados, si corresponde. Los datos de la oferta original siguen disponibles para reconstruir la operación. |

El core podría incorporar un sellado o una tasa al procesar la solicitud, por ejemplo según la provincia. Es un caso a simular y especificar con el contrato de ese core. La plataforma conserva lo que informa cada respuesta. Si aparece una diferencia y el core explica el motivo, se registra. Cuando la causa no viene informada, queda como desconocida. La PoC no calcula cargos legales por su cuenta.

La posibilidad de variación forma parte del comportamiento planteado. **El aviso previo de que el importe puede variar y la aceptación del importe resultante son momentos distintos.** Como regla candidata, si el premio cambia, mostrar el importe anterior, el nuevo y el motivo disponible, y pedir una confirmación expresa antes de registrar la venta simulada. Si un core futuro devuelve el importe recién después de una acción irreversible, hay que aclarar ese contrato antes de habilitar la venta por ese camino.

## Desglose disponible para consulta

Conservar el desglose de ambas etapas con el detalle que efectivamente devuelva el core. Un componente informado en cero se distingue de uno ausente. Si solo llega el total, se registra «solo total». Si llega una parte del desglose, se marca «parcial». Cuando el contrato permite comprobar la suma de los componentes y el resultado difiere del total, queda registrada la inconsistencia sin completar importes por inferencia.

La pantalla principal puede mostrar el premio, su base de pago y el aviso de variación cuando corresponda. El detalle guardado puede servir para una vista ampliada, una plantilla PDF o un correo posterior. Esas salidas son opciones de evolución. Si se incorporan, deben indicar a qué etapa corresponde el desglose y usar el registro de esa operación.

## Caso candidato para segunda lectura

El core simulado ofrece un plan con un premio total de `ARS 4.900` para el período cotizado. Al procesar la solicitud informa `ARS 5.100` para el mismo período, incluido un sellado de `ARS 200` que no figuraba en la cotización. La operación conserva ambos importes y desgloses, muestra la diferencia antes de confirmar y permite reconstruir cuál importe se aceptó. Si el core informa solo el total actualizado, la diferencia sigue siendo visible y su causa queda sin informar.

1. ¿En qué respuesta de cada core aparece el premio definitivo? ¿Qué puede ejecutarse antes de conocerlo?
2. ¿Qué diferencias requieren confirmación expresa? ¿Cómo se presenta un cambio de importe, período o condiciones de pago?
3. ¿Qué componentes reporta cada simulador en cada etapa y con qué regla podemos verificar su total?
4. ¿Alcanza para el Nivel 1 con conservar los desgloses y mostrar el cambio de premio, o necesitamos una vista de detalle en la demo?
