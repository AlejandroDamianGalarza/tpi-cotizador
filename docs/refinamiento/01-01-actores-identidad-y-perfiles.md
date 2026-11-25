# Actores identidad y perfiles comerciales

Punto 1.1 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Este apartado describe cómo cliente y operador entrarían al cotizador, qué datos podrían usar y qué condiciones comerciales pueden afectar una oferta. Las propuestas de alcance y las preguntas abiertas se revisarán antes de aprobar la especificación.

## Base y propósito

El Acta de Constitución y el anexo 2.2 fijan para el Nivel 1 un frontend compartido, perfiles sintéticos de cliente y operador, cotización de Movilidad y venta simulada. El anexo incorpora autenticación real a partir del Nivel 2. Los bancos A y B de este escenario son entidades ficticias distintas de los cores A y B que cotizan el seguro. Los flujos de acceso y pago descritos acá son candidatos a simulación, con datos y documentos de prueba.

La idea de multicanalidad incluye varias formas de entrar como cliente y un recorrido asistido por vendedor. Conviene definir por separado el **método de acceso**, el **estado de identificación del cliente**, el **perfil comercial** que determina condiciones y el **canal** directo o asistido. Un cliente de Banco A puede cotizar por cuenta propia o mediante un operador. En ambos recorridos puede aplicar el mismo convenio, según la elegibilidad acordada.

## Cliente directo

| Situación | Comportamiento planteado | Decisión pendiente |
| --- | --- | --- |
| Visitante anónimo | Puede cotizar con el perfil y precio estándar. | Si se muestran precios orientativos de Banco A y Banco B, y cómo se conserva la cotización al identificarse. |
| Cuenta Google parcial | El escenario parte de correo y nombre cuando esté disponible. Se planteó completar el registro posterior y ofrecer una condición comercial vinculada a Google Pay. | Fuente de los datos personales requeridos, vinculación con una cuenta existente, condición exacta del beneficio y requisitos para habilitar la venta simulada. |
| Registro propio por DNI | Se planteó un alta con DNI y contraseña, con un paso de presentación de una imagen ficticia del documento. El perfil comercial sería estándar mientras carezca de un beneficio aplicable. | Qué representa la verificación simulada, tratamiento de registros duplicados y si un alta de este tipo aporta valor en el primer alcance. |
| Acceso por Banco A o Banco B | Cada banco ficticio aporta una identidad, datos de cliente y su convenio. Los datos suministrados por el banco, incluidos DNI, nombre, apellido y edad o fecha de nacimiento, quedarían en solo lectura dentro del cotizador. Se podrían representar medios de pago preasociados como opciones sintéticas. | Campos requeridos, qué sucede si faltan o discrepan, y si el convenio exige además un medio de pago elegible. |

