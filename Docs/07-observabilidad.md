# 7. Observabilidad

D 7.1 Logs del pipeline (GitHub Actions)

Cada ejecución del pipeline registra el detalle de todas sus etapas (build, test, push, render de Task Definition, deploy) en la pestaña *Actions* de cada repositorio. Estos logs permiten:

- Confirmar que una publicación de imagen y despliegue se completó exitosamente.
- Medir el tiempo que toma cada etapa individual del flujo.
- Diagnosticar fallos de compilación, de publicación en ECR, o de estabilización del servicio en ECS.

D 7.2 Logs de aplicación (Amazon CloudWatch)

Cada servicio ECS envía su salida estándar a un grupo de logs dedicado en CloudWatch, mediante el driver `awslogs`:

| Grupo de logs | Servicio |
|---|---|
| `/ecs/front-despacho` | Frontend |
| `/ecs/back-despachos` | Backend Despachos |
| `/ecs/back-ventas` | Backend Ventas |

Estos logs fueron la fuente principal de diagnóstico para los cinco incidentes documentados en [10-incidentes.md](10-incidentes.md): permitieron identificar excepciones de conexión a base de datos, fallos de health check, y errores de autenticación de MySQL, entre otros.

D 7.3 Métricas (Amazon CloudWatch)

Se monitorea la métrica `CPUUtilization` de cada servicio ECS, que es además la métrica utilizada por las políticas de autoscaling (ver [09-orquestacion-escalabilidad.md](09-orquestacion-escalabilidad.md)). Durante una prueba de carga concurrente sobre el backend de despachos, se registraron picos de utilización de CPU de hasta 35,7%, confirmando que el mecanismo de medición responde correctamente a la carga real del servicio.

Adicionalmente, CloudWatch genera automáticamente **alarmas** (`AlarmHigh` / `AlarmLow`) asociadas a cada política de Target Tracking configurada, una por servicio.

D 7.4 Verificación funcional

Más allá de logs y métricas de infraestructura, se validó el funcionamiento end-to-end de la aplicación mediante:

- Peticiones `curl` directas a los endpoints de cada backend a través del ALB.
- Carga de registros de prueba (`POST`) en ambos backends, confirmando persistencia y recuperación correcta desde MySQL.
- Verificación visual del resultado en el dashboard del frontend.
