# Documentación Técnica — Proyecto Innovatech Chile

**Asignatura:** ISY1101 — Introducción a Herramientas DevOps
**Evaluación:** Evaluación Final Transversal (EFT) — Encargo con Presentación
**Proyecto:** Automatización del ciclo CI/CD y orquestación en producción de la plataforma de gestión de despachos y ventas de Innovatech Chile

Esta carpeta documenta las decisiones técnicas tomadas durante el desarrollo del proyecto, que consistió en llevar una plataforma compuesta por un frontend (React) y dos microservicios backend (Spring Boot), con persistencia en una base de datos relacional (MySQL), desde su contenedorización hasta un entorno de producción orquestado, autoescalable y con despliegue continuo automatizado sobre AWS ECS Fargate.

# Índice de contenidos

| Documento | Contenido |
|---|---|
| [01-arquitectura.md](01-arquitectura.md) | Arquitectura general, decisión ECS vs. EKS, diagrama de la solución |
| [02-contenedores.md](02-contenedores.md) | Dockerfiles, imágenes, `.dockerignore`, orquestación local con Docker Compose |
| [03-registro-imagenes.md](03-registro-imagenes.md) | Publicación de imágenes en Amazon ECR y estrategia de etiquetado (tags) |
| [04-cicd.md](04-cicd.md) | Pipeline de GitHub Actions: build, test, push, deploy |
| [05-infraestructura-nube.md](05-infraestructura-nube.md) | VPC, subredes, grupos de seguridad, ALB, clúster ECS |
| [06-configuracion-secretos.md](06-configuracion-secretos.md) | Variables de entorno, gestión de secretos, principio de mínimo privilegio |
| [07-observabilidad.md](07-observabilidad.md) | Logs de GitHub Actions y métricas de CloudWatch |
| [08-seguridad.md](08-seguridad.md) | Endurecimiento de imágenes, puertos mínimos, reglas restrictivas |
| [09-orquestacion-escalabilidad.md](09-orquestacion-escalabilidad.md) | Justificación de ECS/Fargate, autoscaling, balanceo, recuperación ante fallos |
| [10-incidentes.md](10-incidentes.md) | Incidentes reales diagnosticados durante el desarrollo, causa raíz y solución |
| [11-conclusiones.md](11-conclusiones.md) | Conclusiones y mejoras propuestas para un escenario productivo real |

# Recursos del proyecto

| Recurso | Referencia |
|---|---|
| Repositorio Frontend | `front-despacho` |
| Repositorio Backend Despachos | `back-despachos` |
| Repositorio Backend Ventas | `back-ventas` |
| Clúster ECS | `innovatech-cluster` (us-east-1) |
| URL pública de la aplicación | URL del Application Load Balancer (ver `05-infraestructura-nube.md`) |
