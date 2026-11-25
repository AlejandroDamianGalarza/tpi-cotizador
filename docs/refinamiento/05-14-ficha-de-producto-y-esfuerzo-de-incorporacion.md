# Ficha de producto y esfuerzo de incorporación

Punto 5.14 del refinamiento del TPI. **Estado: borrador para segunda lectura.** La configuración de Movilidad parte de una ficha técnica y un manual de producto revisados. Cada core debe poder relacionar su oferta y sus requisitos con esa ficha. La incorporación de bicicleta eléctrica servirá para medir cuánto trabajo requiere entender, configurar, probar y documentar una nueva modalidad. Se relaciona con los [datos de cotización del 1.2](01-02-datos-y-configuracion-de-cotizacion.md), los [contratos de core del 2.5](02-05-contrato-de-cores-precio-y-confirmacion.md) y las [plantillas de perfiles del 3.8](03-08-administracion-de-perfiles-y-acceso-de-demostracion.md).

El acta pide documentar el tiempo y los archivos modificados al incorporar la segunda modalidad y compararlos con la creación de una aplicación separada. Para que esa comparación sea interpretable, la medición incluye el trabajo que ocurre antes y después de editar la configuración.

## Ficha técnica, manuales y mapeos

| Artefacto | Qué información aporta |
| --- | --- |
| Ficha técnica de producto | Plantilla base de alineación. Identifica la familia, modalidad y versión. Registra datos requeridos y condicionales, unidades, criterios de elegibilidad, coberturas, límites, franquicias, asistencias, vigencias y excepciones aplicables. Cada regla material lleva una referencia a su fuente. |
| Manual de producto | Explica las reglas y sus motivos, cómo se cotiza y solicita, qué puede ofrecerse y cómo se interpretan los casos especiales. Ayuda a resolver ambigüedades de la ficha. |
| Mapeo y configuración por core | Declara campos, códigos, unidades, transformaciones, restricciones y capacidades del contrato simulado. Indica qué regla de la ficha cubre cada mapeo y dónde queda una diferencia. |
| Manual de usuario | Describe cómo cliente u operador inicia una cotización, interpreta las ofertas y completa el recorrido aprobado. Incluye ejemplos de los estados que efectivamente ve cada perfil. |

La ficha común define qué espera la plataforma de la familia de producto. Cada core puede expresar esa modalidad con otros códigos, pedir un dato adicional, ofrecer otra suma o devolver una cobertura de alcance distinto. Esas diferencias se documentan en el mapeo o se elevan como decisión de producto. Si un core pide un dato obligatorio que la ficha no contempla, el equipo debe decidir si amplía la ficha, admite una variante explícita o deja esa combinación fuera. Los datos y contratos de la PoC serán sintéticos.

La plantilla de ficha de producto y la plantilla de perfiles del 3.8 tienen funciones diferentes. La ficha fija el significado de campos, reglas y coberturas. La plantilla de perfil reutiliza condiciones comerciales y opciones habilitadas para un contexto.

## Camino candidato para la PoC

1. Tomar la ficha inicial de **Movilidad v1** y su manual de producto. Revisar qué regla aplica y registrar dudas o discrepancias, con fuente y versión. Si la ficha y el manual difieren, la decisión se documenta antes de aplicar el cambio.
2. Completar la ficha de **bicicleta eléctrica** con los datos nuevos y las diferencias frente a bicicleta convencional. Revisar el manual y los escenarios de aceptación en conjunto.
3. Derivar una configuración versionada y un mapeo chico para cada core. Conservar la relación entre **regla de la ficha, dato o código del core y caso de prueba**. La interpretación y el mapeo se revisan antes de habilitar el producto. Un cálculo o una regla compleja requiere especificación y prueba propias.
4. Validar la estructura de la configuración, las reglas de negocio y los contratos simulados. Ejecutar un recorrido manual desde el formulario hasta las ofertas y probar también un dato obligatorio ausente y una modalidad no soportada por uno de los cores. Conservar el resultado observado y el esperado.
5. Actualizar el manual de usuario con los pasos y mensajes confirmados. Registrar el estado de aprobación de la ficha, la configuración y las pruebas, para que la demo pueda reproducirse.

Para empezar, propongo una **ficha en Markdown, una matriz breve de mapeo y una configuración versionada**. Un editor para administradores o la generación automática de configuración a partir de la ficha se pueden evaluar una vez que el proceso manual esté probado.

## Medición del esfuerzo y referencia

Registrar el tiempo efectivo desde la primera lectura de la ficha hasta tener la modalidad lista para la demo. Separar el tiempo efectivo de trabajo del tiempo calendario y de las esperas por decisiones. Una planilla o registro simple puede indicar por actividad el tiempo, el resultado, la evidencia y los archivos modificados.

| Actividad | Qué registrar |
| --- | --- |
| Entendimiento | Lectura de ficha y manual, dudas, decisiones y validación del alcance. |
| Configuración y adaptación | Cambios de catálogo, formulario, reglas y mapeos. Identificar componentes reutilizados y modificaciones de código o configuración. |
| Pruebas y correcciones | Escenarios ejecutados, fallas encontradas, correcciones y nueva verificación. |
| Documentación y demo | Cambios en manuales, preparación de casos y verificación final del recorrido. |

Comparar ese resultado con una **estimación explícita** de crear una aplicación separada para la misma modalidad, con los mismos criterios funcionales, de prueba y documentación. Desglosar qué piezas habría que crear o duplicar y qué reutilización seguiría disponible. Mostrar aparte el esfuerzo inicial de construir la base compartida cuando sea parte de la comparación. Si más adelante se obtiene un caso equivalente medido y autorizado, registrar el alcance y período antes de usarlo como referencia. Los archivos tocados muestran dónde cambió el sistema. El tiempo se registra por separado.

## Preguntas para la segunda lectura

1. ¿Qué campos mínimos tendrá la plantilla de ficha y quién resolverá discrepancias entre ficha, manual y contrato de core?
2. ¿Qué diferencias dejan una modalidad fuera para un core y cuáles se pueden representar con una adaptación documentada?
3. ¿Qué casos mínimos deben pasar y qué parte del manual de usuario debe estar lista para declarar terminada la incorporación?
4. ¿Qué incluimos en la estimación de la aplicación separada y cómo documentamos su grado de certeza?
