# Comparación de ofertas de Movilidad

Punto 1.3 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Una persona puede mirar planes lado a lado cuando responden al mismo escenario de cotización. La provincia y el código postal ubican el riesgo. La cercanía del premio ayuda a encontrar alternativas de presupuesto. Las coberturas, los límites, las franquicias y las asistencias explican qué recibe la persona por ese importe. Este punto continúa el [refinamiento de datos y configuración de la cotización](01-02-datos-y-configuracion-de-cotizacion.md).

## Qué comparamos

1. **Escenario.** Como criterio inicial, agrupar las ofertas devueltas para la misma solicitud, con igual provincia, código postal y modalidad de Movilidad. Así se mantienen las demás características del riesgo y el perfil comercial solicitado. En una comparación futura entre convenios, el perfil aplicado deberá quedar visible en cada oferta.
2. **Premio a pagar.** Mostrar el importe informado por el core y ordenarlo cuando esté expresado sobre una base común, con moneda, período y condiciones de pago identificables. La cercanía de importes es informativa. Para el Nivel 1 se propone ordenar por premio sin fijar un porcentaje de cercanía. Si los importes corresponden a períodos distintos o faltan datos, mostrarlos con su unidad y evitar un orden de menor a mayor que mezcle bases diferentes.
3. **Alcance.** Para presentar alternativas se pueden mostrar sumas, plazos de cobertura y franquicias distintos, siempre que esas diferencias queden visibles. Para comparar solo cuánto cuesta una protección bajo las mismas condiciones habría que verificar la cobertura efectiva, su suma, la franquicia, la vigencia y las condiciones de pago. La fecha hasta la que se puede aceptar una oferta se muestra aparte del período de cobertura.

La analogía con los planes de autos sirve para discutir catálogos con distintos niveles. En Movilidad cada core puede devolver uno o varios planes con denominaciones propias. La etiqueta «Completo», por ejemplo, no alcanza para determinar qué incluye un plan ni para compararlo con otro de nombre parecido.

## Categorías y detalle de cada core

El comparador puede usar un conjunto acotado de categorías del producto, como responsabilidad civil, fracturas y asistencias. El backend asignaría categorías mediante un mapeo explícito por core y plan o cobertura. Si solo hay nombres parecidos, se pueden usar como pista para revisar el mapeo. La agrupación definitiva requiere conocer el alcance informado. Cuando no se pueda identificar una categoría con respaldo, queda como detalle del core sin agrupar.

- **Coberturas.** «Fractura de muñeca» y «Fracturas en varias zonas» pueden verse en una fila común de **Fracturas**. Cada celda conserva el alcance informado y sus límites. Un plan puede ofrecer responsabilidad civil (RC) con una suma mayor y otro puede excluirla. La suma se muestra por cobertura donde esté disponible.
- **Franquicias.** Mostrar a qué cobertura aplica cada franquicia y su forma de cálculo si el core la informa. Un importe fijo y un porcentaje sobre el daño pueden exhibirse juntos con su unidad y condición, sin deducir cuál termina costando más sin conocer el siniestro. «Sin franquicia» se reserva para una condición explícita.
- **Asistencias.** Registrar el servicio y cuándo se activa. Un traslado particular ante daños a la bicicleta y una ambulancia ante lesiones pueden estar en la categoría de **Asistencias**, con descripciones distintas en la comparación.
- **Datos ausentes.** Mostrar «No incluida» cuando el plan excluya explícitamente la cobertura. Usar «No informado» cuando falte información suficiente. Esa distinción también vale para franquicias, montos y asistencias.

## Ejemplo sintético para revisar

Las tres ofertas ilustrativas corresponden a una bicicleta, a la misma provincia y código postal y al mismo perfil. Comparten moneda, período de cobertura y forma de pago. Los importes, alcances y nombres son ejemplos para la PoC.

| Dato | Core A · Esencial | Core B · Movilidad | Core A · Ampliado |
| --- | --- | --- | --- |
| Premio | ARS 4.900 al mes | ARS 5.100 al mes | ARS 6.500 al mes |
| Daños a la bicicleta | Hasta ARS 100.000 | Hasta ARS 100.000 | Hasta ARS 100.000 |
| Responsabilidad civil | No incluida | Hasta ARS 100.000 | Hasta ARS 200.000 |
| Fracturas | De muñeca | En varias zonas especificadas en el plan | No informado |
| Franquicia por daños a la bicicleta | ARS 10.000 fijos | 10 % del daño | Sin franquicia |
| Asistencia | Traslado particular ante daños a la bicicleta | Ambulancia ante lesiones por caída | No informado |

Los dos primeros planes tienen un premio cercano. El comparador debe mostrar que difieren en RC, alcance de fracturas, franquicia y asistencia. El tercer plan sirve para ver una suma más alta de RC y otras diferencias. El mayor premio por sí solo no establece que el plan sea preferible para todos los casos.

## Prioridad propuesta para el Nivel 1

Usar dos cores simulados y mostrar un conjunto pequeño de planes lado a lado, con etiquetas propias del core y categorías acordadas. Para la demo, dos ofertas de premio cercano con diferencias claras alcanzan para poner a prueba la comparación. Una tercera puede mostrar cómo cambia la suma de RC, si suma evidencia al caso.

Como criterio candidato de aceptación, dada una misma solicitud, las ofertas se agrupan por el escenario del riesgo. Si tienen la misma base de pago se pueden ordenar por premio. El usuario puede ver para cada plan el alcance informado, la suma por cobertura cuando corresponda, la franquicia y el detalle de las asistencias. Una exclusión explícita debe verse distinta de un dato omitido. La recomendación automática de planes, la similitud inferida por nombre y las conversiones entre períodos de pago quedan para evaluar después de validar este recorrido.

## Preguntas para la segunda lectura

1. ¿Qué categorías entran en la demo? ¿Usamos responsabilidad civil, fracturas y asistencias como conjunto inicial o agregamos daños al bien como cobertura base?
2. ¿Qué dato informa cada simulador para distinguir «No incluida» de «No informado»? ¿Quién valida los mapeos cuando dos nombres parecen referirse a la misma categoría?
3. ¿Qué base de importe usamos en el Nivel 1? Si el usuario pide una suma que ambos cores admiten, ¿mostramos en el mismo cuadro planes que devuelven una suma distinta, o los ubicamos aparte como alternativas?
4. ¿Qué vigencia de cobertura y plazo de aceptación simulará cada core? ¿Qué diferencias deberían verse en la vista inicial y cuáles pueden quedar en el detalle?

El punto 1.4 definirá qué devuelve el recorrido cuando hay ofertas de ambos cores, una sola alternativa disponible o ninguna.
