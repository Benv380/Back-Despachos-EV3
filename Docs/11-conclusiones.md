## 11. Conclusiones y proyección hacia un escenario productivo
# 11.1 Resumen (Bastian Ahumada)

El desarrollo de este proyecto permitió implementar, de manera práctica, un entorno completo de orquestación de contenedores en la nube, integrando la contenerización de las aplicaciones, su despliegue sobre Amazon ECS Fargate y la automatización del proceso de entrega mediante un pipeline CI/CD.

Desde mi participación, el principal aprendizaje estuvo relacionado con la preparación de las aplicaciones para ejecutarse de forma consistente mediante contenedores, la configuración de los servicios Spring Boot y su integración con la base de datos MySQL en Amazon RDS. Además, participé en el diagnóstico y resolución de problemas asociados a la autenticación de MySQL y a la correcta configuración de las variables de entorno, comprendiendo la importancia de una adecuada parametrización para garantizar la estabilidad de los servicios en producción.

La experiencia permitió fortalecer conocimientos sobre Docker, configuración de aplicaciones Java, integración con bases de datos y resolución de incidentes durante el proceso de despliegue.

# 11.1 Resumen (Benjamin Marfull)

El desarrollo de este proyecto permitió implementar, de manera práctica, un entorno completo de orquestación de contenedores en la nube, desde la contenerización de los componentes hasta su despliegue sobre un clúster Amazon ECS Fargate con balanceo de carga, alta disponibilidad y automatización del ciclo de entrega.

Desde mi participación, el principal aprendizaje estuvo en el diseño e implementación de la infraestructura sobre AWS, configurando la arquitectura de red mediante VPC, subredes, grupos de seguridad, Application Load Balancer y servicios ECS. Asimismo, participé en la resolución de incidentes relacionados con el enrutamiento del balanceador de carga y la comunicación entre los distintos componentes, comprendiendo la importancia de una arquitectura correctamente segmentada para garantizar la disponibilidad y seguridad de la plataforma.

Esta experiencia permitió consolidar conocimientos sobre infraestructura cloud, redes, balanceo de carga, orquestación de contenedores y arquitectura de aplicaciones desplegadas en producción.

# 11.1 Resumen (Diego Azcarategui)

El desarrollo de este proyecto permitió implementar, de manera práctica, un entorno completo de integración y despliegue continuo para una aplicación basada en contenedores, automatizando todo el proceso desde la construcción de imágenes hasta su despliegue en producción sobre Amazon ECS.

Desde mi participación, el principal aprendizaje estuvo en la implementación del pipeline CI/CD mediante GitHub Actions, la publicación de imágenes en Amazon ECR, la gestión segura de credenciales utilizando GitHub Secrets y AWS Systems Manager Parameter Store, además de la configuración de CloudWatch y las políticas de Auto Scaling para mejorar la operación de la plataforma.

La experiencia permitió comprender la importancia de la automatización de despliegues, la observabilidad y la escalabilidad como elementos fundamentales para mantener aplicaciones modernas ejecutándose de forma segura y confiable en entornos cloud.

# 11.2 Mejoras identificadas para un escenario productivo real

Las siguientes mejoras están condicionadas por las restricciones del entorno académico (AWS Academy Learner Lab) y quedan documentadas como el camino natural hacia un entorno productivo real:

Autenticación federada (OIDC): reemplazar las credenciales estáticas/temporales de GitHub Actions por un rol IAM federado mediante OpenID Connect, eliminando la necesidad de copiar credenciales manualmente en cada sesión de laboratorio.
Alta disponibilidad de base de datos: desplegar RDS en modo Multi-AZ y con respaldos automáticos habilitados (actualmente se usa una instancia de única zona sin retención de backups, por restricción del laboratorio).
Calibración definitiva de autoscaling: definir el umbral de producción (50% de CPU) de forma permanente, una vez validado su comportamiento con tráfico real y patrones de carga representativos.
HTTPS en el Load Balancer: incorporar un certificado de AWS Certificate Manager, en lugar de HTTP plano.
Centralización completa de secretos: migrar la configuración de conexión a base de datos completamente a AWS Secrets Manager o SSM Parameter Store (incluyendo el endpoint), reduciendo aún más la información sensible o de despliegue presente en las Task Definitions versionadas.
Roles IAM específicos: reemplazar el rol LabRole compartido por roles de ejecución y de tarea acotados por servicio, siguiendo estrictamente el principio de mínimo privilegio.