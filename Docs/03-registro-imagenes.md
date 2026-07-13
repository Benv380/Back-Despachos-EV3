# 3. Registro de imágenes (Amazon ECR)

# 3.1 Repositorios

Se crearon tres repositorios independientes en Amazon ECR, uno por componente:

| Repositorio | Componente |
|---|---|
| `front-despacho` | Frontend (React + Nginx) |
| `back-despachos` | Backend de gestión de despachos (Spring Boot) |
| `back-ventas` | Backend de gestión de ventas (Spring Boot) |

Mantener repositorios separados por componente permite versionar, escalar y desplegar cada servicio de forma independiente, sin acoplar sus ciclos de release.

# 3.2 Flujo de publicación

1. El pipeline de GitHub Actions autentica el CLI de Docker contra Amazon ECR usando las credenciales temporales de AWS almacenadas en GitHub Secrets.
2. Se construye la imagen a partir del Dockerfile del componente correspondiente.
3. La imagen se publica (`docker push`) con **dos etiquetas (tags)**:
   - El **SHA del commit** que originó el build, para trazabilidad exacta entre código fuente e imagen desplegada.
   - `latest`, como referencia al build más reciente.
4. La Task Definition de ECS referencia la imagen a desplegar; en el flujo actual se actualiza con cada nueva revisión generada por el pipeline.

# 3.3 Trazabilidad

El uso del SHA del commit como tag permite responder, ante cualquier incidente, la pregunta "¿qué versión exacta del código está corriendo en producción en este momento", sin depender únicamente de la etiqueta mutable `latest`. Esto también habilita rollback a una imagen específica simplemente referenciando un SHA anterior en una nueva revisión de la Task Definition.
