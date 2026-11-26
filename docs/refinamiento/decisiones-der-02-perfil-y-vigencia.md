# Decisiones del DER sobre perfiles y vigencia de ofertas

Segunda entrega del TPI. Este registro documenta las decisiones 2 y 3 para el esquema de datos, el DER y el diccionario. Se apoya en los refinamientos de [administración de perfiles](03-08-administracion-de-perfiles-y-acceso-de-demostracion.md), [versiones en curso](03-09-versiones-en-curso-y-recuperacion.md), [vencimiento](04-10-vencimiento-y-recotizacion-de-ofertas.md) y [contratos de los cores](05-15-segunda-lectura-de-fichas-y-cores.md).

## Decisión 2 Perfiles preparados y revisión aplicada

**Aprobada para el Nivel 1.** Se usan perfiles comerciales preparados, como el perfil estándar y un convenio sintético de Banco A. Cada cotización conserva una referencia a la revisión de perfil que se aplicó. Esa revisión debe seguir disponible para consultar qué condiciones comerciales se usaron, aun si después cambia el perfil activo. El DER distingue el perfil comercial de su revisión y del actor que inició la cotización. La administración por entidad, las plantillas editables y la auditoría administrativa completa quedan para niveles posteriores.

## Decisión 3 Vigencia por oferta

El plazo de aceptación pertenece a cada **oferta**. Puede provenir de una regla de negocio en la ficha y del límite que el core devuelve para esa oferta. Si el core informa un vencimiento, ese instante es el máximo que la plataforma puede reconocer. Aplicar una regla comercial más corta permite limitar la oferta desde la plataforma.

| Situación | Vencimiento que se aplica | Evidencia |
| --- | --- | --- |
| La ficha carece de plazo y el core informa su límite | Usar la fecha del core. | Guardar el valor recibido y su origen. |
| La ficha define un límite igual o anterior al del core | Usar el límite de la ficha. | Guardar ambos valores y la revisión de configuración aplicada. |
| La ficha define un límite posterior al del core | Usar el límite del core y alertar al responsable de configuración durante la ejecución. | Guardar ambos límites, el valor efectivo y el motivo del ajuste. |
| La ficha define un plazo y el core no informa vencimiento | Aplicar el límite interno calculado desde la ficha. | Identificarlo como límite de la plataforma. |
| Faltan ambos plazos o el dato recibido es inválido | Pendiente de acordar. Se propone tratar la vigencia como desconocida y requerir una validación antes de confirmar. | Registrar la causa por separado de un vencimiento informado. |

Ejemplo sintético. Si la ficha indica las 19:00 y el core permite aceptar hasta las 18:00, el cliente ve las 18:00 y se registra un aviso para el responsable. Si la ficha indica las 17:00, se muestra ese límite y se conservan las dos fechas. El backend usa el límite efectivo al validar la confirmación. El plazo para aceptar el premio queda separado del período de cobertura.

## Consecuencias para el modelo y la verificación

- En la cotización se registra la **revisión de perfil aplicada**. El catálogo puede publicar otra revisión para nuevas operaciones.
- En la oferta se conservan el **límite de ficha**, el **límite del core** y el **vencimiento efectivo**, con el dato de origen y la regla aplicada. Un límite ausente se representa por separado de uno inválido.
- La alerta de recorte debe relacionar oferta, plan, core, revisión de ficha o configuración, ambos límites, valor efectivo y momento del hecho. Para el Nivel 1 se propone que quede como **aviso consultable por el responsable**, con un caso sintético que permita verificar que lo recibe o lo ve. El canal concreto y el seguimiento del aviso quedan por definir.
- Los esquemas de core deberán expresar correctamente la vigencia. En el borrador, Core A usa `validaHasta` a nivel de respuesta. Para soportar plazos distintos por plan hay que ajustar el esquema y definir el tratamiento de un eventual límite general. Core B utiliza `termDays` para el período de cobertura. El límite interno de aceptación se registra en la oferta cuando B carece de fecha externa.

Queda por acordar desde qué instante se calcula un plazo de ficha expresado como duración, qué canal usa la alerta y qué resultado se muestra cuando falta una vigencia válida. Estos puntos deben quedar en la especificación y en los casos de prueba correspondientes.
