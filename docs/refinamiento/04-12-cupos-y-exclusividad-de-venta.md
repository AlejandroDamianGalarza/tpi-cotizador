# Bienes cupos y exclusividad en la venta simulada

**Estado: refinamiento para la segunda entrega.** Este documento extiende el [envío y la atribución de la venta del 4.12](04-12-atribucion-de-venta-y-estado-de-solicitud.md) y las [reglas de reintento y póliza vigente del 2.5](02-05-contrato-de-cores-precio-y-confirmacion.md). Una persona puede tener varias bicicletas y cotizarlas por separado. Los cupos y las restricciones entre cores se definen como reglas comerciales explícitas cuando correspondan.

## Reglas distintas de elegibilidad y duplicación

| Regla | Qué controla | Ejemplo sintético |
| --- | --- | --- |
| Idempotencia | Repetir la misma confirmación debe devolver su resultado sin crear otra venta. | Un doble clic sobre la confirmación de la bicicleta 1 conserva la misma solicitud. |
| Venta por cotización | Como criterio candidato, cada cotización corresponde a una bicicleta y admite una sola venta exitosa. | Si se confirma la oferta de A para la bicicleta 1, vender también la oferta de B del mismo seguimiento sería otra operación y quedaría bloqueada por esta regla si se aprueba. |
| Cupo por cliente | Un límite de la plataforma puede abarcar las ventas de un cliente en el producto o período definido. | Si se fija un cupo global de tres ventas activas, la cuarta supera el cupo aunque un core tenga disponibilidad propia. |
| Cupo por cliente y core | Cada core puede aplicar un límite propio según el acuerdo y la ficha. | Si A admite tres y B admite cuatro operaciones para el cliente, completar el cupo de A afecta sus ofertas. B puede seguir habilitado si sus reglas se cumplen. |
| Exclusividad entre cores | Una regla de la plataforma puede impedir vender cuando ya existe una operación o cobertura en otro core, dentro de un alcance explícito. | Puede acotarse al mismo bien y período o abarcar al cliente para un producto. La segunda opción también afectaría bicicletas distintas y requiere que ese alcance se acuerde de forma expresa. |

El ejemplo de tres y cuatro son valores de prueba. Para un cupo hay que decidir qué consume una unidad, por ejemplo una solicitud aceptada, una cobertura activa o una venta registrada durante un período. También falta decidir cuándo se libera el cupo y cómo tratar una solicitud pendiente o de resultado desconocido. Un cupo confirmado por un core y uno calculado con registros propios tienen fuentes de evidencia distintas.

## Ficha técnica y acuerdo comercial

Una restricción debe indicar en la ficha o en un anexo versionado qué cliente, bien, producto, core y período evalúa, qué estados cuentan, qué fuente confirma la condición y desde cuándo aplica. También debe indicar qué se muestra cuando la condición impide vender y si la oferta puede seguir visible para comparar.

Si existe exclusividad entre cores, la ficha debe referenciar el acuerdo aplicable y aclarar qué pasa con un cliente que ya compró en otro core. El backend aplica la regla explícita y conserva el motivo. Una venta aceptada por A no bloquea por sí sola otra bicicleta en B en el criterio base propuesto. Si se pacta ese bloqueo, queda registrado como una regla distinta de evitar que dos ofertas cubran el mismo bien y período.

## Base propuesta para el DER del Nivel 1

- **Cliente y bien asegurable.** Representar que un cliente puede estar asociado a varias bicicletas. La identidad del bien se define con datos sintéticos en la PoC. Una cotización anónima puede empezar con los datos del riesgo y vincular cliente y bien al identificarse, según la regla que se apruebe.
- **Cotización y oferta.** Como simplificación candidata, una cotización sigue un solo bien y puede contener varias ofertas de distintos cores. Cotizar otra bicicleta abre otro seguimiento.
- **Solicitud de venta.** Referencia la oferta elegida, el cliente y el bien, con el core trazable desde la oferta. La idempotencia pertenece a la solicitud de confirmación. La regla de una venta exitosa por cotización, si se aprueba, se valida aparte y cubre también el caso de ofertas distintas del mismo seguimiento.
- **Evolución de elegibilidad.** Conservar las referencias de cliente, bien, producto, core, estado y fecha que permiten evaluar reglas futuras. Los cupos y la exclusividad quedan como ampliaciones configurables. Si se incorporan, se verifican al confirmar y se registra la versión de la regla y su resultado. Para cupos bajo control local, la prueba debe contemplar dos confirmaciones concurrentes cerca del límite.

Para cerrar el DER, queda por aprobar la regla de **una bicicleta por cotización y una sola venta exitosa por seguimiento**. También siguen pendientes qué ofertas de cada core se pueden vender en Nivel 1 y qué acuse permite contabilizar esa venta.
