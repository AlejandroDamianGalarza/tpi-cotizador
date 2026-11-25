# Segunda lectura de fichas y contratos de core

Punto 5.15 del refinamiento del TPI. **Estado: segunda lectura registrada, pendiente de actualizar los contratos.** Este documento recoge las decisiones sobre configuración de fichas, equivalencias, vigencia por plan, selección de oferta y manejo de fallas. Los [esquemas preliminares de los cores](05-15-criterios-y-contratos-de-cores.md) siguen como referencia para revisar los cambios.

## 1 Ficha base y requisitos propios de cada core

La ficha común sirve de base para la familia Movilidad. Los campos adicionales pueden incorporarse mediante un anexo o una revisión de la ficha. Un dato puede ser obligatorio para una combinación de core, plan y etapa, aunque no esté en la ficha base. El frontend presenta los campos condicionales que el backend expone desde una configuración validada. El contrato externo informa la necesidad del dato y el anexo define cómo la plataforma puede obtenerlo, validarlo y mostrarlo.

Si aparece un requisito nuevo, el funcional o responsable de producto define su significado y el uso permitido. Los responsables técnicos del core y de la plataforma revisan el mapeo, la incompatibilidad detectada y la contingencia. El anexo registra campo, etapa, origen, validación, combinaciones afectadas y evidencia de la revisión. Si falta una forma aprobada de obtener o alimentar el dato, la revisión de configuración queda bloqueada para su publicación.

Si una incompatibilidad se descubre en ejecución, el sistema registra el caso, avisa al responsable y bloquea la versión o combinación afectada hasta que el responsable de configuración revise y corrija el problema. El alcance del bloqueo debe poder indicarse por core, plan o perfil cuando la incidencia esté localizada. Las ofertas válidas de otras combinaciones pueden seguir disponibles.

## 2 Categorías y datos sintéticos comparables

La plataforma propone las categorías y registra qué código y alcance de cada ficha ficticia se agrupan bajo ellas. Esta inferencia se revisa antes de incorporarla al mapeo aprobado. Una categoría compartida ayuda a comparar. La equivalencia de condiciones requiere además comprobar riesgo cubierto, suma, franquicia y vigencia aplicables.

| Categoría propuesta | Core A | Core B | Diferencia visible |
| --- | --- | --- | --- |
| Daños al bien | `DAN-BIC` | `CYCLE_DAMAGE` | Ambos ejemplos tienen suma de ARS 100.000. A aplica una franquicia fija de ARS 10.000 y B una de 10 % del daño. |
| Accidentes personales › Fracturas | `AP-MUN` | `FRACTURES` | A informa fractura de muñeca por caída. B informa fracturas en las zonas detalladas por el plan. Cada detalle se conserva. |
| Responsabilidad civil | `RC` | `LIABILITY` | El plan base de A la excluye. B la incluye hasta ARS 100.000. |
| Asistencias | Traslado ante daños a la bicicleta | Ambulancia ante lesiones por caída | Ambos son servicios de asistencia y se muestran con su condición de activación. |

Datos candidatos para la demo. Los tres planes parten de la misma bicicleta, provincia, CP, perfil y suma de ARS 100.000 para daños al bien. Los premios corresponden al mismo período de cobertura de 30 días. El plazo para aceptar el precio se indica por separado.

| Core y plan sintético | Premio por período | RC | Plazo de aceptación propuesto |
| --- | ---: | --- | --- |
| A · Esencial | ARS 4.900 | No incluida | 1 hora, informado por el core |
| A · Ampliado | ARS 6.500 | Hasta ARS 200.000 | 7 días, informado por el core |
| B · Movilidad | ARS 5.100 | Hasta ARS 100.000 | 24 horas como límite interno de ejemplo, si la regla se aprueba |

La ficha de cada plan debe explicar por qué su plazo difiere. Una condición ligada a un valor volátil puede justificar un plazo más corto en los datos de prueba, siempre que se registre como regla explícita. Los ejemplos muestran coherencia bajo condiciones controladas y se identifican como sintéticos.

## 3 Estado actual y recotización desde la interfaz

