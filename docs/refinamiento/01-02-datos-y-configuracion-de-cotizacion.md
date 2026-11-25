# Datos y configuración de la cotización de Movilidad

Punto 1.2 del refinamiento del TPI. **Estado: borrador para segunda lectura.** La propuesta es resolver un perfil comercial antes de cotizar y usarlo junto con la versión del producto para determinar qué opciones son válidas. Este punto continúa el [refinamiento de actores, identidad y perfiles comerciales](01-01-actores-identidad-y-perfiles.md).

## Perfil comercial y productor

Toda cotización requiere un perfil comercial efectivo. En el recorrido anónimo se usaría el perfil estándar. Para una persona identificada o una venta asistida, el perfil depende de la elegibilidad y de los permisos que se definan en el punto 1.1. La plataforma mantiene los perfiles y resuelve su aplicación en el backend.

Un perfil puede afectar el precio, las ofertas disponibles o los montos admitidos. Se planteó asociarlo a un código de productor. Como diseño candidato, el perfil usa una clave interna y la plataforma resuelve el código aplicable para cada core y producto. Así un mismo perfil puede traducirse de forma distinta según la integración. El perfil comercial, la entidad del convenio, el productor y el operador son conceptos distintos, incluso si los datos de prueba los relacionan uno a uno.

Hay dos formas de representar una condición comercial. Un core puede determinarla por un código de productor, plan o producto. Otro puede aceptar un porcentaje de descuento en el pedido. En ambos casos el backend elige la condición aplicable y el adaptador representa esa decisión en el contrato correspondiente. El premio cotizado se toma de la respuesta del core. Queda por definir si algún caso requiere cálculo de precio en la plataforma y qué lo autoriza.

Descuentos y campañas como Black Friday son variantes posibles de estas condiciones. Requieren definir vigencia, combinación con convenios y prioridades. Hay que distinguir la vigencia de la configuración comercial de la vigencia de una oferta y registrar cuál se aplicó en cada operación.

## Datos mínimos y ubicación

Para bicicleta convencional se propuso pedir el código postal del riesgo. Falta precisar si representa el domicilio del titular, el lugar habitual de guarda u otra ubicación relevante para el core. También se debe acordar si la provincia y la localidad son necesarias para cotizar y en qué casos un código postal alcanza para determinarlas.

Para el primer alcance, propongo **entrada manual de código postal y un conjunto local acotado de ejemplos**. Si la provincia o localidad es necesaria y la referencia local no resuelve unívocamente el código ingresado, el flujo debe pedir una selección o responder que esa ubicación aún no está contemplada, según lo que se apruebe. La búsqueda asistida por provincia y localidad se puede especificar como una mejora posterior.

