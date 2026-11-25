# Propuesta de Trabajo Práctico Integrador

> Copia Markdown de la primera entrega. Documento de origen `Anexo_Tecnico_Propuesta_TPI_Alejandro_Galarza_v2_2.docx`. Se retiró el DNI de la portada para publicar esta copia.

Plataforma versionada de cotización de seguros con integración canónica y medición centralizada

| Campo | Valor |
| --- | --- |
| Versión | 2.2 |
| Estado | Borrador corregido para revisión y entrevista |
| Modalidad | Proyecto individual |
| Integrante | Alejandro Galarza |
| Fecha | 12 de noviembre de 2026 |

Documento de trabajo para lectura y aprobación

## Índice de secciones

1. Nombre del proyecto
2. Resumen ejecutivo
3. Problema y oportunidad de mejora
4. Caso demostrativo
5. Principios de diseño
6. Arquitectura propuesta
7. Fronteras y responsabilidades
8. Modelo canónico y particularidades de producto
9. Cores simulados y referencias de cotización
10. Precio y desglose del premio
11. Versionado y activación
12. Autenticación y autorización
13. Simultaneidad e idempotencia
14. Métricas, analítica y registro de incidentes
15. Validación de capacidad
16. Niveles acumulativos de entrega
17. Nivel 1 - Mínimo defendible
18. Nivel 2 - Base recomendada
19. Nivel 3 - Entrega sólida
20. Nivel 4 - Entrega extendida
21. Relación entre disponibilidad y nivel
22. Entregables por nivel
23. Exclusiones generales
24. Riesgos y mitigaciones
25. Hitos sugeridos
26. Regla de finalización de nivel
27. Demostración propuesta
28. Defensa resumida
29. Referencias técnicas y del dominio

### Cómo leer la numeración

- Versión 2.2 identifica la revisión de este documento.
- Las secciones 1 a 29 organizan el documento; no son pasos de implementación.
- Nivel 1 a Nivel 4 identifica alternativas acumulativas de alcance.
- Movilidad v1 y Movilidad v2 identifica versiones funcionales del producto demostrativo.
- PR, ESP, VER, CA, FIN y DEMO son identificadores estables y no una numeración continua.

## 1. Nombre del proyecto

Plataforma versionada de cotización de seguros con integración canónica y medición centralizada

## 2. Resumen ejecutivo

En la comercialización de seguros es frecuente que productos, canales y conexiones con diferentes cores evolucionen como aplicaciones separadas. Esto puede producir frontends y backends similares, integraciones repetidas, repositorios numerosos y métricas comerciales fragmentadas.

El proyecto propone validar una mejora arquitectónica acotada mediante un prototipo para la familia de producto Movilidad, con las modalidades bicicleta y bicicleta eléctrica. La plataforma utilizará un frontend compartido por cliente y operador, una API de experiencia, un backend modular, un contrato canónico de cotización y adaptadores para dos cores simulados con contratos y capacidades diferentes.

La solución permitirá mantener varias versiones de una experiencia, activar una versión previamente desplegada, volver a una versión anterior, normalizar propuestas y registrar cotizaciones, intentos de integración, ofertas y ventas simuladas en un punto central.

El desarrollo se realizará mediante Spec-Driven Development (SDD) asistido por un modelo de inteligencia artificial. Los documentos originales se conservarán como evidencia cruda y una etapa de refinamiento producirá contexto curado, especificaciones revisables, contratos y criterios de aceptación antes de comenzar la implementación.

El proyecto no busca construir una plataforma universal ni reemplazar sistemas existentes. Busca producir evidencia de que una arquitectura modular basada en contratos, versiones y adaptadores permite incorporar una variante sin duplicar una aplicación completa y manteniendo la trazabilidad técnica y comercial.

## 3. Problema y oportunidad de mejora

El escenario que motiva la propuesta presenta los siguientes problemas:

- Desarrollo separado de frontend, BFF y backend para productos o canales similares.
- Duplicación de recorridos para cliente final y operador.
- Integraciones específicas y repetidas con distintos cores.
- Falta de un mecanismo central para activar y revertir versiones.
- Dificultad para incorporar una modalidad sin crear otra aplicación.
- Cotizaciones y ventas distribuidas entre diferentes desarrollos.
- Falta de trazabilidad por producto, modalidad, versión, core, canal y actor.
- Diferencias operativas entre cores, como cotizaciones recuperables y cotizaciones volátiles.
- Diferentes niveles de detalle en el precio informado por cada core.

La hipótesis del proyecto es:

Un prototipo basado en un monolito modular, un contrato canónico acotado, definiciones versionadas y adaptadores de core permite incorporar variantes con menos duplicación y mayor trazabilidad que la creación de aplicaciones independientes.

## 4. Caso demostrativo

Se utilizará una familia sintética llamada Movilidad para evitar afirmar una clasificación regulatoria que no forma parte del alcance.

### Movilidad v1

- Cotización de bicicleta tradicional.
- Recorrido genérico.
- Campos y validaciones básicas.
- Disponible inicialmente para cliente final.

