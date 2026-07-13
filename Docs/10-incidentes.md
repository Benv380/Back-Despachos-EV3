# 10. Incidentes diagnosticados durante el desarrollo

Durante el desarrollo del proyecto se presentaron cinco incidentes reales, todos diagnosticados mediante el análisis de logs en CloudWatch y de las métricas y eventos de ECS / Application Load Balancer. Se documenta cada uno con su síntoma, causa raíz, acción correctiva y aprendizaje.

# Incidente 1 — Variables de entorno sin resolver en la conexión a base de datos

- **Síntoma:** el servicio `back-ventas-svc` no alcanzaba estado estable; el conteo de tareas se mantenía en 0 mientras ECS reemplazaba continuamente las tareas fallidas (*crash loop*).
- **Causa raíz:** en los logs de CloudWatch se observó la excepción `Failed to parse the host:port pair '${DB_ENDPOINT}:${DB_PORT}'`: las variables de entorno requeridas por Spring Boot no habían sido inyectadas en la Task Definition, por lo que el framework interpretó el texto literal de la variable como si fuera el host.
- **Acción correctiva:** se agregaron `DB_ENDPOINT`, `DB_PORT`, `DB_NAME` y `DB_USERNAME` al bloque `environment` de la Task Definition, registrando una nueva revisión del servicio.
- **Aprendizaje:** toda dependencia externa de un contenedor debe inyectarse explícitamente vía variables de entorno en el entorno de orquestación; no basta con que el código la referencie.

# Incidente 2 — Desalineación entre el puerto de la aplicación y la infraestructura

- **Síntoma:** el servicio `back-despachos-svc` mostraba comportamiento inestable; las tareas se iniciaban pero eran reemplazadas constantemente por el health check del Target Group.
- **Causa raíz:** el código (`application.properties`) define `server.port=8081`, mientras que Security Group, Target Group y Task Definition habían sido configurados asumiendo el puerto 8080 por defecto.
- **Acción correctiva:** se creó un nuevo Target Group en el puerto correcto (8081), se actualizó la regla del listener del ALB, se amplió el Security Group de servicios para el puerto 8081, y se corrigió el `containerPort` en la Task Definition.
- **Aprendizaje:** es indispensable verificar el puerto real en que escucha cada aplicación (revisando su configuración, no asumiéndolo) antes de definir Security Groups, Target Groups y Task Definitions.

# Incidente 3 — Error de autenticación MySQL 8 ("Public Key Retrieval is not allowed")

- **Síntoma:** tras corregir los incidentes 1 y 2, el backend de despachos seguía sin conectar a la base de datos, con un error distinto en los logs de Hibernate/HikariCP.
- **Causa raíz:** MySQL 8 usa por defecto el plugin `caching_sha2_password`; al establecer la conexión sin SSL (`useSSL=false`), el driver JDBC requiere autorización explícita para solicitar la llave pública de autenticación, autorización que no estaba presente en la cadena de conexión.
- **Acción correctiva:** se inyectó la variable `SPRING_DATASOURCE_URL` en la Task Definition, sobrescribiendo la URL de conexión con una que incluye `allowPublicKeyRetrieval=true`, sin necesidad de modificar ni recompilar el código.
- **Aprendizaje:** Spring Boot permite sobrescribir cualquier propiedad de configuración mediante variables de entorno con prioridad superior, lo cual es útil para corregir parámetros de infraestructura sin tocar el código de la aplicación.

# Incidente 4 — Regla de ruteo del ALB desincronizada con la ruta real del backend de despachos

- **Síntoma:** las peticiones a `/api/v1/despachos` a través del ALB devolvían error 404, aun cuando el servicio y la base de datos ya funcionaban correctamente.
- **Causa raíz:** la regla del listener estaba definida con el patrón `/api/despachos/*`, mientras la ruta real expuesta por el controlador Spring (`@RequestMapping`) era `/api/v1/despachos` — un desajuste de un segmento de la URL.
- **Acción correctiva:** se actualizó la condición de la regla del listener a `/api/v1/despachos*`, alineándola con la ruta real del controlador.
- **Aprendizaje:** las reglas de ruteo de un Load Balancer deben validarse contra la ruta real expuesta por el código, no contra una convención asumida; un pequeño desajuste en el path es indistinguible de un servicio caído desde el punto de vista del cliente.

# Incidente 5 — Mismo problema de ruteo en ventas, combinado con URLs hardcodeadas en el frontend

- **Síntoma:** al insertar y consultar datos desde el frontend, la interfaz quedaba en blanco con el error de consola `TypeError: e.filter is not a function`. Una inspección con `curl` mostró que `/api/v1/ventas` devolvía el HTML del frontend en lugar de un arreglo JSON.
- **Causa raíz:** dos causas combinadas — (1) la regla del listener para ventas mantenía el patrón antiguo `/api/ventas/*` (sin el prefijo `v1`), por lo que el ALB derivaba la solicitud a la regla por defecto (el frontend); y (2) el frontend tenía URLs de backend escritas literalmente apuntando a una IP privada de red local (`192.168.x.x`), inalcanzable desde Internet.
- **Acción correctiva:** se corrigió el patrón de la regla del listener a `/api/v1/ventas*`, y se refactorizaron los componentes del frontend para usar rutas relativas (por ejemplo, `/api/v1/ventas`) en lugar de URLs absolutas hardcodeadas, aprovechando que el frontend y los backends comparten el mismo Load Balancer.
- **Aprendizaje:** el uso de rutas relativas en el frontend, en vez de URLs absolutas con direcciones fijas, hace la aplicación independiente de dónde se despliegue el backend, y evita este tipo de error al promover el código entre entornos.