El backend devuelve el estado actual de cada oferta. Un plan vigente se puede recuperar y reutilizar mientras siga elegible. Si otro plan venció, el frontend muestra ese estado y ofrece **Actualizar cotización**. La acción conserva la referencia pública y crea una nueva instancia interna con motivo de vencimiento. El adaptador usa la referencia previa cuando el core admite consultarla o ejecuta un pedido nuevo con los datos guardados y la fecha que corresponda. Si falta un dato, el flujo solicita solo lo necesario. La persona puede revisar el nuevo resultado antes de avanzar.

El [esquema inicial de Core A](contratos/core-a-cotizacion.schema.json) declara `validaHasta` para toda la respuesta. Para representar Esencial y Ampliado con plazos distintos hay que agregar una vigencia por plan y acordar si existe además un límite general. El [esquema inicial de Core B](contratos/core-b-cotizacion.schema.json) usa `termDays` para el período de cobertura. Si su contrato sigue sin informar vencimiento, la vida de la oferta sale de una regla de la plataforma por core, plan y perfil.

## 4 Elección y detalle del premio

El cliente compara las opciones disponibles y elige un plan habilitado para su caso. En el detalle se muestra el premio estimado y el desglose disponible con estado **Completo**, **Parcial** o **Solo total**. Si el core informa que pueden agregarse sellados o tasas al solicitar, la pantalla lo aclara. Cualquier ajuste por zona o perfil se muestra según la regla aprobada y la evidencia del core. Si cambia el premio al avanzar, el importe anterior y el nuevo quedan registrados. La revisión del importe resultante sigue el criterio candidato del [2.7](02-07-moneda-y-desglose-del-premio.md) antes de confirmar la venta simulada.

## 5 Éxito, rechazo y seguimiento de fallas

Recibir un premio cuenta como cotización exitosa cuando el código está documentado como éxito para esa operación y la oferta cumple las validaciones acordadas para su plan, moneda y condiciones. La ficha técnica del core mantiene las equivalencias de sus códigos por operación. Un mensaje con premio y código desconocido se marca inválido. Una respuesta fatal o desconocida se registra y se deriva al responsable del core o de configuración cuando pueda identificarse, con la referencia necesaria para el seguimiento.

| Ejemplo | Tratamiento esperado |
| --- | --- |
| El core exige edad y la configuración aprobada sabe obtenerla en esa etapa | El backend expone el campo cuando corresponde. El frontend lo solicita y el backend valida. |
| El plan rechaza por edad según una regla documentada | Registrar el rechazo de elegibilidad para ese plan. Mantener la posibilidad de cotizar o comprar otro plan que sí resulte elegible y esté habilitado para venta. |
| El core exige edad y la ficha carece de un mapeo aprobado | Registrar la incompatibilidad, alertar y bloquear la configuración afectada hasta su revisión. |
| El plan devuelve premio con código desconocido | Invalidar esa respuesta y conservarla para diagnóstico según las reglas de auditoría. Una oferta válida de otro core puede seguir su curso. |

Un filtro adicional de la ficha o del perfil se define como regla de negocio versionada, con condiciones y vigencia explícitas. El backend lo aplica y la interfaz refleja el resultado. Una regla experimental de edad requiere aprobación antes de habilitarse. El análisis comercial puede revisar rechazos y conversiones agregadas por motivo y tramo, cuando el dato esté disponible y autorizado. Una caída de ventas motiva revisar la ficha o evaluar una variante temporal. Para atribuir la caída a la edad hace falta evidencia adicional.

## Cambios pendientes para cerrar los contratos

- Definir la estructura de anexos y un primer campo condicional con su etapa, mapeo y contingencia.
- Actualizar el contrato de A para vigencia por plan y definir cómo resuelve el backend el estado efectivo de cada oferta. Acordar qué se recupera del core y qué se vuelve a cotizar.
- Definir los casos de plan sin oferta, precio inconsistente, código desconocido y bloqueo localizado. Acordar en qué momento se alerta y quién revisa la configuración.
- Cerrar qué planes pueden venderse en cada core y qué acuse confirma la **solicitud de venta**. Esta confirmación se rige por el [4.12](04-12-atribucion-de-venta-y-estado-de-solicitud.md) y es una operación distinta de una cotización exitosa.
