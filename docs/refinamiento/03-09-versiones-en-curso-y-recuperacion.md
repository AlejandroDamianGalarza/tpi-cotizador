# Versiones en curso y recuperación de cotizaciones

Punto 3.9 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Una cotización iniciada conserva la configuración con la que se creó. Si se bloquea esa versión por un incidente, la operación afectada se detiene en el paso correspondiente y puede recuperarse con una versión aprobada o volver a iniciarse. Este punto continúa la [administración de perfiles del 3.8](03-08-administracion-de-perfiles-y-acceso-de-demostracion.md) y se relaciona con la [confirmación del 2.5](02-05-contrato-de-cores-precio-y-confirmacion.md) y la [recotización del 2.6](02-06-huella-historial-y-notificaciones.md).

## Configuración guardada de la cotización

La sesión conserva la referencia necesaria para retomar el proceso. Como diseño candidato, el backend fija una foto de configuración al iniciar la cotización y la guarda con su ID interno. La foto identifica la versión del producto y de la experiencia, el perfil comercial y su revisión, y las referencias a las reglas y los esquemas aplicados. Cada intento registra además qué adaptador y mapeo utilizó. Guardar una referencia a una revisión exige que esa revisión siga disponible para reconstruir la operación.

Una nueva versión publicada o una reasignación comercial se puede aplicar a cotizaciones nuevas. Las que ya comenzaron siguen con su configuración guardada mientras sea utilizable. La foto conserva lo aplicado. La vigencia de la oferta y las verificaciones que correspondan al confirmar siguen sus propias reglas. Falta decidir si «iniciar» ocurre al abrir el formulario o al crear el primer registro de cotización, y cómo se retoma un formulario que quedó abierto antes de ese momento.

## Publicación y compatibilidad

La versión compatible se determina para el contexto de la operación y sus pasos pendientes. Una versión puede admitir los datos ya capturados y, aun así, ser incompatible con el perfil, las reglas de cotización, el contrato del core o el importe ya elegido. Queda por definir qué compatibilidad se exige en cada paso y quién la valida.

| Situación | Comportamiento candidato |
| --- | --- |
| Se publica una versión nueva compatible | La cotización en curso conserva la versión fijada. Los recorridos nuevos usan la versión asignada y habilitada para su contexto. |
| Se publica una versión superior incompatible | Cada cotización en curso sigue con su versión guardada si todavía está habilitada y disponible. Las nuevas usan una combinación aprobada y compatible con el perfil y el recorrido. |
| La versión guardada se bloquea por un bug | Impedir las acciones afectadas y comunicar que el proceso necesita revisión. Conservar el registro original y ofrecer recuperación si existe una versión compatible aprobada para ese caso. |
| La versión guardada deja de estar disponible y no existe reemplazo compatible | Conservar el historial y pedir iniciar una cotización nueva si hay una configuración vigente apta. Si no la hay, informar que por el momento no se puede continuar. |

Para un primer escenario, propongo declarar de forma explícita de qué versión a cuál se permite recuperar un proceso y en qué casos. El número de versión no decide la compatibilidad. Antes de publicar un cambio hay que saber qué perfiles y recorridos quedan habilitados, qué versiones pueden coexistir y qué opciones quedan para las cotizaciones abiertas.

## Bloqueo y recuperación por incidente

Marcar una versión como afectada debe quedar registrado con motivo, alcance, momento y actor autorizado. Se tiene que definir qué se bloquea, por ejemplo iniciar, volver a cotizar o confirmar. El estado de la versión y el de cada operación son datos distintos. Una venta confirmada con anterioridad y una solicitud enviada sin respuesta requieren un tratamiento propio.

La «sanitización» se entiende como **recuperar datos permitidos en un proceso nuevo o revisado**. Como opción candidata, partir del ID original, copiar solo los campos compatibles aprobados y validarlos con la versión corregida. Los datos faltantes se vuelven a pedir. El proceso original conserva su configuración y resultados para poder reconstruir lo sucedido.

- Si se recupera un formulario que todavía no obtuvo ofertas, se puede reutilizar lo validado y completar lo que falte.
- Si ya había una oferta, revisar su vigencia y qué afectó el incidente. Si hace falta recotizar o cambian el premio, la cobertura o las condiciones, mostrar la nueva oferta y pedir una nueva elección antes de confirmar.
- Si ya se envió una confirmación y el resultado quedó desconocido, resolver el estado de esa solicitud antes de reenviarla o iniciar una venta sustituta. Las ventas ya confirmadas conservan el vínculo a la oferta aceptada.

## Prioridad propuesta para la PoC

Primero demostrar que una cotización iniciada en v1 puede continuar en v1 después de activar v2, mientras una nueva usa v2 si el perfil está habilitado. Eso permite probar el versionado sin incorporar un sistema general de migraciones. Si se aprueba sumar el escenario de incidente, probar un bloqueo simulado con una única transición compatible declarada e incluir un caso sin reemplazo. La administración completa de compatibilidad y recuperación queda pendiente de estimación junto con el 3.8.

## Preguntas para la segunda lectura

1. ¿En qué paso se fija la foto de configuración y cuánto tiempo se puede retomar una operación con ella?
2. ¿Qué dimensiones deben ser compatibles para retomar datos, volver a cotizar y confirmar? ¿Quién aprueba la relación entre versiones?
3. ¿Quién puede bloquear una versión, con qué alcance y qué mensaje se muestra a cliente y operador?
4. ¿Qué campos pueden recuperarse y en qué casos se exige una cotización nueva y una nueva aceptación?
5. ¿Qué capacidades de administración y recuperación queremos demostrar y cuáles pueden representarse con versiones y reglas de prueba ya preparadas?
