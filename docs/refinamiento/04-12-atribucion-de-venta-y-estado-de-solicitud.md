# Atribución de la venta y estado de la solicitud

Punto 4.12 del refinamiento del TPI. **Estado: borrador para segunda lectura.** El cliente es tomador de la solicitud y asegurado en el caso base. La venta simulada se registra al enviar exitosamente la solicitud y conserva quién la realizó, por qué canal y con qué perfil. Este punto continúa la [definición de actores del 1.1](01-01-actores-identidad-y-perfiles.md), la [confirmación del 2.5](02-05-contrato-de-cores-precio-y-confirmacion.md) y el [seguimiento del 4.11](04-11-referencia-publica-e-instancias-de-cotizacion.md).

## Cliente, vendedor y canal

| Recorrido | Tomador y asegurado | Vendedor registrado | Canal |
| --- | --- | --- | --- |
| Cliente completa la solicitud | El cliente | El mismo cliente, identificado como tal | Autoasistido |
| Operador completa la solicitud para el cliente | El cliente | El operador autorizado que la envió | Asistido |

Ambos recorridos guardan el mismo tipo de registro. El **perfil comercial aplicado** queda vinculado a la cotización y a la oferta seleccionada. La **atribución de la venta** identifica a quien envió la solicitud. Queda por definir si se registra además al actor que inició la cotización cuando otra persona termina la operación. Los reportes pueden distinguir esa participación del vendedor sin recalcular el perfil desde los datos actuales del usuario.

## Envío de la solicitud y registro de la venta

La regla planteada ubica la venta simulada al enviar la solicitud. Como criterio candidato, se registra una vez cuando el backend tiene constancia del **envío exitoso**, por ejemplo un acuse válido del core que deja la solicitud en trámite. El tiempo de procesamiento posterior depende del core. Queda por acordar qué respuesta cuenta como acuse válido y en qué momento exacto el evento entra en los reportes de venta.

El backend conserva fecha y hora de envío, estado y los vínculos con cliente, actor, canal, perfil, oferta exacta y core. Si la misma confirmación se repite, aplica la regla idempotente del 2.5. Si el core rechaza la solicitud o un timeout deja el resultado desconocido, la plataforma puede mostrar el seguimiento con su estado real. Para el criterio candidato, esos casos quedan fuera del conteo de ventas exitosas mientras no haya evidencia suficiente. El resultado desconocido se consulta o reconcilia antes de reenviar.

## Solicitud en trámite y cobertura

Se planteó que el cliente pueda quedar cubierto mientras su solicitud sigue en trámite. Para representarlo, el contrato ficticio del core debe indicar desde cuándo y hasta cuándo hay cobertura y cómo se reconoce ese resultado. La etiqueta «pendiente» puede referirse al trámite posterior a una recepción exitosa o a la falta de una respuesta concluyente. Conviene conservar por separado el estado de la solicitud y el estado de la cobertura.

| Resultado observado | Mensaje candidato para la persona |
| --- | --- |
| Recepción confirmada y cobertura provisional informada | «Solicitud recibida. Cobertura provisional informada desde [fecha y hora]», con la vigencia que corresponda y etiqueta de simulación. |
| Recepción confirmada y cobertura todavía sin verificar | «Solicitud recibida. La cobertura aún está por confirmar». |
| Envío con resultado desconocido | «Estamos verificando el estado de tu solicitud». |
| Solicitud rechazada | Mostrar «Solicitud rechazada» y, cuando esté disponible, un motivo apto para el cliente. |

Para el Nivel 1 se puede demostrar el trámite sin emitir una póliza. Si se quiere incluir cobertura provisional, queda como caso sintético explícito de un core, con inicio, vigencia y desenlace definidos. La comunicación al cliente, incluida una notificación del 2.6, debe reflejar el estado que se pudo confirmar.

## Identificador de solicitud y referencia del core

El cliente recibe un **identificador propio de solicitud** para consultar su estado. Se relaciona con el número de seguimiento de la cotización del 4.11 y con la oferta aceptada. El backend vincula la solicitud con los intentos y con la referencia que el core pueda emitir o aceptar para consulta. Según el core, puede ser un número de propuesta, un ID de solicitud o una combinación de parámetros.

Cuando la referencia sea por parámetros, se guardan donde corresponda los datos mínimos definidos por el contrato, por ejemplo ramo, producto, canal y códigos de productor o socio. Esos datos describen el contexto. Su capacidad para identificar una solicitud concreta debe verificarse con el contrato simulado. Los identificadores y códigos internos del core se reservan para la traza y para consultas autorizadas.

## Preguntas para la segunda lectura

1. ¿Qué estado o acuse de cada simulador confirma el envío exitoso y dispara el registro de venta?
2. ¿Qué información del core habilita a mostrar cobertura provisional, desde cuándo y hasta qué evento o fecha?
3. ¿La demo necesita representar cobertura provisional o alcanza con mostrar una solicitud recibida con seguimiento?
4. ¿Qué referencia permite consultar una solicitud por cada core? ¿Cómo se comunica al cliente un resultado que todavía no se pudo verificar?