### Movilidad v2

- Cotización de bicicleta tradicional o bicicleta eléctrica.
- Para bicicleta eléctrica incorpora potencia, batería y valor específico.
- Puede mostrar capacidades adicionales al operador.
- Se activa para una audiencia determinada sin eliminar la versión anterior.

Las dos versiones estarán previamente instaladas. La activación y el rollback modificarán la asignación de versiones, no cargarán ni ejecutarán código arbitrario.

## 5. Principios de diseño

- PR-01: Una identidad interna por operación. La plataforma genera sus propios identificadores antes de invocar un core.
- PR-02: Canónico acotado. Se normaliza la operación de cotización y sus datos comunes, sin crear un objeto universal con todos los campos de todos los ramos.
- PR-03: Particularidades explícitas. Los datos propios de Movilidad se expresan mediante un esquema de riesgo versionado.
- PR-04: Adaptadores para diferencias externas. Los nombres, códigos, protocolos y capacidades particulares de cada core quedan dentro de su adaptador.
- PR-05: Configuración limitada. JSON Schema y configuraciones resuelven estructura, validaciones y mapeos simples. Las reglas complejas permanecen en código TypeScript probado.
- PR-06: Versiones inmutables. Una definición publicada no se modifica; un cambio produce una nueva versión.
- PR-07: Trazabilidad antes que volumen. Toda operación debe poder reconstruirse mediante identificadores y versiones persistidas.
- PR-08: Evidencia reproducible. Las afirmaciones de reutilización y rendimiento se validan mediante pruebas y condiciones documentadas.

### 5.1 Spec-Driven Development asistido por IA

SDD será una metodología del proyecto y no una funcionalidad entregada a los usuarios de la plataforma.

El flujo será:

```text
Documentos crudos sanitizados
        |
        v
Extracción asistida por IA
hechos, términos, reglas, conflictos y dudas
        |
        v
Contexto curado y aprobado por la persona responsable
        |
        v
Especificación de la funcionalidad
objetivo, actores, flujos, invariantes y aceptación
        |
        v
Plan técnico y contratos
OpenAPI, JSON Schema, decisiones y modelo de datos
        |
        v
Tareas y pruebas
        |
        v
Implementación y verificación de convergencia
```

El modelo de IA no será considerado una fuente de verdad. Sus resultados serán candidatos sujetos a revisión. Una salida no aprobada no podrá convertirse automáticamente en contrato, migración, prueba o código.

#### Capas documentales

```text
docs/
├── raw/                    documentos originales sanitizados e inmutables
├── sources/                inventario, origen, fecha y hash de cada fuente
├── domain/
│   ├── glossary.md         vocabulario de seguros y equivalencias
│   ├── facts.md            hechos aprobados y su procedencia
│   ├── rules.md            reglas de negocio confirmadas
│   ├── unknowns.md         dudas y decisiones pendientes
│   └── conflicts.md        contradicciones entre fuentes
├── adr/                    decisiones de arquitectura
└── traceability/           relación entre fuentes, requisitos y pruebas

specs/
└── <feature>/
    ├── spec.md             qué se construye y por qué
    ├── acceptance.md       escenarios y criterios verificables
    ├── plan.md             diseño técnico aprobado
    ├── tasks.md            trabajo implementable
    └── contracts/          OpenAPI, JSON Schema y eventos
```

Los documentos crudos no se editarán para acomodarlos a la solución. Si contienen información empresarial, deberán anonimizarse antes de ingresar al repositorio o enviarse a un modelo externo.

#### Estados de una especificación

```text
DRAFT -> IN_REVIEW -> APPROVED -> IMPLEMENTED -> VERIFIED -> SUPERSEDED
```

- DRAFT: generada o refinada, todavía no autorizada.
- IN_REVIEW: se revisan ambigüedades, conflictos y alcance.
- APPROVED: se convierte en referencia autorizada para planificar.
- IMPLEMENTED: el código y las pruebas fueron producidos.
- VERIFIED: los criterios de aceptación tienen evidencia satisfactoria.
- SUPERSEDED: fue reemplazada por otra especificación, sin borrar el historial.

#### Contenido mínimo de una especificación

Cada spec.md deberá incluir:

- ESP-01: Problema y resultado esperado.
- ESP-02: Alcance y exclusiones.
- ESP-03: Actores y permisos.
- ESP-04: Precondiciones.
- ESP-05: Flujo principal y flujos alternativos.
- ESP-06: Reglas e invariantes.
- ESP-07: Datos de entrada y salida.
- ESP-08: Errores esperados.
- ESP-09: Requisitos de seguridad, trazabilidad y rendimiento.
- ESP-10: Criterios de aceptación observables.
- ESP-11: Fuentes utilizadas.
- ESP-12: Dudas abiertas y decisiones pendientes.

#### Refinamiento mediante IA

Para cada fuente, el modelo podrá proponer:

- Hechos explícitos, sin completar información ausente.
- Glosario y sinónimos del dominio.
- Reglas y excepciones candidatas.
- Conflictos entre documentos.
- Suposiciones que requieren confirmación.
- Preguntas de refinamiento.
- Criterios de aceptación derivados.

