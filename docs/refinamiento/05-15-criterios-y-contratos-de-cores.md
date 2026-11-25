# Criterios de core y contratos preliminares de Movilidad

Punto 5.15 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Se proponen dos contratos de cotización simulados, con diferencias que obligan a probar el contrato común del backend. Las denominaciones, los montos y los códigos son sintéticos. [Ficha de producto y alineación](05-14-ficha-de-producto-y-esfuerzo-de-incorporacion.md) · [Contrato y reintentos](02-05-contrato-de-cores-precio-y-confirmacion.md) · [Moneda y desglose](02-07-moneda-y-desglose-del-premio.md)

## Referencia de la SSN

El punto 25.1.2 del RGAA trata la documentación contractual. Incluye el detalle de riesgos y capitales asegurados, las franquicias, carencias y plazos aplicables, y el desglose del premio. El punto 26.1.6 explica qué conceptos considera la tarifa y qué se adiciona al formar el premio final. La SSN también explica la diferencia entre prima y premio en su guía para asegurados. [RGAA actualizado, puntos 25.1.2 y 26.1.6](https://www.argentina.gob.ar/normativa/nacional/238219/actualizacion) · [Guía de la SSN](https://www.argentina.gob.ar/superintendencia-de-seguros/portal-ciudadano-0/contrato-de-seguro)

La PoC toma ese criterio de transparencia para representar una **cotización preliminar**. Conserva el total en ARS, su base de pago, la cobertura, los límites y la franquicia cuando el core los informa. También distingue vigencia de la oferta y período de cobertura. Un premio informado solo como total se identifica como tal. El desglose de una futura solicitud se conserva en su propia etapa. La validación de estos JSON acredita estructura de datos para dos contratos ficticios. La revisión normativa de un plan real y la emisión de pólizas quedan fuera de este alcance.

## Dos cores para validar el contrato común

| Aspecto | Core A recuperable | Core B volátil |
| --- | --- | --- |
| Entrada | Código de producto, plan de suma fija, productor y zona. | Modalidad, zona, suma explícita, fecha del pedido y descuento opcional autorizado por perfil. |
| Precio | ARS en centavos enteros y desglose completo según los componentes definidos para el simulador. | ARS como texto decimal y premio total. |
| Vigencia y recuperación | ID externo y límite de aceptación declarado. La consulta puede devolver el registro o su estado. | Sin ID de cotización ni límite externo declarado en este borrador. La plataforma conserva los parámetros permitidos y, si se aprueba, aplica el límite interno del 4.10. |
| Coberturas | Códigos propios, estados explícitos y franquicia fija cuando se informa. | Códigos distintos, inclusión opcional y franquicia porcentual con base de aplicación cuando se informa. |

Se propone **un criterio común y dos contratos externos**. El backend envía el mismo escenario desde la ficha de Movilidad, con provincia, CP, modalidad y suma elegida. El adaptador A traduce la suma a un código y el B la envía como monto. Para el ejemplo, ambos ofrecen cobertura sobre el bien hasta ARS 100.000. Así se puede comprobar, sobre el mismo riesgo, que el backend traduce las dos respuestas sin perder diferencias. El contrato preliminar acepta CP de cuatro cifras en ambos simuladores. La elección entre CP y CPA para el producto sigue pendiente en el 1.2.

El modelo común conserva el plan, la cobertura efectiva y su detalle, estado de inclusión, suma, franquicia, asistencia, moneda, premio, base de pago y nivel de desglose cuando corresponda. Una cobertura omitida queda como **No informada**. Una exclusión explícita se puede mostrar como **No incluida**. Un valor cero informado se conserva. La huella y los IDs internos se generan en el backend. El código de plan de B identifica el tipo de plan y no sirve como identificador externo de esa cotización.

Un tercer core aporta si prueba una capacidad distinta y aprobada, por ejemplo una respuesta asincrónica con referencia de seguimiento y entrega posterior de ofertas. Para el Nivel 1 propongo conservar los dos del acta. Se puede simular demora o error en cualquiera para probar el 1.4 sin agregar otro contrato.

## Esquemas y ejemplos

Los dos archivos usan [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12). Las definiciones por operación están en `$defs`. El esquema raíz permite verificar ejemplos completos. El diseño HTTP, la recuperación operativa de A y la confirmación de venta se especificarán en sus respectivos contratos. En estos borradores, las fechas usan `format`; al implementar se debe configurar su validación semántica.

### Core A

[Esquema de Core A](contratos/core-a-cotizacion.schema.json). Para la recuperación se conserva `cotizacionId`. `codPlan: SA100` representa en los datos simulados una suma de ARS 100.000 para daños a la bicicleta. El código de productor se resuelve en el backend.

Solicitud de cotización.

```json
{
  "codProducto": "BIC-01",
  "codPlan": "SA100",
  "codProductor": "SIM-A-STD",
  "zona": { "provincia": "BA", "cp": "1838" }
}
```

Respuesta de cotización.

```json
{
  "estado": "COTIZADA",
  "cotizacionId": "SIM-A-Q001",
  "validaHasta": "2026-11-25T18:00:00-03:00",
  "moneda": "ARS",
  "planes": [
    {
      "idPlan": "SA100-BASE",
      "denominacion": "Esencial",
      "periodoDias": 30,
      "basePago": "POR_PERIODO",
      "premioCentavos": 490000,
      "desglose": {
        "primaPura": 340000,
        "gastosProduccion": 50000,
        "gastosExplotacion": 40000,
        "recargos": 10000,
        "impuestos": 30000,
        "sellados": 20000
      },
      "coberturas": [
        { "codigo": "DAN-BIC", "estado": "INCLUIDA", "descripcion": "Daños a la bicicleta", "limiteCentavos": 10000000, "franquiciaCentavos": 1000000 },
        { "codigo": "AP-MUN", "estado": "INCLUIDA", "descripcion": "Fractura de muñeca por caída" },
        { "codigo": "RC", "estado": "NO_INCLUIDA" }
      ],
      "asistencias": ["Traslado particular ante daños a la bicicleta"]
    }
  ]
}
```

### Core B

[Esquema de Core B](contratos/core-b-cotizacion.schema.json). `quotedOn` es la fecha del pedido a este core. Volver a cotizar puede requerir actualizarla. La solicitud de ejemplo conserva el mismo escenario que A y omite el descuento para el perfil estándar.

Solicitud de cotización.

```json
{
  "assetKind": "CYCLE",
  "area": { "region": "BUE", "postalCode": "1838" },
  "amount": "100000.00",
  "currency": "ARS",
  "quotedOn": "2026-11-25"
}
```

Respuesta de cotización.

```json
{
  "result": "OK",
  "currency": "ARS",
  "options": [
    {
      "tier": "BASIC",
      "label": "Movilidad",
      "total": "5100.00",
      "termDays": 30,
      "paymentBasis": "TERM",
      "benefits": [
        { "key": "CYCLE_DAMAGE", "label": "Daños a la bicicleta", "active": true, "limit": "100000.00", "deductiblePct": 10, "deductibleBasis": "LOSS" },
        { "key": "FRACTURES", "label": "Fracturas en las zonas indicadas en el plan", "active": true },
        { "key": "LIABILITY", "label": "Responsabilidad civil", "active": true, "limit": "100000.00" }
      ],
      "services": ["Ambulancia ante lesiones por caída"]
    }
  ]
}
```

El backend mostraría ARS 4.900 y ARS 5.100 para períodos de 30 días en este ejemplo, con las diferencias de cobertura visibles. El desglose de A suma su total según la regla del simulador. El total de B se mostraría como **Solo total**. La diferencia de franquicias y coberturas se conserva según el [comparador del 1.3](01-03-comparacion-de-ofertas.md).

## Criterios candidatos de aceptación

1. Los dos adaptadores pueden procesar una bicicleta con el mismo CP y suma de riesgo y devolver ofertas en el modelo común. Una suma fija codificada y un monto explícito se conservan como valores monetarios comparables.
2. El resultado de A puede vincularse a su ID externo y al vencimiento informado. El de B queda trazado por los identificadores e insumos permitidos de la plataforma.
3. Cada core distingue rechazo de negocio, datos inválidos, modalidad no soportada y falla temporal. Un timeout de transporte sin respuesta se registra como estado del intento del backend. El usuario conserva la oferta válida del otro core si se aprueba el flujo parcial del 1.4.
4. El adaptador verifica el premio y el desglose según lo que el contrato permita. Las pruebas también cubren coherencia entre importes, vigencias y mapeos. La validación estructural se complementa con pruebas de esas reglas.

Quedan para segunda lectura los campos condicionales de bicicleta eléctrica, los códigos y límites de prueba, qué habilita la venta simulada y cómo consulta cada core los casos de confirmación. Las solicitudes de venta, pólizas y pagos tienen reglas separadas de estos esquemas de cotización.
