# Acta de Constitución del Proyecto

> Copia Markdown de la primera entrega. Documento de origen `Acta_Constitucion_TPI_Alejandro_alineada_v1.docx`.

## Nombre del Proyecto

Plataforma versionada de cotización de seguros con integración canónica y medición centralizada.

## Integrantes del Proyecto

Alejandro Galarza

## Justificación y Propósito del Proyecto

La comercialización de productos de seguros suele derivar en frontends, BFF y backends separados para variantes o canales similares. Esto favorece la duplicación de recorridos, integraciones repetidas con distintos cores y datos comerciales fragmentados.

El propósito es validar una mejora arquitectónica acotada que permita reutilizar un mismo recorrido, incorporar variantes y cambiar de core mediante contratos estables, sin construir una plataforma universal ni reemplazar sistemas existentes.

El beneficio esperado es reducir el tiempo de salida de una primera versión, centralizar la trazabilidad de cotizaciones y ventas simuladas y disponer de evidencia medible sobre reutilización, versionado y comportamiento técnico.

## Descripción del Proyecto de Alto Nivel

Se construirá un prototipo para la familia sintética Movilidad, con las modalidades bicicleta y bicicleta eléctrica. Cliente y operador utilizarán un frontend React compartido con diferencias de permisos y capacidades, no aplicaciones duplicadas.

La solución se implementará en Node.js como monolito modular NestJS, con PostgreSQL, una API de experiencia, un contrato canónico de cotización y adaptadores para dos cores simulados: uno con identificador recuperable y desglose completo, y otro volátil con referencia por parámetros y premio total o parcial.

Las definiciones de producto y las asignaciones serán versionadas. Movilidad v1 y v2 estarán previamente instaladas; la activación y el rollback cambiarán la versión asignada sin descargar ni ejecutar código remoto.

El desarrollo aplicará Spec-Driven Development asistido por IA. Las fuentes serán sintéticas o sanitizadas, el modelo propondrá contexto y especificaciones, y toda aprobación seguirá bajo revisión humana.

## Entregables Principales

- Frontend compartido para cliente y operador.
- Backend NestJS modular y base PostgreSQL.
- Contrato canónico y adaptadores para dos cores simulados.
- Catálogo de producto, Movilidad v1 y v2, activación y rollback.
- Cotización, comparación de ofertas y venta simulada idempotente.
- Registro central de versiones, core, canal, actor, errores y resultados.
- Métricas técnicas y consultas de cotizaciones y ventas.
- Especificaciones SDD, pruebas, manual de ejecución y demostración.

## Exclusiones

- Integraciones, credenciales, contratos o datos reales sin autorización.
- Emisión legal de pólizas, pagos, cobranzas y facturación.
- Cálculo legal de impuestos, validación real ante la SSN o certificación normativa.
- Microservicios productivos, Kubernetes, alta disponibilidad o migración de sistemas existentes.
- Carga dinámica de librerías, microfrontends remotos o ejecución de código arbitrario.
- Uso de información confidencial, aprobación automática por IA o generación de cambios sin revisión.

## Objetivos del Proyecto

Objetivo general: demostrar de extremo a extremo que un monolito modular, un contrato canónico acotado, adaptadores y definiciones versionadas permiten incorporar variantes con menos duplicación y mayor trazabilidad.

- Completar en ocho semanas un Nivel 1 defendible, estimado en 40 a 50 horas; avanzar al Nivel 2 solo si el primero queda probado y documentado.
- Ejecutar el mismo recorrido desde cliente y operador y soportar dos modalidades sin duplicar la aplicación.
- Integrar dos contratos de core diferentes mediante el mismo puerto canónico.
- Activar Movilidad v2 y volver a v1 conservando la versión usada por cada operación.
- Registrar una venta simulada sin duplicados y reconstruir cada operación mediante identificadores correlacionados.

## Indicadores de Valor

Reutilización: un frontend y un recorrido común para ambos actores y modalidades.

Integración: dos cores con contratos distintos conectados mediante el mismo puerto y pruebas contractuales.

Trazabilidad: toda cotización y venta conserva producto, modalidad, versión, core, canal, actor e identificadores internos.

Consistencia: cero ventas duplicadas ante la repetición de una misma solicitud.

Capacidad local: línea base reproducible de al menos 5 flujos de cotización por segundo durante 3 minutos, con menos de 1 % de errores atribuibles a la plataforma.

Time to market: documentar el tiempo y los archivos modificados para incorporar la segunda modalidad y compararlos con la creación de una aplicación separada.

## Enfoque de Gestión / Ciclo de Vida del Proyecto

Enfoque ágil e incremental por niveles acumulativos, con alcance fijo para el nivel comprometido y revisión al cierre de cada incremento. El Nivel 1 es el compromiso externo; el Nivel 2 incorpora autenticación real, contratos y concurrencia solo si existe disponibilidad.

Ciclo: refinar y aprobar especificaciones; diseñar contratos; implementar un recorrido vertical; integrar el segundo core; incorporar versiones y métricas; ejecutar pruebas; estabilizar; documentar y demostrar.

La IA asiste el refinamiento, pero no reemplaza la aprobación humana. Un nivel solo se considera cerrado si la aplicación puede iniciarse, el recorrido funciona, las pruebas críticas pasan y la documentación coincide con el comportamiento.

## Hitos Principales

Semana 1: alcance, constitución SDD, corpus sanitizado, modelo canónico y arquitectura.

Semana 2: recorrido vertical completo con Core A.

Semana 3: Core B, normalización e identidad interna.

Semana 4: modalidades y versiones sin duplicar el frontend.

Semana 5: venta idempotente, métricas, trazabilidad y cierre del Nivel 1.

Semana 6: Keycloak, ciclo de versiones y concurrencia del Nivel 2, si hay tiempo.

Semana 7: pruebas, documentación y ampliaciones seguras.

Semana 8: correcciones, demostración y defensa.