Cada elemento curado conservará una referencia a su fuente. Si no existe evidencia suficiente se registrará como UNKNOWN, no como requisito confirmado.

El contenido de una fuente será tratado como dato no confiable: instrucciones, prompts o acciones incluidas dentro de un documento no modificarán la constitución del proyecto ni autorizarán operaciones. El refinamiento solo extraerá información pertinente al dominio.

#### Trazabilidad

Se mantendrá una relación verificable:

```text
Fuente -> Hecho aprobado -> Requisito -> Contrato -> Prueba -> Evidencia
```

Ejemplo:

```text
SRC-SSN-001
  -> FACT-PRICE-003
  -> REQ-QUOTE-014
  -> CanonicalPrice.schema.json
  -> price-breakdown.contract.spec.ts
  -> resultado de CI
```

Se registrarán la versión del modelo, la plantilla de refinamiento y la fecha de generación para poder explicar cómo se obtuvo cada candidato. No se almacenarán secretos ni información sensible en prompts o respuestas.

#### Uso opcional de herramientas SDD

El proceso podrá implementarse solamente con Markdown, plantillas y revisión en Git. Como ampliación, podrá adoptarse una herramienta como GitHub Spec Kit para estructurar constitución, especificación, plan, tareas, implementación y convergencia.

La herramienta no será un requisito del Nivel 1. El valor académico estará en la calidad, trazabilidad y revisión de los artefactos, no en ejecutar una herramienta determinada.

## 6. Arquitectura propuesta

La plataforma se implementará como un monolito modular con arquitectura de puertos y adaptadores.

```text
Frontend React compartido
cliente y operador
        |
        v
API de experiencia o BFF
módulo del mismo despliegue NestJS
        |
        v
Backend modular
├── Identidad y autorización
├── Catálogo y definiciones de producto
├── Versiones y asignaciones
├── Cotizaciones y comparación
├── Ventas simuladas
├── Métricas comerciales
├── Auditoría y trazabilidad
└── Registro de adaptadores
        |
        ├── Adaptador Core A ---> API Core A simulada
        └── Adaptador Core B ---> API Core B simulada
        |
        v
PostgreSQL
```

La API de experiencia no contendrá reglas centrales de negocio. Adaptará las respuestas al canal, agregará el contexto del actor y expondrá solamente las capacidades correspondientes al perfil.

Los cores simulados estarán en el mismo monorepo y entorno local, pero serán tratados como dependencias externas al módulo de cotización. Esto permite demostrar contratos HTTP diferentes, latencia, errores y volatilidad sin implementar microservicios productivos.

## 7. Fronteras y responsabilidades

| Elemento | Responsabilidad |
| --- | --- |
| Frontend | Presentación, navegación y componentes compartidos |
| API de experiencia | Adaptación por canal y perfil; composición de respuestas |
| Módulo de producto | Definición y reglas propias de Movilidad |
| Módulo de cotización | Caso de uso, identidad interna, selección y comparación |
| Modelo canónico | Operación común, riesgo tipado, ofertas y referencias |
| Adaptador de core | Traducción, protocolo, errores y capacidades del core |
| Core simulado | Contrato externo ficticio y comportamiento configurable |
| Métricas comerciales | Consultas de cotizaciones, ofertas y ventas persistidas |
| Observabilidad | Métricas técnicas, logs correlacionados y tiempos |

## 8. Modelo canónico y particularidades de producto

El contrato canónico tendrá un sobre común y un riesgo específico discriminado:

```ts
interface CanonicalQuotationRequest {
  quotationId: string;
  productCode: string;
  productVersion: string;
  channel: "customer" | "operator";
  actor: ActorReference;
  applicant: Applicant;
  risk: MobilityRisk;
  selectedCoverages: string[];
}

interface MobilityRisk {
  type: "mobility";
  modality: "bicycle" | "electric_bicycle";
  brand: string;
  model?: string;
  insuredValue: Money;
  batteryPower?: number;
}
```

JSON Schema describirá los campos, tipos, obligatoriedad y condiciones estructurales. Las reglas semánticas complejas se implementarán en el módulo de producto.

OpenAPI documentará las APIs HTTP. JSON Schema no reemplazará la implementación del adaptador ni será utilizado para ejecutar código recibido dinámicamente.

## 9. Cores simulados y referencias de cotización

### Core A

- Devuelve identificador externo de cotización.
- Permite recuperar la cotización.
- Informa desglose completo del premio.
- Utiliza códigos y nombres propios.

### Core B

- No devuelve identificador externo.
- La cotización es volátil.
- Informa solamente el premio total o un desglose parcial.
- Requiere conservar parámetros de recuperación y recotizar antes de una operación posterior.

La plataforma siempre generará:

```text
quotationId   identidad interna de la cotización
attemptId     identidad del intento contra un core
correlationId trazabilidad transversal
```

La referencia entregada por el adaptador podrá ser:

