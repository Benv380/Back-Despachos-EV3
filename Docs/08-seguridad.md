# 8. Seguridad básica

# 8.1 Endurecimiento de imágenes

- Uso de imágenes base minimalistas: `eclipse-temurin:*-jre-alpine` para los backends y `nginx:alpine` para el frontend, reduciendo la superficie de ataque frente a imágenes base completas (menos paquetes, menos vulnerabilidades potenciales).
- Construcción multietapa: las herramientas de compilación (Maven, Node, JDK completo) y el código fuente no forman parte de la imagen final, solo el artefacto ya compilado.
- Ningún secreto ni credencial se incluye en la imagen ni en el Dockerfile; todo se inyecta en tiempo de ejecución (ver [06-configuracion-secretos.md](06-configuracion-secretos.md)).

# 8.2 Exposición mínima de puertos

Cada contenedor expone únicamente el puerto que su aplicación efectivamente utiliza:

- Frontend: puerto 80 (Nginx).
- Backend Despachos: puerto 8081.
- Backend Ventas: puerto 8080.

Ningún contenedor expone puertos adicionales de administración, debug o similares.

# 8.3 Reglas restrictivas en Security Groups

Como se detalla en [05-infraestructura-nube.md](05-infraestructura-nube.md), la red se segmenta en tres capas con acceso estrictamente descendente:

```
Internet → (puerto 80) → ALB → (80/8080/8081) → Servicios ECS → (3306) → RDS
```

Ni los servicios de aplicación ni la base de datos son accesibles directamente desde Internet: el único punto de entrada público es el Application Load Balancer.

# 8.4 Mínimo privilegio en credenciales de base de datos

La contraseña de la base de datos se almacena cifrada (`SecureString`) en SSM Parameter Store y se inyecta únicamamente a los backends que efectivamente la necesitan, nunca en texto plano en código, imágenes o repositorios.

# 8.5 Mejoras identificadas para producción

- Incorporar HTTPS en el Application Load Balancer mediante un certificado de AWS Certificate Manager (actualmente el tráfico externo es HTTP plano, aceptable solo en el entorno académico de laboratorio).
- Reemplazar credenciales estáticas/temporales de GitHub Actions por un rol IAM federado vía OIDC.
- Migrar completamente la configuración de conexión a base de datos (incluyendo el endpoint) a Secrets Manager o SSM, reduciendo aún más la información presente en las Task Definitions versionadas.
