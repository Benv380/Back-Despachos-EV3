# 5. Infraestructura en la nube (AWS)

# 5.1 Red (VPC y subredes)

Se reutilizó la **VPC por defecto** de la cuenta de AWS, con subredes públicas en **dos zonas de disponibilidad** distintas dentro de `us-east-1`, lo que permite distribuir las tareas de ECS y el Application Load Balancer entre ambas zonas para tolerancia a fallos.

# 5.2 Grupos de seguridad (Security Groups)

Se definieron tres grupos de seguridad siguiendo un modelo de **mínimo privilegio**, donde cada capa solo acepta tráfico desde la capa inmediatamente anterior:

| Security Group | Tráfico entrante permitido |
|---|---|
| `innovatech-alb-sg` | Puerto 80, desde Internet (`0.0.0.0/0`) — único punto de entrada público |
| `innovatech-svc-sg` | Puertos 80, 8080 y 8081, únicamente desde `innovatech-alb-sg` |
| `innovatech-rds-sg` | Puerto 3306 (MySQL), únicamente desde `innovatech-svc-sg` |

Con esta configuración, ni los contenedores de aplicación ni la base de datos quedan expuestos directamente a Internet: todo el tráfico externo pasa obligatoriamente por el Load Balancer.

# 5.3 Rol de ejecución (IAM)

Para los permisos de ejecución de las tareas (descarga de imágenes desde ECR, escritura de logs en CloudWatch, lectura de parámetros en SSM) se utiliza el rol predefinido **`LabRole`** del entorno AWS Academy Learner Lab, tanto como *execution role* como *task role*, dado que el entorno no permite la creación de roles IAM personalizados. En un entorno productivo real se reemplazaría por roles específicos con permisos acotados exclusivamente a las acciones necesarias de cada servicio 

# 5.4 Application Load Balancer y Target Groups

Se creó un Application Load Balancer (`innovatech-alb`) sobre las dos subredes públicas, asociado al Security Group del ALB. Se configuraron **tres Target Groups de tipo IP** (requerido por el modo de red `awsvpc` de Fargate):

| Target Group | Puerto | Servicio |
|---|---|---|
| `tg-front` | 80 | Frontend |
| `tg-despachos` | 8081 | Backend Despachos |
| `tg-ventas` | 8080 | Backend Ventas |

El listener del puerto 80 deriva por defecto al Frontend, y posee dos reglas de prioridad adicionales que enrutan según el path de la solicitud (`/api/v1/despachos*` y `/api/v1/ventas*`) al backend correspondiente.

# 5.5 Clúster ECS

El clúster se creó bajo el nombre `innovatech-cluster`, junto con los grupos de logs en CloudWatch asociados a cada servicio (`/ecs/front-despacho`, `/ecs/back-despachos`, `/ecs/back-ventas`). Cada servicio se configuró con `desiredCount=2` (dos tareas por servicio, distribuidas en las dos zonas de disponibilidad), lanzamiento Fargate, y registro automático en su Target Group correspondiente.

# 5.6 Base de datos

La persistencia se resuelve mediante **Amazon RDS para MySQL 8.0** (`db.t3.micro`), ubicada dentro de la misma VPC y accesible únicamente desde los servicios ECS a través de `innovatech-rds-sg`.