```ts
type CoreQuotationReference =
  | { kind: "external-id"; externalId: string }
  | { kind: "parameters"; parameters: object; fingerprint: string }
  | { kind: "not-recoverable" };
```

El fingerprint permitirá detectar solicitudes equivalentes, pero no reemplazará al identificador interno.

## 10. Precio y desglose del premio

La oferta canónica no se limitará a un precio único. Permitirá representar:

- Prima pura.
- Prima de tarifa, si el core la informa con ese significado.
- Gastos de producción.
- Gastos de explotación.
- Recargos.
- Impuestos y tasas.
- Sellados.
- Percepciones, cuando correspondan y sean informadas.
- Cargos financieros.
- Premio total.

Cada oferta indicará el nivel de detalle recibido:

```ts
type BreakdownStatus = "COMPLETE" | "PARTIAL" | "TOTAL_ONLY";
```

La plataforma no inventará componentes faltantes ni recalculará impuestos no informados. Los importes se almacenarán como valores decimales, no como números de punto flotante.

## 11. Versionado y activación

Se distinguirán:

- VER-01: Versión de definición de producto.
- VER-02: Versión del esquema de riesgo.
- VER-03: Versión de experiencia.
- VER-04: Versión del adaptador.
- VER-05: Versión de asignación o release.
- VER-06: Versión de la oferta cotizada.

Una cotización iniciada conservará las versiones con las que fue creada, aunque posteriormente se active otra versión.

```text
Movilidad v1 desplegada --+
                          +--> Version Resolver --> versión asignada
Movilidad v2 desplegada --+
```

El rollback cambiará la versión activa entre implementaciones previamente desplegadas. No implicará subir librerías desde el panel ni utilizar microfrontends remotos.

## 12. Autenticación y autorización

La identidad se incorporará gradualmente según el nivel alcanzado.

El diseño objetivo utilizará OpenID Connect mediante Keycloak y autorización en el backend. Los roles mínimos serán:

```text
customer        cotiza y consulta sus operaciones
operator        cotiza para un cliente y ve capacidades comerciales
product-manager activa, retira y revierte versiones
```

El frontend podrá ocultar acciones no disponibles, pero la autorización definitiva se realizará en la API.

No se implementarán contraseñas propias, registro público, recuperación de contraseña, autenticación corporativa ni segundo factor.

## 13. Simultaneidad e idempotencia

Se validarán dos dimensiones.

### 13.1 Simultaneidad técnica

- Solicitudes activas.
- Flujos de cotización por segundo.
- Tiempo de respuesta p50, p95 y p99.
- Errores y timeouts.
- Llamadas activas por core.
- Espera del pool de conexiones.
- CPU y memoria del entorno de prueba.

### 13.2 Simultaneidad de negocio

- Dos solicitudes con la misma clave de idempotencia generan una sola cotización lógica.
- Una misma oferta no puede registrar dos ventas.
- Solo puede existir una asignación activa para la misma combinación de producto, canal y audiencia cuando las reglas sean excluyentes.
- Una actualización concurrente de versión utiliza bloqueo optimista o produce un conflicto controlado.

Se utilizarán transacciones, restricciones únicas, claves de idempotencia y respuestas 409 Conflict cuando corresponda.

## 14. Métricas, analítica y registro de incidentes

### 14.1 Métricas técnicas

Se medirán como mínimo:

```text
http_requests_total
http_request_duration
quotation_started_total
quotation_completed_total
quotation_failed_total
core_request_duration
core_timeout_total
active_core_requests
```

### 14.2 Información comercial

Las preguntas sobre cuánto se cotizó, cuánto se vendió, quién intervino y qué versión convirtió mejor se responderán con datos persistidos:

```text
BusinessEvent
├── eventType
├── occurredAt
├── productCode
├── productVersion
├── coreCode
├── adapterVersion
├── channel
├── actorType
├── operatorId seudónimo
├── quotationId
├── offerId
├── saleId
├── amount
├── currency
└── result
```

Los identificadores de clientes u operadores no se utilizarán como etiquetas de métricas técnicas.

### 14.3 Incidentes

Cada llamada podrá reconstruirse mediante:

```text
correlationId -> quotationId -> attemptId -> core -> resultado o error
```

Los logs serán estructurados y no contendrán tokens, contraseñas ni datos personales completos.

Si se obtienen estadísticas empresariales reales, solo se utilizarán datos agregados y autorizados. En ausencia de autorización, todas las cargas, actores e incidentes serán sintéticos.

## 15. Validación de capacidad

No se afirmará que la solución soporta una cantidad productiva de usuarios. Se documentará el entorno y se medirá el comportamiento bajo una carga sintética conocida.

Los cores simulados permitirán configurar latencia y porcentaje de error:

```text
CORE_A_LATENCY_MS=200
CORE_A_ERROR_RATE=0
CORE_B_LATENCY_MS=350
CORE_B_ERROR_RATE=0
```

La carga se expresará principalmente como flujos de cotización por segundo. Los usuarios virtuales representarán recorridos concurrentes, no usuarios registrados o sesiones inactivas.

