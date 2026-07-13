# 9. Orquestación y escalabilidad





# 9.1 Por qué un servicio de orquestación y no un despliegue manual

Un despliegue manual de contenedores (por ejemplo, `docker run` directo sobre una o más instancias EC2) obligaría a resolver manualmente, y de forma frágil, problemas que un orquestador resuelve de forma nativa:

- **Recuperación ante fallos:** si un contenedor muere, nadie lo vuelve a levantar automáticamente.
- **Escalado:** agregar o quitar capacidad ante cambios de demanda requeriría intervención manual constante.
- **Balanceo de carga:** distribuir tráfico entre múltiples instancias de un mismo servicio requeriría configuración externa adicional.
- **Despliegues sin downtime:** actualizar la versión en ejecución sin cortar el servicio es complejo de lograr manualmente.

Amazon ECS (en modo Fargate) resuelve estos cuatro puntos de forma integrada, con configuración declarativa (Task Definitions y Services), sin necesidad de administrar servidores.





# 9.2 Autoscaling

Se configuró autoscaling de tipo **Target Tracking por utilización de CPU** para los tres servicios, mediante AWS Application Auto Scaling, con una capacidad mínima de 2 tareas y máxima de 6 tareas por servicio.





# Justificación del umbral

Se definió un umbral de **50% de utilización de CPU** como valor objetivo (Target Value) para la configuración de producción documentada. Este valor entrega margen para absorber picos de tráfico mientras Fargate aprovisiona tareas adicionales (proceso que toma aproximadamente un minuto), evitando tanto el sobre-aprovisionamiento como la saturación del servicio ante demanda sostenida. Los parámetros de cooldown se configuraron en 60 segundos tanto para scale-out como para scale-in, evitando oscilaciones ("flapping") en el número de tareas.

> Nota: durante la demostración en vivo en laboratorio, dado el bajo consumo de CPU que genera la carga de prueba disponible, el umbral se ajustó temporalmente a un valor menor para poder evidenciar el comportamiento de autoscaling dentro del tiempo de la sesión práctica. Esta calibración es habitual en pruebas de carga con recursos acotados y no representa la configuración recomendada para producción.





# Evidencia de simulación de carga

Se generó una prueba de carga concurrente sobre el backend de despachos mediante múltiples procesos en paralelo realizando solicitudes HTTP continuas a través del ALB. Se observó un incremento sostenido de la métrica `CPUUtilization` en CloudWatch, alcanzando picos de hasta 35,7%, confirmando que el mecanismo de medición y la política de autoscaling responden correctamente a la carga real del servicio.





# 9.3 Balanceo de carga

El Application Load Balancer distribuye el tráfico entre las tareas activas de cada servicio, registradas en su respectivo Target Group, verificando su salud mediante health checks HTTP antes de enviarles tráfico.





# 9.4 Recuperación ante fallos y despliegues sin downtime

Se validó la capacidad de autorecuperación del sistema forzando nuevos despliegues (`force-new-deployment`) sobre los servicios backend tras la corrección de cada incidente (ver [10-incidentes.md](10-incidentes.md)). En todos los casos, ECS ejecutó una estrategia de **rolling update**: las tareas nuevas se registraron en el Target Group y comenzaron a recibir tráfico antes de que las tareas antiguas fueran retiradas, sin interrupción del servicio observable desde el cliente.

Se validó además el funcionamiento integral de la solución desplegada: acceso público al frontend a través de la URL del Load Balancer, respuesta correcta de ambos backends, comunicación Frontend → Backend operativa a través del mismo ALB, persistencia de datos en RDS, y estabilidad de los tres servicios en su conteo deseado de tareas (2/2 en los tres casos).
