# Workflows reutilizables

Tareas compartidas que otros repositorios invocan mediante `workflow_call`. Cada consumidor decide cuándo ejecutarlas y proporciona su configuración.

## Workflows

| Workflow | Propósito | Activación |
| --- | --- | --- |
| [Comprobar identidad AWS](aws-identity-check.yml) | Autenticar y comprobar que la cuenta y el ARN coinciden con los esperados. | Llamada desde otro workflow. |
| [Validar Node.js](node-npm-jest-ci.yml) | Instalar dependencias, comprobar formato y lint, ejecutar pruebas y compilar. Las pruebas de integración son opcionales. | Llamada desde otro workflow. |
| [Construir y publicar en ECR](docker-build-publish-ecr.yml) | Construir una imagen Docker y publicarla en ECR cuando corresponda. | Llamada desde otro workflow. |
| [Validar workflows](ci-workflows.yml) | Revisar los workflows de este repositorio con actionlint. | Push a `main` y pull requests con cambios en `.github/workflows`; también manual. Es CI propio. |

## Operación

| Aspecto | Funcionamiento |
| --- | --- |
| Ejecución | Máquinas de **GitHub Actions** con Ubuntu. |
| Versionado | Los consumidores referencian una versión publicada mediante `uses: .../archivo.yml@version`. |
| Identidad AWS | Recibe región, cuenta, ARN esperado y los secretos `AWS_ACCESS_KEY_ID` y `AWS_SECRET_ACCESS_KEY`. |
| Node.js | Recibe `working-directory` y `run-integration-tests`; utiliza la versión de Node indicada en `package.json` y los scripts npm de la aplicación. |
| Construcción Docker | Recibe `context`, `dockerfile` e `image-name`; si se omite el nombre, utiliza el del repositorio consumidor. |
| Publicación ECR | Requiere `AWS_IMAGE_PUBLISH_ENABLED=true`, evento `push` y rama `main`. En otras condiciones construye sin publicar. |
| Configuración ECR | Variables del consumidor: `AWS_REGION`, `AWS_ACCOUNT_ID` y `AWS_IMAGE_PUBLISH_ENABLED`. Secretos: `AWS_ECR_PUBLISHER_ACCESS_KEY_ID` y `AWS_ECR_PUBLISHER_SECRET_ACCESS_KEY`. |
| Preparación ECR | El repositorio de imágenes y los permisos del publicador deben existir previamente. |
| Etiqueta de imagen | `sha-<commit>`. Si ya existe en ECR, se conserva la imagen publicada y se omite su reconstrucción. |

Publicar una imagen no actualiza la versión desplegada en Kubernetes. Esa selección se declara por separado en `paideia-infra-config`.