Los límites de rate limiting protegerán contra abuso, pero no serán utilizados como evidencia de capacidad.

## 16. Niveles acumulativos de entrega

Cada nivel es acumulativo. Un nivel superior solo comienza cuando el anterior está ejecutable, probado, documentado y preparado para demostración.

| Nivel | Nombre | Esfuerzo acumulado | Resultado |
| --- | --- | --- | --- |
| 1 | Mínimo defendible | 40 a 50 horas | Validación integral del concepto con identidad simulada |
| 2 | Base recomendada | 55 a 70 horas | Autenticación real, contratos y concurrencia controlada |
| 3 | Entrega sólida | 80 a 100 horas | Resiliencia, observabilidad y configuración versionada |
| 4 | Entrega extendida | 110 a 140 horas | Evaluación avanzada de capacidad y extensibilidad |

## 17. Nivel 1 - Mínimo defendible

### Objetivo

Demostrar la mejora de extremo a extremo con el menor número de componentes posible.

### Funcionalidad

- Un producto Movilidad.
- Modalidades bicicleta y bicicleta eléctrica.
- Un frontend compartido.
- Perfiles sintéticos de cliente y operador seleccionados en modo demostración.
- Dos versiones previamente instaladas.
- Activación y rollback simple.
- Dos contratos de core simulados.
- Core A con ID externo y desglose completo.
- Core B volátil y con premio total.
- Normalización y comparación de ofertas.
- Venta simulada idempotente.
- Consulta simple de cotizaciones y ventas.

### Seguridad

- Datos completamente sintéticos.
- Sin contraseñas ni autenticación real.
- El modo de identidad simulada estará identificado como una limitación del prototipo.
- Validación de entradas y secretos fuera del repositorio.

### Simultaneidad

- Idempotencia de venta.
- Restricción de una versión activa por alcance definido.
- Prueba automatizada de doble solicitud de venta.

### Métricas y trazabilidad

- Persistencia de producto, modalidad, versión, core, canal y actor.
- correlationId, quotationId y attemptId.
- Totales simples de cotizaciones y ventas.
- Registro de errores por intento de core.

### Rendimiento

- Smoke test automatizado.
- Línea base reproducible con al menos 5 flujos de cotización por segundo durante 3 minutos.
- Menos de 1 % de errores atribuibles a la plataforma.
- Resultado informado como evidencia local, no como capacidad productiva.

### SDD

- Constitución breve con principios de alcance, pruebas, privacidad y arquitectura.
- Directorios docs/raw, docs/domain y specs creados.
- Una especificación aprobada para el recorrido de cotización.
- Una especificación aprobada para venta idempotente.
- Documentos crudos sintéticos o sanitizados.
- Refinamiento asistido por IA ejecutado manualmente y revisado por Alejandro.
- Relación simple entre cada requisito crítico y su prueba.

### Criterios de aceptación del Nivel 1

- CA-N1-01: Cliente y operador completan el mismo recorrido desde el frontend compartido.
- CA-N1-02: Las dos modalidades no requieren duplicar la aplicación.
- CA-N1-03: Los dos cores se integran mediante el mismo puerto canónico.
- CA-N1-04: Las diferencias de ID y desglose quedan encapsuladas.
- CA-N1-05: Toda operación conserva sus versiones y referencias.
- CA-N1-06: La venta repetida no produce duplicados.
- CA-N1-07: El rollback selecciona una versión previamente instalada.
- CA-N1-08: La demostración se ejecuta con datos sintéticos siguiendo un manual.
- CA-N1-09: Ninguna funcionalidad crítica se implementa sin una especificación aprobada.
- CA-N1-10: Los requisitos críticos pueden relacionarse con una fuente y una prueba.

## 18. Nivel 2 - Base recomendada

### Objetivo

Agregar identidad real, límites explícitos y mejor evidencia de desacoplamiento.

### Incrementos funcionales

- Keycloak mediante OpenID Connect.
- Roles customer, operator y product-manager.
- Autorización de endpoints administrativos en el backend.
- Ciclo DRAFT -> APPROVED -> ACTIVE -> RETIRED.
- Versiones publicadas inmutables.
- JSON Schema versionado para los riesgos de Movilidad.
- Suite contractual reutilizable para todos los adaptadores.
- Manejo de timeout y resultado parcial cuando un core falla.
- Logs estructurados con correlación.

### Simultaneidad

- Clave de idempotencia para crear cotizaciones y ventas.
- Prueba de activación concurrente de versiones.
- Conflicto controlado mediante restricción única o bloqueo optimista.
- Límite configurable de llamadas activas por adaptador.

### Métricas

- Filtros comerciales por producto, versión, modalidad, core, canal y actor.
- Tiempo de respuesta por adaptador.
- Conteo de timeouts y errores por core.
- Registro de cambios administrativos.

### Rendimiento

- 10 flujos de cotización por segundo durante 10 minutos.
- Hasta 50 usuarios virtuales para sostener la tasa.
- Menos de 1 % de errores de plataforma.
- p95 inferior a 2 segundos con latencias simuladas documentadas.
- Cero iteraciones descartadas y cero operaciones duplicadas.