Georef documenta la búsqueda y normalización de provincias y localidades y ofrece descargas para uso local. Su referencia consultada no establece una conversión de localidad a código postal. Correo Argentino ofrece un buscador de CPA y distingue el CPA vigente de ocho caracteres del antiguo CP de cuatro cifras. Antes de proponer una integración postal hay que confirmar una fuente adecuada, cobertura y condiciones de uso. El buscador público, por sí solo, deja abierta la disponibilidad de una API para la PoC. [Normalización con Georef](https://www.argentina.gob.ar/georef/normalizacion-de-unidades-territoriales) · [Descargas de Georef](https://www.argentina.gob.ar/georef/descarga-de-la-base-completa) · [Información de Correo Argentino](https://www.correoargentino.com.ar/5-sobre-la-informacion-que-brinda-correo-oficial) · [Buscador de CPA](https://www.correoargentino.com.ar/formularios/cpa)

En la referencia pública consultada, Luis Guillón figura con el CP de cuatro cifras `1838` y Monte Grande con `1842`. Esos valores son ejemplos de CP, no de CPA completo ni de una relación universal de localidad a código postal. [Referencia del BCRA](https://www.bcra.gob.ar/archivos/Pdfs/comytexord/C10045.pdf)

El anexo conserva la bicicleta convencional como modalidad inicial y agrega bicicleta eléctrica en v2. Quedan pendientes los campos mínimos de cada modalidad, las unidades, las coberturas y los datos que acepta cada core. Incluir monopatín queda como ejemplo de extensión futura. Puede exigir reglas, campos y mapeos nuevos, además de una opción en el catálogo.

## Configuración visible y mapeos internos

| Responsable | Qué define o consume |
| --- | --- |
| Producto Movilidad | Define modalidades, datos obligatorios y opcionales, reglas de riesgo y etiquetas compatibles con la versión del producto. |
| Backend de cotización | Resuelve el perfil, el productor y las opciones vigentes. Combina reglas del producto, condiciones comerciales y capacidades conocidas de los cores. Vuelve a validar la selección cuando recibe la solicitud. |
| API de experiencia y frontend | Presentan la configuración pública resultante, como «Bicicleta», campos, opciones y límites. El formulario envía identificadores estables de la plataforma y valores de riesgo. La búsqueda geográfica, si se incorpora, ayuda a completar el formulario. |
| Adaptador de core | Traduce la modalidad, el productor, el plan, el monto o el porcentaje a lo que requiere su core. También traduce la respuesta a una oferta con términos comparables cuando corresponda. |

La plataforma debe decidir qué modalidad muestra, cuál propone por defecto y cuándo deja de ofrecerla. Como regla candidata, si se deshabilita una modalidad en un core, se puede mantener visible si otro core aún la ofrece. Si deja de estar disponible para todo el contexto, la interfaz debe informarlo. Queda por precisar qué pasa con un formulario abierto o una oferta existente al cambiar la configuración. Una regla de visibilidad temporal y una nueva versión funcional del producto son cambios distintos.

## Sumas aseguradas

La suma asegurada solicitada, el valor declarado del bien y el premio son datos diferentes. El primero indica el monto de cobertura que se quiere cotizar, el segundo describe el riesgo cuando el producto lo requiere y el tercero es el precio que informa la oferta.

| Modalidad de selección | Comportamiento candidato |
| --- | --- |
| Montos fijos | El usuario selecciona una opción del catálogo, por ejemplo una suma vinculada a un código de plan. El backend traduce esa opción al código o monto según el core. |
| Monto personalizado | El usuario ingresa un importe. El backend valida moneda, límites y, si corresponde, incremento admitido por la configuración y el core. |
| Opciones generadas desde un rango | El backend puede calcular opciones y entregar la lista al frontend. Con extremos de 10.000 y 100.000 y cuatro valores equiespaciados, el ejemplo produce 10.000, 40.000, 70.000 y 100.000. Faltan fijar redondeo, incrementos y tratamiento de valores repetidos. |

Para comparar ofertas por precio, propongo usar al menos una suma disponible en ambos cores y mostrar su valor efectivo. Si se pide un monto que solo admite uno de ellos, hay que decidir si se omite el otro core con un motivo visible o si se muestran alternativas claramente diferenciadas. Cambiar el monto al cotizar requiere una decisión expresa. El adaptador debe conservar la suma efectivamente ofrecida para que una diferencia de cobertura resulte visible.

## Prioridad propuesta y caso de revisión

Para el Nivel 1 propongo empezar con perfiles sintéticos **estándar y un convenio**, código postal manual, conjuntos de datos locales y opciones de suma predefinidas. Una misma opción puede traducirse a un código fijo para un core y a un monto para el otro. Eso permite demostrar el mapeo y comparar ofertas en una condición común. El rango editable, la generación de opciones, las campañas superpuestas y el buscador externo se pueden evaluar como incrementos según su valor para la demo.

Caso candidato para revisar. Un visitante cotiza una bicicleta con el perfil estándar y una suma común a ambos cores. En otra cotización, un cliente elegible aplica un convenio simulado. Al reconstruir el caso se puede identificar la versión de producto, el perfil y su versión o referencia de configuración, las condiciones aplicadas, la suma solicitada, la suma efectivamente ofertada y el resultado de cada core. La expectativa exacta de precios y disponibilidad dependerá de las reglas que se aprueben para los simuladores.

## Preguntas para la segunda lectura

1. ¿Qué identifica el código postal del riesgo? ¿Aceptaremos CP numérico, CPA o ambos? ¿Qué hacemos con una ubicación ambigua o fuera del conjunto de prueba?
2. ¿Qué campos y coberturas exige cada modalidad? ¿Qué datos adicionales de bicicleta eléctrica cambian realmente la cotización y el contrato con los cores?
3. ¿Puede un perfil resolver distintos productores o productos según el core? Si falta un mapeo, ¿se omite ese core o se permite una alternativa comercial informada?
4. ¿Cuándo queda fijada la configuración aplicable? Si cambia una vigencia, un descuento o una opción entre apertura del formulario, cotización y venta, ¿qué conserva la operación y cuándo se pide cotizar de nuevo?
5. ¿Qué tipos de suma necesitamos demostrar en el Nivel 1? ¿Qué puede ajustar el operador y cómo respondemos cuando una suma solo está disponible en uno de los cores?

El punto 1.3 podrá retomar estas respuestas para definir cuándo dos ofertas son comparables, qué diferencias deben mostrarse y cuándo una alternativa debe presentarse aparte.
