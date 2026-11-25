# Criterios y prueba de rendimiento del cotizador

Punto 5.13 del refinamiento del TPI. **Estado: borrador para segunda lectura.** Se proponen criterios para comparar la experiencia pública del cotizador y para medir la capacidad local de la PoC. El acta fija una línea base de **al menos 5 flujos de cotización por segundo durante 3 minutos, con menos de 1 % de errores atribuibles a la plataforma**. Este punto precisa cómo medirla y se relaciona con los [resultados parciales del 1.4](01-04-resultados-parciales-y-ofertas-tardias.md) y los [contratos de core del 2.5](02-05-contrato-de-cores-precio-y-confirmacion.md).

## Referencias públicas del mercado

| Referencia | Qué publican y qué permite comparar |
| --- | --- |
| [BBVA Argentina, seguro para bicicletas](https://www.bbva.com.ar/personas/productos/seguros/bicicleta.html) | Distingue los recorridos de clientes y no clientes y promociona la contratación en minutos desde Banca Online. Sirve para comparar canales y pasos del recorrido. |
| [123Seguro Argentina](https://123seguro.com.ar/) | Promociona cotizar en segundos, comparar precios y coberturas y elegir entre contratación online y asesoramiento. Sirve para identificar qué acciones forman parte de la experiencia pública. |
| [CDL y AWS, operación de cotización de seguros](https://aws.amazon.com/blogs/apn/10-million-insurance-quotes-per-hour-how-cdl-and-aws-are-tackling-the-cost-of-living-crisis/) | Documentan un caso británico en el que una solicitud del consumidor se multiplica en consultas a intermediarios y aseguradoras. Sirve para distinguir un flujo del cliente de las llamadas que genera el orquestador. |

Las referencias argentinas consultadas describen rapidez y facilidad del recorrido. Sus páginas citadas dejan abiertos el volumen de tráfico, los percentiles de latencia y la proporción de errores. El caso británico usa otra escala y otro mercado. La PoC puede adoptar unidades comparables y medir sus propios resultados. Un contraste técnico con una aseguradora requeriría métricas autorizadas obtenidas bajo condiciones equivalentes de producto, carga, entorno y alcance.

## Qué medir en Movilidad

| Criterio | Medición candidata |
| --- | --- |
| Experiencia del cliente | Tiempo desde **Cotizar** hasta la primera oferta válida visible y tiempo hasta que se resuelven los intentos previstos o vence su espera. Registrar qué ve el usuario si falta una respuesta. La observación en pantalla se puede hacer en una prueba funcional aparte. |
| Capacidad del orquestador | Flujos iniciados y finalizados, resultados por flujo y ritmo efectivo de entrada. Contar por separado las solicitudes o consultas de estado a la API y los intentos contra cada core. Informar p50 y p95 de los tiempos medidos. |
| Salud de cada integración | Latencia, rechazos de negocio, respuestas sin oferta, errores técnicos y timeouts por core. Poder relacionar cada resultado con el flujo que lo originó. |
| Saturación | Registrar concurrencia observada y uso de recursos que ayuden a explicar el resultado, por ejemplo CPU, memoria o espera de conexiones si se mide. |

La separación toma como referencia las señales de latencia, tráfico, errores y saturación de Google SRE. El tiempo de primera oferta, el tiempo de cierre del comparador y la duración de la llamada a cada core responden preguntas distintas. [Google SRE, monitoreo de sistemas distribuidos](https://sre.google/sre-book/monitoring-distributed-systems/)

## Protocolo candidato para el Nivel 1

1. **Definir el flujo.** Una iteración representa una solicitud válida de Movilidad. Se envía al orquestador, se ejecutan las consultas previstas a los dos cores simulados y se verifica el estado final esperado. Si el resultado llega por etapas, la iteración incluye la consulta necesaria para observarlo dentro de una espera definida. La venta simulada y su idempotencia tienen una prueba separada.
2. **Fijar el entorno.** Documentar versiones, recursos de la máquina, base de datos y conjunto sintético de entradas. Como punto de partida, el anexo muestra demoras de `200 ms` y `350 ms` para Core A y B y cero errores inyectados. Documentar también el timeout y los criterios de cierre antes de ejecutar.
3. **Aplicar la carga.** Programar una llegada constante de 5 flujos por segundo durante 180 segundos, 900 inicios previstos. Registrar cuántos efectivamente empiezan, cuántos terminan, las iteraciones descartadas y el tiempo permitido para que finalicen las últimas. Si el generador no sostiene la tasa, el intento no demuestra la línea base. Los usuarios virtuales son recursos para generar esa carga y no equivalen a clientes activos en producción. [Grafana k6, tasa de llegada constante](https://grafana.com/docs/k6/latest/using-k6/scenarios/executors/constant-arrival-rate/) · [Iteraciones descartadas](https://grafana.com/docs/k6/latest/using-k6/scenarios/concepts/dropped-iterations/)
4. **Contar errores por flujo.** Como regla candidata, dividir los flujos con al menos una falla atribuible a la plataforma por los flujos válidos iniciados. Una solicitud aceptada que se pierde, una oferta mal asociada o un resultado funcionalmente incorrecto cuentan aunque la API devuelva un estado HTTP exitoso. Clasificar por separado el rechazo de negocio, el fallo de core y la causa desconocida. Informar también cuántos flujos quedaron sin oferta utilizable, por cualquier causa. Los casos sin clasificar y los flujos inconclusos se revisan antes de declarar la prueba aprobada.
5. **Verificar una degradación controlada.** En una ejecución aparte, hacer lento a un core o simular un fallo. Si el comportamiento del 1.4 entra en la entrega, comprobar que la otra oferta válida puede aparecer, que se registra el motivo y que el resultado es trazable. Este escenario se reporta por separado del ensayo normal.

El anexo propone para el Nivel 2 una tasa de 10 flujos por segundo durante 10 minutos y p95 inferior a 2 segundos bajo latencias simuladas documentadas. Para aplicar ese p95 hay que indicar qué intervalo mide. En el Nivel 1, la propuesta es **informar** el tiempo de primera oferta y de cierre con su alcance y sus datos, sin sumar un nuevo umbral de latencia al compromiso del acta.

## Preguntas para la segunda lectura

1. ¿Qué resultado y tiempo máximo dan por terminado un flujo de prueba cuando ambos cores responden normalmente? ¿Cómo observamos la primera oferta si el comparador actualiza resultados por etapas?
2. ¿Qué demoras y timeouts simulan una respuesta normal y una degradada? ¿Qué respuesta del usuario cuenta como válida en cada escenario?
3. ¿Qué datos agregados y autorizados podríamos conseguir en la empresa para comparar el mismo flujo, intervalo, canal y producto, y qué parte conviene tomar solo como referencia de experiencia?