### SDD

- Plantillas estables para spec.md, acceptance.md, plan.md y tasks.md.
- Inventario de fuentes con identificador, fecha, origen y hash.
- Glosario, hechos, reglas, dudas y conflictos separados.
- Registro de versión del modelo y plantilla usada en cada refinamiento.
- Revisión obligatoria antes de pasar una especificación a APPROVED.
- OpenAPI y JSON Schema vinculados a la especificación que los origina.
- Análisis de consistencia antes de implementar.

### Criterios de aceptación del Nivel 2

- CA-N2-01: Los permisos se validan en el backend.
- CA-N2-02: Una versión aprobada no puede modificarse.
- CA-N2-03: Todos los adaptadores superan la misma suite contractual.
- CA-N2-04: Un core lento no impide registrar el resultado del otro.
- CA-N2-05: Las operaciones concurrentes mantienen las invariantes de negocio.
- CA-N2-06: La prueba de carga aprueba o falla mediante umbrales automáticos.
- CA-N2-07: Una afirmación sin fuente suficiente se registra como duda y no como requisito.
- CA-N2-08: Los cambios de alcance actualizan especificación, plan, tareas y pruebas.

## 19. Nivel 3 - Entrega sólida

### Objetivo

Demostrar una solución observable, resistente a fallos controlados y con asignación versionada.

### Incrementos funcionales

- Reglas de asignación por canal, rol y porcentaje determinístico.
- Vigencia y prioridad de asignaciones.
- Separación explícita de versiones de producto, esquema, experiencia, adaptador y release.
- Dashboard técnico y comercial básico.
- Docker Compose para frontend, backend, PostgreSQL, Keycloak y cores simulados.
- OpenTelemetry o instrumentación equivalente para métricas y trazas.

### Resiliencia

- Bulkhead o semáforo por adaptador.
- Timeouts configurables.
- Simulación de core lento y core inestable.
- Recotización de ofertas volátiles.
- Comparación de la oferta original y la recotizada.
- Estado explícito de desglose completo, parcial o solo total.

### Rendimiento

- Pruebas de carga normal, stress y pico.
- Identificación del primer punto donde se incumple el SLO.
- Registro de CPU, memoria, pool de conexiones y llamadas activas.
- Factor de seguridad documentado para el límite recomendado del prototipo.

### SDD

- Flujo asistido y repetible desde refinamiento hasta convergencia.
- Verificación automática de formatos y enlaces de trazabilidad.
- Control de deriva entre OpenAPI, JSON Schema, implementación y pruebas.
- Matriz de cobertura entre requisitos y criterios de aceptación.
- Evidencia de convergencia por funcionalidad.
- Adopción opcional de Spec Kit u otra herramienta equivalente.

### Criterios de aceptación del Nivel 3

- CA-N3-01: El mismo usuario sintético recibe una asignación estable.
- CA-N3-02: Una configuración inválida se rechaza antes de publicarse.
- CA-N3-03: Un fallo de un core queda aislado y trazado.
- CA-N3-04: Una oferta volátil se recotiza sin perder su historial.
- CA-N3-05: Métricas, logs y trazas permiten reconstruir un incidente sintético.
- CA-N3-06: El entorno completo se inicia mediante un procedimiento documentado.
- CA-N3-07: Los contratos publicados coinciden con las especificaciones aprobadas.
- CA-N3-08: Toda desviación conocida queda resuelta o registrada explícitamente.

## 20. Nivel 4 - Entrega extendida

### Objetivo

Ampliar la evidencia de extensibilidad y producir una evaluación técnica más profunda sin convertir el prototipo en una plataforma empresarial.

### Incrementos

- Una segunda familia mínima, por ejemplo Hogar, sin circuito completo de emisión.
- Reutilización de componentes visuales y del sobre canónico.
- Una extensión de producto cuya regla no pueda resolverse con JSON Schema.
- Tercer contrato simulado o variante significativa de integración.
- Prueba prolongada de estabilidad.
- Exportación de resultados de carga y negocio.
- Comparación formal del esfuerzo de incorporación.
- Informe de incidentes sintéticos y mitigaciones.

### SDD

- Corpus curado reutilizable para nuevas especificaciones.
- Conjunto de casos de evaluación para medir la calidad del refinamiento por IA.
- Comparación entre especificación generada, revisión humana y resultado final.
- Detección automatizada de fuentes nuevas o especificaciones posiblemente desactualizadas.

### Evidencia

- Tiempo de incorporación de la variante.
- Archivos y módulos modificados.
- Componentes reutilizados.
- Pruebas contractuales ejecutadas.
- Defectos encontrados.
- Capacidad observada y condiciones del entorno.

### Criterios de aceptación del Nivel 4

