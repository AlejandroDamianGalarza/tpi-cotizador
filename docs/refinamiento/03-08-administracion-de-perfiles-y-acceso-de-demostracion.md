# Administración de perfiles y acceso de demostración

Punto 3.8 del refinamiento del TPI. **Estado: borrador para segunda lectura.** El recorrido inicial usa el perfil comercial anónimo. La administración por entidad debe poder crear perfiles a partir de una base común y limitar quién puede editarlos o usarlos. La demostración podrá representar esas identidades con accesos rápidos. Este punto continúa el [refinamiento de actores y perfiles](01-01-actores-identidad-y-perfiles.md) y el de [configuración de cotización](01-02-datos-y-configuracion-de-cotizacion.md).

## Perfil inicial y alcance de edición

El visitante empieza como anónimo y puede cotizar con el perfil comercial estándar. Identificarlo después puede habilitar otros perfiles, según el convenio y la elegibilidad que se definan. Queda pendiente si el visitante conserva la oferta previa, vuelve a cotizar con el nuevo perfil o elige entre las alternativas que estén vigentes.

La edición planteada corresponde a la **configuración comercial**, que puede incluir datos requeridos, condiciones, planes, sumas y mapeos internos. Queda por aclarar si la base anónima es única para toda la plataforma o si cada entidad también tiene una variante desde la que crea sus perfiles. Como criterio inicial, propongo una base estándar común y perfiles propios de cada entidad derivados de esa base. El visitante usa las condiciones resueltas para su contexto. Los permisos administrativos definen quién puede cambiar la configuración.

## Opciones de plantilla y versionado

| Opción | Efecto sobre los perfiles |
| --- | --- |
| Copiar una versión de la plantilla | El nuevo perfil parte de una configuración conocida y conserva la referencia a su origen. Los cambios posteriores de la plantilla requieren una adopción explícita. |
| Heredar cambios de la plantilla | Los perfiles pueden incorporar cambios de base según reglas de herencia. Hay que definir precedencia, propagación, vigencia, compatibilidad y reversión. |

Para la primera versión de la administración propongo **copiar una plantilla versionada al crear el perfil**. Un cambio se prepara en borrador y, si se aprueba publicarlo, genera una revisión nueva. Las cotizaciones previas conservan el perfil y la versión que se les aplicó. Queda por decidir quién puede publicar o revertir un perfil y qué pasa con una cotización en curso al entrar en vigencia otra revisión.

La versión del perfil comercial y la versión del producto tienen ciclos distintos. También queda por definir a qué versión de Movilidad se asigna cada perfil al inicio y quién puede cambiar esa asignación. Crear un perfil a partir de una plantilla no implica activarlo ni habilitar una nueva versión del producto.

## Administración por entidad y CRM

| Rol propuesto | Responsabilidad y alcance |
| --- | --- |
| Administrador principal de plataforma | Crea y administra las entidades ficticias y define qué proveedor de acceso se asocia a cada una. Queda pendiente si también publica la base común o aprueba cambios comerciales de las entidades. |
| Administrador de Banco A o Banco B | Completa y edita borradores de los perfiles de su propia entidad, a partir de la base autorizada. Los perfiles del otro banco quedan fuera de su ámbito de edición. Falta definir quién publica los cambios. |
| Responsable del CRM | Puede consultar los perfiles bancarios que se le habilitaron. Puede limitar qué perfiles quedan disponibles para cada operador del CRM dentro de lo autorizado para su organización. Los cambios comerciales del perfil pertenecen a la entidad responsable. |
| Operador del CRM | Puede usar los perfiles que le asignaron cuando el cliente sea elegible. El permiso de ver o usar un perfil y el permiso de delegarlo son independientes. |

Queda por precisar qué datos puede ver el CRM. Como punto de partida, mostrar condiciones comerciales necesarias para cotizar y mantener los códigos internos y ajustes de integración en su ámbito administrativo. El backend verifica entidad, rol y asignación para cada acción. Una asignación del CRM puede restringir las opciones de sus operadores dentro del conjunto habilitado para el CRM.

## Proveedor de acceso y botonera de demostración

La asociación entre entidad y proveedor de acceso se administra de forma independiente del perfil comercial. Un proveedor identifica el origen del acceso. La entidad y los permisos del actor se resuelven por separado de las condiciones comerciales del cliente. Quedan por definir las reglas de asociación, qué ocurre si una entidad cambia de proveedor y qué datos de identidad se confiarán a cada integración. Los bancos ficticios de este punto son entidades comerciales y conservan su distinción de los cores que cotizan.

Para la demo propongo una **botonera de identidades sintéticas con un botón «Ingresar»**. Puede empezar con «Continuar como visitante», «Cliente Banco A» y «Operador Banco A». Si queremos revisar el aislamiento entre entidades y los permisos del CRM, se pueden preparar identidades del Banco B, un responsable de CRM, un operador con asignación limitada y un administrador principal. Cada botón selecciona una identidad predefinida para el entorno de demostración. El backend toma sus permisos de los datos de prueba, no de un rol libre enviado por el frontend.

El alta por DNI y contraseña queda como opción posterior de acceso. Para la demostración, las identidades, entidades y perfiles pueden venir preparados. Seleccionar un administrador en la botonera permite demostrar las acciones que se aprueben para ese rol. La creación de entidades desde pantalla, el editor completo de perfiles y los proveedores reales quedan sujetos a una decisión de alcance. El contexto de auditoría registra si el acceso fue simulado.

## Preguntas para la segunda lectura

1. ¿Quién mantiene el perfil anónimo base? ¿Necesita cada entidad una variante propia sin identificar al cliente?
2. ¿Qué campos puede configurar un banco y cuáles quedan definidos por la plataforma? ¿Quién publica y revierte cada revisión?
3. ¿Qué ve el responsable del CRM y hasta dónde puede asignar perfiles a sus operadores? ¿Cómo se revoca una asignación?
4. ¿Qué perfil y versión de Movilidad recibe cada identidad de la demo al iniciar? ¿Qué cambio puede verse durante el recorrido?
5. ¿Qué acciones del administrador principal tienen que verse en la primera demo y cuáles pueden quedar representadas con datos ya preparados?
