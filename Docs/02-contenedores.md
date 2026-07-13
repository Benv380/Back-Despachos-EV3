# 2. Contenedores

# 2.1 Estrategia de contenedorización

Cada uno de los tres componentes se contenerizó de forma independiente mediante un **Dockerfile multietapa** propio:

- **Backends (Spring Boot):** primera etapa compila el proyecto con Maven; segunda etapa ejecuta el `.jar` resultante sobre una imagen JRE liviana (`eclipse-temurin`, variante `-jre` sobre base Alpine/slim).
- **Frontend (React/Vite):** primera etapa compila los assets estáticos con Node; segunda etapa los sirve mediante Nginx sobre una imagen `nginx:alpine`.

El uso de imágenes multietapa evita que las herramientas de build (Maven, Node, código fuente, dependencias de desarrollo) terminen en la imagen final, reduciendo su tamaño y su superficie de ataque.

# 2.2 Buenas prácticas aplicadas

- **Imágenes base minimalistas:** `eclipse-temurin:*-jre-alpine` para los backends y `nginx:alpine` para el frontend, en lugar de imágenes base completas.
- **`.dockerignore`:** se excluyen del contexto de build carpetas como `target/`, `node_modules/`, `.git/` y archivos de configuración local, evitando que se filtren al build y acelerando el proceso.
- **Variables de entorno:** ningún valor de configuración (endpoint de base de datos, credenciales, puertos) queda hardcodeado en la imagen; todos se inyectan en tiempo de ejecución a través de la Task Definition de ECS.
- **Un proceso por contenedor:** cada contenedor ejecuta un único proceso (la aplicación Java o Nginx), facilitando el escalado y el diagnóstico independiente de cada componente.

# 2.3 Orquestación local con Docker Compose

Para el entorno de desarrollo local se define un archivo `docker-compose.yml` que levanta los tres servicios junto con una instancia local de MySQL, replicando la topología de producción a menor escala:

- Red interna dedicada (`bridge`) para la comunicación entre contenedores por nombre de servicio.
- Volumen persistente para los datos de MySQL, de forma que la información sobreviva a reinicios del contenedor.
- Variables de entorno definidas en el propio `docker-compose.yml` (o mediante un archivo `.env` no versionado) para desarrollo local, análogas a las que se inyectan en producción vía Task Definition.
- Mapeo de puertos hacia el host (`80`, `8080`, `8081`, `3306`) para permitir pruebas manuales sin necesidad de desplegar en la nube.

# 2.4 Publicación de imágenes

Las imágenes resultantes de cada Dockerfile se publican en repositorios independientes de Amazon ECR — ver [03-registro-imagenes.md](03-registro-imagenes.md) — como parte del pipeline de CI/CD.