- CA-N4-01: La segunda familia reutiliza catálogo, versionado, trazabilidad y métricas.
- CA-N4-02: La extensión específica no contamina los adaptadores de otros productos.
- CA-N4-03: La comparación distingue resultados medidos de beneficios esperados.
- CA-N4-04: La prueba prolongada no presenta crecimiento no explicado de errores o recursos.
- CA-N4-05: La evaluación del modelo identifica omisiones, afirmaciones sin fuente y conflictos no detectados.
- CA-N4-06: El corpus curado puede producir una especificación nueva sin utilizar documentos confidenciales.

## 21. Relación entre disponibilidad y nivel

| Disponibilidad promedio | Horas en 8 semanas | Objetivo razonable |
| --- | --- | --- |
| 4 horas por semana | 32 | Nivel 1 reducido; un solo core y defensa conceptual del segundo |
| 6 horas por semana | 48 | Nivel 1 completo |
| 8 horas por semana | 64 | Nivel 2 con alcance controlado |
| 10 horas por semana | 80 | Nivel 3 inicial |
| 12 horas por semana | 96 | Nivel 3 completo |
| 14 a 18 horas por semana | 112 a 144 | Nivel 4 |

La recomendación es comprometer externamente el Nivel 1, trabajar internamente hacia el Nivel 2 y considerar los niveles superiores solo cuando el nivel anterior esté cerrado.

## 22. Entregables por nivel

| Entregable | N1 | N2 | N3 | N4 |
| --- | --- | --- | --- | --- |
| Frontend compartido | Sí | Sí | Sí | Sí |
| Backend NestJS modular | Sí | Sí | Sí | Sí |
| Dos cores simulados | Sí | Sí | Sí | Sí |
| Contrato canónico | Sí | Sí | Sí | Sí |
| Versiones y rollback | Sí | Sí | Sí | Sí |
| Venta idempotente | Sí | Sí | Sí | Sí |
| Identidad simulada | Sí | - | - | - |
| Keycloak y roles | - | Sí | Sí | Sí |
| JSON Schema versionado | Básico | Sí | Sí | Sí |
| Suite contractual | Básica | Sí | Sí | Sí |
| Prueba de carga | Línea base | Umbrales | Stress | Estabilidad |
| Métricas comerciales | Consulta | Filtros | Dashboard | Histórico y exportación |
| Observabilidad técnica | Logs básicos | Logs correlacionados | Métricas y trazas | Análisis avanzado |
| Documentos crudos y contexto curado | Sí | Sí | Sí | Sí |
| Especificaciones revisadas | Manuales | Plantillas | Flujo automatizado | Corpus reutilizable |
| Trazabilidad requisito a prueba | Básica | Completa | Validada en CI | Evaluada y comparada |
| Evaluación del refinamiento por IA | Revisión humana | Registro de generación | Controles automáticos | Casos de evaluación |
| Segunda familia | - | - | - | Sí |

## 23. Exclusiones generales

- Integraciones, contratos, credenciales o datos reales sin autorización.
- Emisión legal de pólizas.
- Pagos, cobranzas y facturación.
- Cálculo legal de impuestos por parte del orquestador.
- Validación real ante la Superintendencia de Seguros de la Nación.
- Certificación de cumplimiento normativo.
- Carga dinámica de librerías o ejecución de código arbitrario.
- Microfrontends remotos.
- Microservicios productivos, Kubernetes y mensajería distribuida.
- Alta disponibilidad y escalabilidad productiva.
- Migración de aplicaciones o repositorios existentes.
- Motor universal de reglas.
- Editor visual completo de productos.
- Pruebas A/B con significancia estadística.
- Uso de información confidencial de una empresa.
- Entrenamiento o ajuste fino de un modelo de IA.
- Aprobación automática de especificaciones generadas por IA.
- Envío de documentos empresariales no sanitizados a proveedores externos.
- Generación autónoma de código sin revisión de especificación, pruebas y cambios.

## 24. Riesgos y mitigaciones

| Riesgo | Impacto | Mitigación |
| --- | --- | --- |
| Modelo canónico demasiado amplio | Alto | Limitarlo a cotización de Movilidad y extensiones tipadas |
| Configuración convertida en lenguaje de programación | Alto | Usar JSON Schema para estructura y código para semántica |
| Sobreingeniería | Alto | Monolito modular y niveles acumulativos |
| Autenticación consume el tiempo del MVP | Medio | Identidad simulada en N1 y Keycloak desde N2 |
| Pruebas de carga no reproducibles | Alto | Documentar hardware, datos, latencias y umbrales |
| Confundir usuarios registrados con concurrencia | Medio | Medir flujos por segundo y solicitudes activas |
| Core lento bloquea la plataforma | Alto | Timeout y límites por adaptador desde N2/N3 |
| Datos sensibles en logs | Alto | Datos sintéticos, sanitización y revisión de logs |
| Rollback prometido sobre código no instalado | Alto | Limitarlo a versiones previamente desplegadas |
| Falta de tiempo | Alto | Congelar el nivel alcanzado y reservar tiempo para pruebas y defensa |
| Alucinaciones o inferencias no respaldadas del modelo | Alto | Separar hechos, supuestos y dudas; exigir fuente y aprobación humana |
| Filtración de información por prompts | Alto | Sanitizar documentos y prohibir secretos o datos personales |
| Especificaciones desactualizadas respecto del código | Alto | Trazabilidad, revisión de cambios y controles de deriva desde N2/N3 |
| Automatizar el proceso SDD antes de entenderlo | Medio | Comenzar con plantillas Markdown y automatizar solo después de validarlas |

