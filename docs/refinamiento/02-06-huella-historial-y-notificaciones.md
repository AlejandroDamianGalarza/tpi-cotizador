# Huella historial y notificaciones de Movilidad

Punto 2.6 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Este punto define qué información permite volver sobre una cotización, revisar diferencias y reconstruir una venta simulada desde el acceso hasta su aviso al cliente. Continúa el [refinamiento del contrato de los cores](02-05-contrato-de-cores-precio-y-confirmacion.md) y retoma los permisos de historial del [punto 1.1](01-01-actores-identidad-y-perfiles.md).

El anexo 2.2 ya contempla identificadores internos de cotización e intento, una huella para solicitudes equivalentes y trazabilidad de la venta. Ubica la recotización de ofertas volátiles en Nivel 3. Las reglas de búsqueda manual del historial, el registro explícito del método de acceso y el aviso de venta son propuestas de este refinamiento. Su incorporación al Nivel 1 requiere una decisión de alcance.

## Huella y referencias de la cotización

La huella permite comprobar si dos operaciones partieron de los mismos criterios según una regla de normalización versionada. También ayuda a investigar inconsistencias. Por ejemplo, si coinciden el riesgo y la configuración pero cambia el premio, la traza permite revisar fecha, plan, vigencia, versión y resultado del core. Una diferencia puede deberse a una actualización válida de tarifa. Una coincidencia de huellas requiere interpretar qué campos cubría esa versión de la regla.

| Dato | Para qué sirve |
| --- | --- |
| ID interno de cotización | Identifica la operación y permite abrirla desde una consulta autorizada. |
| ID de intento y referencia del core | Identifican cada llamada. El core puede dar un ID externo, requerir parámetros para volver a cotizar o carecer de un mecanismo de recuperación. |
| Huella de criterios | Detecta coincidencias sobre los campos acordados, aun entre cotizaciones distintas. La huella no permite reconstruir los valores originales. |
| Referencia a la cotización de origen | Vincula una cotización nueva con la que se usó para iniciarla, aunque cambie la huella, el premio o la vigencia. |

Para poder recrear un pedido hace falta conservar por separado un resumen autorizado de los parámetros necesarios. Hay que decidir si la huella compara solo el riesgo o también perfil, productor, plan o condiciones comerciales. También hay que distinguir las fechas técnicas de las fechas que cambian el significado del pedido. Si una fecha altera el riesgo o la cobertura solicitada, debe formar parte de la comparación que corresponda. Como referencia técnica, la canonicalización de JSON permite obtener una representación estable para calcular una huella, una vez acordadas las reglas del dominio. [RFC 8785](https://www.rfc-editor.org/info/rfc8785/)

## Historial y vigencia de ofertas

Con un cliente seleccionado, el operador podría consultar manualmente las cotizaciones que él mismo hizo para esa persona. El acceso a las que inició el cliente, otro operador u otra entidad se define según los permisos del 1.1. La consulta usa la referencia del cliente y los filtros autorizados. La huella ayuda a comparar resultados después de acceder al historial.

Conviene distinguir dos acciones:

1. **Abrir una cotización anterior.** Mostrar qué se pidió, qué ofertas llegaron y qué plazo de aceptación informó cada core. Una oferta vencida sigue en el historial y se marca como vencida. Se conserva por separado el período de cobertura y el plazo para aceptar el precio.
2. **Volver a cotizar a partir de la anterior.** Como diseño candidato, crear una cotización nueva vinculada al ID original. Validar el contexto vigente, usar una fecha nueva cuando el contrato del core lo requiera y generar intentos nuevos. Si el core permite presentar los mismos datos de riesgo, su adaptador arma el pedido con la versión compatible. El nuevo premio, las condiciones y la vigencia se guardan en una oferta nueva para que el usuario los revise antes de confirmar.

Queda por definir si la recotización usa el perfil y la versión vigentes o permite conservar alguno del original, y qué pasa si una modalidad, un plan o un convenio ya no está disponible. El adaptador debe distinguir la fecha del pedido de la fecha de inicio de cobertura cuando el core las use con sentidos diferentes.

## Evidencia de la venta simulada

Para reconstruir un reclamo o investigar una irregularidad, la PoC puede registrar una secuencia de hechos vinculados. Como mínimo, la confirmación identifica la fecha y hora registrada por el backend, el cliente, el actor que la realizó, el canal, el perfil comercial y la oferta exacta con su premio y versión. También conserva el ID de venta, el resultado y la referencia externa cuando exista. Las consultas permiten diferenciar una intención o solicitud pendiente de una venta efectivamente confirmada en la simulación.

El registro debe poder indicar el método de acceso que observó la plataforma y su origen, por ejemplo cuenta Google, acceso por DNI o Banco A. En venta asistida se distingue **cómo accedió el operador** de **cómo se identificó al cliente**. Si la cotización empezó anónima y después se vinculó a un cliente, esa transición se registra por separado. En la PoC se marca el origen como simulado. Para una futura integración, el registro sirve para investigar el recorrido y su alcance probatorio dependerá de cómo se verifiquen y conserven esos hechos.

La traza usa referencias internas y tiene permisos de consulta propios. Los logs operativos deben excluir contraseñas, tokens, fotos de DNI, documentos completos y datos de pago. Los accesos al historial o al registro de auditoría se pueden registrar cuando lo requiera la regla que se apruebe. [Guía de registro de eventos de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html#data-to-exclude)

## Aviso al cliente

Al confirmar una venta simulada se plantea generar un aviso al cliente por el medio acordado. Correo, WhatsApp y SMS son candidatos. El mensaje podría incluir un resumen y una referencia, código o enlace para consultar la operación. Si se elige un enlace o código, quedan por definir quién puede usarlo y por cuánto tiempo.

El aviso queda asociado al ID de venta. El registro incluye canal, destino enmascarado o referencia interna, fecha y hora, plantilla o tipo de mensaje, intentos y resultado. Conviene distinguir **aviso generado**, **envío intentado**, **aceptado por el canal**, **entrega confirmada** y **fallo** según la evidencia disponible. En un simulador el resultado debe indicarse como simulado. La aceptación del envío y la recepción por el cliente son hechos diferentes.

Como criterio candidato, repetir la confirmación devuelve la misma venta y conserva un solo aviso de confirmación por canal elegido. Si el aviso falla, se registra y puede reintentarse sobre el mismo aviso. El estado de la venta y el estado de la notificación se consultan por separado.

## Prioridad propuesta y preguntas abiertas

Para el Nivel 1, propongo demostrar la trazabilidad desde una cotización hasta una venta simulada y registrar, si entra en el alcance, **un solo canal de aviso simulado**. La consulta manual acotada del historial y la recotización pueden decidirse por separado. La recotización de Core B conserva la prioridad que tiene en el anexo hasta que se acuerde un cambio.

1. ¿Qué campos definen dos pedidos equivalentes y qué diferencias queremos detectar? ¿Qué parámetros necesitamos conservar para armar un nuevo pedido por core?
2. ¿Qué ve el operador en el historial, por cuánto tiempo y con qué trazabilidad de acceso?
3. ¿Qué fecha se envía al recotizar y qué perfil, versión y condiciones se vuelven a resolver? ¿Qué tiene que reconfirmar el cliente si cambian el premio, la cobertura o la vigencia?
4. ¿Qué hechos y fuentes permiten afirmar que la venta se confirmó y distinguir una operación de cliente de una asistida?
5. ¿Qué canal simulado usamos primero, cuál es el momento de generar el aviso y qué resultado queda disponible si falla?
