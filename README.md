# paideia-infra-automations

Tareas reutilizables de GitHub Actions para los repositorios de Paideia. Los consumidores deciden cuándo ejecutarlas y proporcionan entradas, variables y secretos.

## Tareas disponibles

| Tarea | Propósito | Workflow |
| --- | --- | --- |
| **Node.js / npm / Jest** | Comprobar formato y lint, ejecutar pruebas y compilar. | [node-npm-jest-ci.yml](.github/workflows/node-npm-jest-ci.yml) |
| **Docker / ECR** | Construir imágenes y publicarlas en ECR cuando corresponda. | [docker-build-publish-ecr.yml](.github/workflows/docker-build-publish-ecr.yml) |
| **Identidad AWS** | Comprobar la cuenta y la identidad esperadas. | [aws-identity-check.yml](.github/workflows/aws-identity-check.yml) |

El repositorio también tiene su propio [CI de workflows](.github/workflows/ci-workflows.yml). Los procedimientos específicos de sandbox están en `paideia-infra-config`.

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

La versión referenciada debe estar publicada y contener el workflow. Para distribuir cambios, se publica una nueva versión y después se actualizan sus consumidores.

## Operación

| Aspecto | Funcionamiento |
| --- | --- |
| Contrato | Las tareas reutilizables reciben su configuración mediante `workflow_call`. |
| Credenciales | El consumidor proporciona los secretos necesarios desde GitHub Actions. |
| Recursos AWS | Los repositorios ECR y permisos se preparan previamente en `paideia-infra-config`. |
| Despliegue | La publicación de una imagen y la selección de la versión desplegada son pasos separados. |

Entradas, condiciones de publicación y secretos requeridos: [guía de workflows](.github/workflows/README.md).
