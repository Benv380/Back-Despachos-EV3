# 4. Pipeline CI/CD (GitHub Actions)

# 4.1 Estructura general

Se implementó un pipeline de integración y despliegue continuo **independiente para cada uno de los tres repositorios** (`front-despacho`, `back-despachos`, `back-ventas`), definido como workflow de GitHub Actions (`.github/workflows/deploy.yml` en cada repositorio).

# 4.2 Etapas del pipeline

1. **Trigger:** disparo automático ante cada `push` a la rama `main` del repositorio.
2. **Configuración de credenciales:** se configuran las credenciales de AWS mediante GitHub Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`).
3. **Login en ECR:** autenticación del cliente Docker contra el registro de Amazon ECR.
4. **Build:** construcción de la imagen Docker a partir del Dockerfile multietapa del componente.
5. **Test:** ejecución de las pruebas del proyecto (para los backends, pruebas unitarias con Maven; para el frontend, verificación del build de producción) antes de continuar con el pipeline.
6. **Push:** publicación de la imagen en el repositorio ECR correspondiente, etiquetada con el SHA del commit y `latest`.
7. **Render de Task Definition:** se genera una nueva revisión de la Task Definition de ECS, incorporando la URI de la imagen recién publicada.
8. **Deploy:** despliegue de la nueva Task Definition al servicio ECS correspondiente, esperando a que el servicio alcance un estado estable (`wait-for-service-stability`) antes de marcar el workflow como exitoso.

# 4.3 Gestión de secretos en el pipeline

Las credenciales de AWS nunca se escriben en el código del workflow ni en el repositorio: se almacenan como **GitHub Secrets** y se inyectan como variables de entorno únicamente durante la ejecución del job. Ver detalle en [06-configuracion-secretos.md](06-configuracion-secretos.md).

# 4.4 Limitación del entorno académico

Una limitación propia del entorno AWS Academy Learner Lab es que las credenciales de AWS son temporales y caducan junto con la sesión del laboratorio; por lo tanto, los tres GitHub Secrets deben actualizarse manualmente cada vez que se reinicia el laboratorio.

En un entorno productivo real, esta limitación se resolvería reemplazando las credenciales estáticas/temporales por un **rol IAM federado mediante OpenID Connect (OIDC)** entre GitHub Actions y AWS, eliminando por completo la necesidad de copiar credenciales manualmente. La creación de dicho rol federado no está permitida en el entorno académico utilizado, por lo que queda documentada como mejora para producción (ver [11-conclusiones.md](11-conclusiones.md)).

# 4.5 Tiempos de ejecución

Las ejecuciones del pipeline completan el ciclo build → test → push → deploy en un rango de aproximadamente 1 a 3 minutos por servicio, dependiendo del tamaño de la imagen y del tiempo de estabilización del despliegue en ECS. La etapa que típicamente concentra más tiempo es la construcción de la imagen Docker y la espera de estabilidad del servicio.