## 25. Hitos sugeridos

| Semana | Resultado verificable |
| --- | --- |
| 1 | Alcance, constitución SDD, corpus inicial sanitizado, modelo canónico y arquitectura |
| 2 | Recorrido vertical con Core A |
| 3 | Core B, normalización e identidad interna |
| 4 | Modalidades y versiones sin duplicar frontend |
| 5 | Venta idempotente, métricas, trazabilidad SDD y cierre de Nivel 1 |
| 6 | Keycloak, ciclo de versiones y concurrencia de Nivel 2 |
| 7 | Pruebas, documentación y ampliaciones seguras |
| 8 | Correcciones, demostración y defensa |

Los hitos de semanas 6 y 7 podrán recortarse si el Nivel 1 necesita estabilización.

## 26. Regla de finalización de nivel

No se comenzará un nivel superior hasta responder afirmativamente:

- FIN-01: ¿La aplicación puede iniciarse desde cero siguiendo el manual?
- FIN-02: ¿El recorrido principal funciona con datos sintéticos?
- FIN-03: ¿Las pruebas críticas pasan?
- FIN-04: ¿La documentación coincide con el comportamiento actual?
- FIN-05: ¿La demostración puede ejecutarse sin modificar código?
- FIN-06: ¿Existe tiempo para implementar, probar y documentar el siguiente incremento?

Si la última respuesta es negativa, se congela el alcance y se prepara la defensa del nivel alcanzado.

## 27. Demostración propuesta

- DEMO-01: Iniciar sesión o seleccionar identidad, según el nivel.
- DEMO-02: Cotizar una bicicleta mediante Movilidad v1.
- DEMO-03: Comparar ofertas normalizadas de Core A y Core B.
- DEMO-04: Mostrar el ID externo del Core A y la referencia por parámetros del Core B.
- DEMO-05: Mostrar desglose completo y premio total sin desglose.
- DEMO-06: Registrar una venta simulada y repetir la solicitud para demostrar idempotencia.
- DEMO-07: Activar Movilidad v2 para operador o una audiencia.
- DEMO-08: Cotizar una bicicleta eléctrica.
- DEMO-09: Ejecutar rollback a v1.
- DEMO-10: Consultar cotizaciones, ventas y trazabilidad.
- DEMO-11: Ejecutar o presentar el resultado de la prueba de carga del nivel alcanzado.
- DEMO-12: Mostrar una fuente cruda sanitizada, el hecho curado y la especificación aprobada.
- DEMO-13: Recorrer un requisito desde su fuente hasta la prueba que lo verifica.

## 28. Defensa resumida

El proyecto no propone construir una plataforma universal ni reemplazar los sistemas existentes. Valida sobre un producto y dos cores simulados si un frontend compartido, un contrato canónico acotado, definiciones versionadas y adaptadores permiten incorporar variantes sin duplicar aplicaciones. El desarrollo utiliza SDD asistido por IA: los documentos crudos se conservan como evidencia, el modelo propone hechos y requisitos, y una revisión humana aprueba la especificación antes de generar contratos, pruebas y código. Cada operación conserva su identidad, versiones, core, canal y actor. La capacidad se informa únicamente bajo una carga sintética y un entorno documentado. El alcance está organizado en niveles para garantizar una entrega funcional aun cuando la disponibilidad sea limitada.

## 29. Referencias técnicas y del dominio

- NestJS Modules: https://docs.nestjs.com/modules
- Arquitectura Ports and Adapters: https://alistair.cockburn.us/hexagonal-architecture
- JSON Schema: https://json-schema.org/docs
- OpenAPI Specification: https://spec.openapis.org/oas/
- Keycloak OpenID Connect: https://www.keycloak.org/securing-apps/oidc-layers
- Grafana k6 Thresholds: https://grafana.com/docs/k6/latest/using-k6/thresholds/
- OpenTelemetry Signals: https://opentelemetry.io/docs/concepts/signals/
- ACORD Property and Casualty Standards: https://www.acord.org/standards-architecture/acord-data-standards/Property_Casualty_Data_Standards
- SSN Contrato de Seguro: https://www.argentina.gob.ar/superintendencia-de-seguros/portal-ciudadano-0/contrato-de-seguro
- Reglamento General de la Actividad Aseguradora: https://www.argentina.gob.ar/normativa/nacional/norma-238219/actualizacion
- GitHub Spec Kit: https://github.com/github/spec-kit
- Spec Kit para proyectos existentes: https://github.com/github/spec-kit/blob/main/docs/guides/existing-projects.md
- Contract-Driven Development con Spec Kit: https://github.com/github/spec-kit/blob/main/docs/guides/contract-driven-development.md
