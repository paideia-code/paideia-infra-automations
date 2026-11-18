# paideia-infra-automations

Tareas reutilizables de GitHub Actions. Los repositorios consumidores deciden cuándo ejecutarlas y proporcionan entradas, permisos y credenciales según cada tarea.

## Tareas disponibles

| Tarea | Propósito | Workflow |
| --- | --- | --- |
| **Node.js / npm / Jest** | Comprobar formato y lint, ejecutar pruebas y compilar. | [node-npm-jest-ci.yml](.github/workflows/node-npm-jest-ci.yml) |
| **Docker / GHCR** | Construir y publicar una imagen con una versión explícita. | [docker-build-publish-ghcr.yml](.github/workflows/docker-build-publish-ghcr.yml) |
| **Docker / ECR** | Construir imágenes y publicarlas en ECR cuando corresponda. | [docker-build-publish-ecr.yml](.github/workflows/docker-build-publish-ecr.yml) |
| **Identidad AWS** | Comprobar la cuenta y la identidad esperadas. | [aws-identity-check.yml](.github/workflows/aws-identity-check.yml) |
| **CI del repositorio** | Validar los workflows con actionlint en push y pull requests a sus archivos; también manualmente. | [ci-workflows.yml](.github/workflows/ci-workflows.yml) |

Las cuatro primeras tareas se invocan mediante `workflow_call` y se ejecutan en runners Ubuntu de GitHub Actions. Los procedimientos específicos de sandbox están en `paideia-infra-config`.

## Uso desde otro repositorio

Ejemplo de `.github/workflows/ci.yml` en una aplicación:

```yaml
name: CI

on: [push, pull_request]

permissions:
  contents: read

jobs:
  validate:
    uses: paideia-code/paideia-infra-automations/.github/workflows/node-npm-jest-ci.yml@v0.9.0
```

La etiqueta referenciada debe estar publicada y contener el workflow. Para distribuir cambios: publicar el commit, comprobar el CI, crear una nueva etiqueta y después actualizar los consumidores. La versión del workflow y la versión de la imagen son independientes.

## Configuración del consumidor

| Tarea | Configuración necesaria |
| --- | --- |
| **Node.js** | `working-directory` (por defecto `.`) y `run-integration-tests` (por defecto `false`). Lee la versión de Node de `package.json` y utiliza los scripts npm del proyecto. |
| **Docker, ambos registros** | `context` (por defecto `.`), `dockerfile` (por defecto `Dockerfile`) e `image-name` (por defecto el nombre del repositorio consumidor). |
| **GHCR** | `version` obligatoria, con formato `0.0.1` o `v0.0.1`; publica la etiqueta `0.0.1` cuando se invoca. El job consumidor concede `contents: read` y `packages: write`. Utiliza el `GITHUB_TOKEN` automático; no requiere secretos adicionales. |
| **ECR** | Variables `AWS_REGION`, `AWS_ACCOUNT_ID` y `AWS_IMAGE_PUBLISH_ENABLED`. Secretos `AWS_ECR_PUBLISHER_ACCESS_KEY_ID` y `AWS_ECR_PUBLISHER_SECRET_ACCESS_KEY`. El repositorio ECR y los permisos deben existir previamente. |
| **Identidad AWS** | Entradas `aws-region`, `aws-account-id` y `expected-principal-arn`; secretos `AWS_ACCESS_KEY_ID` y `AWS_SECRET_ACCESS_KEY`. |

ECR publica solo con `AWS_IMAGE_PUBLISH_ENABLED=true`, evento `push` y rama `main`. En otras condiciones construye sin publicar. Utiliza `sha-<commit>` y conserva la imagen si esa etiqueta ya existe.

En GHCR, el consumidor decide el evento de publicación; por ejemplo, un push del tag Git `v0.0.1`. Para permitir descargas sin autenticación, configurar el paquete como público. Mantener cada versión publicada asociada al mismo contenido.

Publicar una imagen no despliega la aplicación: su versión se selecciona por separado en los manifiestos del repositorio de despliegue.
