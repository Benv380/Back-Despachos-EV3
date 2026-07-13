# 6. Configuración y gestión de secretos

# 6.1 Principio aplicado

Se siguió el principio de separar **configuración** (no sensible) de **credenciales** (sensibles), y dentro de las credenciales, aplicar mínimo privilegio en su acceso.

# 6.2 Variables de entorno no sensibles

Los parámetros de conexión no sensibles (`DB_ENDPOINT`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`) se gestionan como variables de entorno regulares (bloque `environment`) en la Task Definition de cada backend.

# 6.3 Secretos sensibles: AWS Systems Manager Parameter Store

La contraseña de la base de datos MySQL se gestiona mediante **AWS Systems Manager Parameter Store**, como parámetro de tipo `SecureString` (cifrado), bajo la ruta `/innovatech/DB_PASSWORD`.

La Task Definition de cada backend referencia este parámetro mediante el bloque `secrets`, que inyecta el valor descifrado como variable de entorno (`DB_PASSWORD`) únicamente en tiempo de ejecución del contenedor:

```json
"secrets": [
  {
    "name": "DB_PASSWORD",
    "valueFrom": "arn:aws:ssm:us-east-1:<account-id>:parameter/innovatech/DB_PASSWORD"
  }
]
```

Con este mecanismo, el valor de la contraseña **nunca queda expuesto en texto plano** en la definición de la tarea, en el código fuente, ni en los repositorios de GitHub.

# 6.4 Secretos del pipeline: GitHub Secrets

Las credenciales de AWS necesarias para que GitHub Actions pueda publicar imágenes en ECR y desplegar en ECS (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) se almacenan como **GitHub Secrets** a nivel de repositorio, y se inyectan como variables de entorno solo durante la ejecución del workflow, sin quedar visibles en los logs ni en el historial de commits.

# 6.5 Mínimo privilegio

El acceso a los parámetros de SSM y a ECR/ECS está acotado al rol `LabRole` asignado al entorno de ejecución de las tareas (restricción del entorno académico — ver [05-infraestructura-nube.md](05-infraestructura-nube.md)). En un escenario productivo real, este acceso se limitaría mediante políticas IAM específicas por servicio, en lugar de un rol compartido de alcance amplio.
