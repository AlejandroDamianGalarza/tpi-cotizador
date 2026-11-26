# Inicialización local de la base para la demo

**Estado: criterio de arranque local aprobado.** El entorno de demostración del TPI prepara la base del intermediario antes de iniciar la API. Este diseño se incorpora al manual de ejecución y al listado de módulos de la segunda entrega como tarea de inicialización. Se relaciona con el [alcance por niveles](alcance-por-niveles-y-bases-de-extension.md) y las [decisiones de perfiles y vigencia](decisiones-der-02-perfil-y-vigencia.md).

## Secuencia de arranque

Una vez implementados los servicios, el manual prevé `docker compose up --build` desde la raíz del monorepo.

```text
PostgreSQL listo
  → db-setup aplica las migraciones
  → db-setup carga los datos sintéticos de demo
  → db-setup termina correctamente
  → arranca la API
```

`db-setup` es un servicio de ejecución puntual, separado de la API. Puede usar la misma imagen con otro comando. Primero aplica las migraciones. Después valida y carga las identidades sintéticas, el catálogo y las revisiones de perfiles y mapeos requeridas por la demo. Los nombres propuestos para ambos comandos son `db:migrate` y `db:seed:demo`.

En Compose, `db-setup` espera que PostgreSQL esté sano (`service_healthy`). Durante el arranque, la API espera que `db-setup` termine correctamente (`service_completed_successfully`). El proceso de API arranca con la base ya preparada. [Orden de arranque en Docker Compose](https://docs.docker.com/compose/how-tos/startup-order/)

## Resultado esperado al repetir la carga

- Si la identidad de una revisión y su contenido coinciden, se reutiliza el registro existente. Si se intenta cargar la misma revisión con otro contenido, la preparación informa un conflicto. El cambio debe publicarse como otra revisión.
- Las asignaciones iniciales se preparan donde hagan falta al iniciar una base vacía. Repetir la carga conserva las activaciones y el historial de la demo ya ejecutada.
- Una falla en migración o carga se informa desde `db-setup`. La API queda a la espera de una preparación exitosa en ese arranque.
- Las claves de revisión que respaldan esta regla se reflejan en el DER y en el diccionario de datos. La prueba de idempotencia de venta se verifica por separado.

## Verificación prevista

1. Arrancar desde una base vacía y completar el recorrido de demo con los datos cargados.
2. Volver a ejecutar la preparación y comprobar que se conservan las mismas revisiones y asignaciones vigentes.
3. Intentar cargar una revisión existente con contenido distinto y comprobar que se informa el conflicto antes del arranque de la API.

CI puede probar los mismos comandos con una base descartable. Si después se automatiza un despliegue, un paso por entorno aplica las migraciones y las configuraciones autorizadas. El comando `db:seed:demo` queda reservado para los entornos de prueba y demostración.