El acceso bancario toma como referencia la experiencia pública de PagoMisCuentas. La referencia orienta el recorrido. Los contratos y datos de los bancos ficticios se definirán para la PoC. [PagoMisCuentas para personas](https://www.pagomiscuentas.com/)

El inicio de sesión con Google identifica una cuenta mediante `sub` y puede aportar correo y datos básicos de perfil. La API web de Google Pay devuelve información del medio de pago y, según la solicitud, datos como correo o dirección. Su respuesta documentada carece de un DNI y de una afirmación de identidad civil aplicable a este caso. Para completar un perfil parcial habría que especificar una fuente separada de datos de identidad. La disponibilidad de un medio de pago y la verificación de identidad se evaluarían por separado. [Google OpenID Connect](https://developers.google.com/identity/openid-connect/reference) · [Objetos de respuesta de Google Pay](https://developers.google.com/pay/api/web/reference/response-objects)

Una imagen ficticia puede representar que se presentó el documento. Si hace falta demostrar una verificación, el resultado puede provenir de un registro sintético con casos predefinidos, por ejemplo persona encontrada, duplicado o datos inconsistentes. La existencia de un registro sintético y la autenticidad documental son afirmaciones diferentes.

## Convenios y precios

El visitante puede obtener el precio estándar sin identificarse. También se planteó mostrar, para el mismo riesgo y cobertura, una estimación por convenio antes del acceso bancario. Esa estimación se mostraría como **condicionada a elegibilidad**. Queda por resolver si el acceso conserva la oferta, genera una nueva cotización o pide confirmar un precio distinto. Mostrar precios antes de pedir identificación permitiría probar una hipótesis sobre abandono. Medir su efecto requeriría evidencia específica.

Para una cuenta registrada por DNI, se planteó completar manualmente el dato del medio de pago y avisar sobre un posible beneficio bancario. Antes de incluirlo hay que definir qué acredita la elegibilidad, cómo se confirma el precio y si eso justifica ampliar el recorrido. La declaración de un banco o la selección de una tarjeta sería un dato de entrada. La elegibilidad requiere una regla expresa. También falta definir quién calcula o devuelve cada precio, qué pasa si un convenio carece de oferta y qué condiciones deben coincidir para comparar precios.

## Operador y venta asistida

- **Operador de una entidad.** El operador de Banco A selecciona un cliente de su entidad y usa los datos disponibles en ese contexto. La habilitación para operar por cuenta del cliente y los datos visibles requieren reglas propias.
- **Operador de un centro de ventas.** Se planteó un operador autorizado para Banco A y Banco B. Podría elegir entre los perfiles habilitados para ese cliente o recibir una sugerencia por prioridad y disponibilidad. Queda por decidir quién elige, qué define el orden y qué condición permite pasar al siguiente perfil. La autorización del operador y la elegibilidad del cliente se evalúan por separado.
- **Historial.** El operador podría consultar cotizaciones del cliente seleccionado para explicar las ofertas o preparar una alternativa. Falta delimitar si ve las que el cliente hizo por sí mismo, las originadas por otros vendedores o entidades y por cuánto tiempo. También falta definir el registro de la consulta y el tratamiento de una cotización anónima después de identificar al cliente.
- **Facultades adicionales.** El operador podría disponer de un rango más amplio que el cliente en el cotizador. Antes de especificarlo hay que aclarar si «precio de la movilidad» significa valor del bien, suma asegurada o premio, y quién fija, aplica y audita el rango. Cambiar el valor del riesgo y modificar el premio tienen efectos distintos sobre la oferta.
- **Atribución.** En una venta asistida se necesita distinguir al vendedor que realizó la operación del cliente al que se atribuye. El registro debe poder reconstruir el canal, el perfil comercial aplicado y la oferta seleccionada. Queda por definir si también se necesita identificar un operador que cotizó cuando otro termina la venta.

## Prioridad propuesta para la PoC

| Prioridad | Escenario candidato | Qué permite comprobar |
| --- | --- | --- |
| Primero | Visitante con precio estándar, cliente ficticio de Banco A y operador fijo de ese banco. Usar identidades, perfiles y elegibilidad de prueba. | Mismo recorrido de cotización, una diferencia comercial observable y atribución de una venta asistida. |
| Después, si aporta evidencia | Banco B como segundo convenio y una vista orientativa de precios, o una consulta acotada del historial. | Reutilización del perfil en otro convenio, transparencia antes del acceso o valor del canal asistido. Cada opción se evalúa por separado. |
| Refinamiento posterior | Registro con imagen simulada, cuenta Google parcial y su vinculación, elegibilidad por medio de pago, operador multientidad, historial entre entidades y ampliación de bandas. | Cada caso agrega reglas de identidad, autorización, precio o continuidad de oferta que conviene especificar antes de comprometerlo. |

Para el primer escenario, el criterio candidato es que una persona pueda cotizar sin identificarse con el perfil estándar y que un operador de Banco A pueda cotizar sobre un cliente autorizado del mismo banco. Al completar la venta simulada, el reporte debe permitir identificar cliente, vendedor, canal asistido y convenio aplicado. Este escenario se revisará junto con el alcance del acta, que mantiene dos cores simulados para cotizar y comparar.

## Preguntas para la segunda lectura

1. ¿Qué escenarios quedan comprometidos para el Nivel 1? ¿Alcanza con un convenio simulado y un operador fijo para demostrar el valor?
2. ¿Cuándo se considera identificado el cliente para la venta? ¿Cómo se vinculan dos accesos con una misma persona y qué pasa con su cotización anónima?
3. ¿El convenio depende de una afiliación, del medio de pago o de ambos? ¿Cuándo el precio es orientativo y en qué momento pasa a ser elegible para venta?
4. ¿Qué clientes y qué cotizaciones puede ver un operador? ¿Puede continuar una cotización iniciada por el cliente o solo consultarla y generar una nueva?
5. ¿Qué dato puede ajustar el operador, dentro de qué límites y con qué autorización? ¿Qué evidencia conserva el sistema al aplicar el ajuste?

El punto 1.2 retomará los datos mínimos del riesgo y las coberturas de bicicleta y bicicleta eléctrica. Para ese refinamiento conviene llevar una primera decisión sobre qué campos del cliente y qué condiciones del convenio intervienen en la cotización.
