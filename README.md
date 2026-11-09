# Automatizaciones de Paideia Code

Procesos reutilizables para validar aplicaciones, construir y publicar imágenes
y ejecutar operaciones comunes de infraestructura.

## Estado

Separación acordada el 9 de noviembre de 2026. Esta carpeta contiene la base
documental; todavía no hay workflows implementados ni se ha publicado desde aquí
un nuevo repositorio GitHub.

## Responsabilidad

| Repositorio | Contenido |
| --- | --- |
| paideia-infra-terraform | Módulos escritos para Terraform |
| paideia-infra-platform | Charts y configuración común de Kubernetes |
| paideia-infra-automations | Workflows reutilizables y auxiliares comunes |
| paideia-infra-environments | Configuración y política de cada ambiente |

Los repositorios de aplicaciones conservan sus propios eventos de CI y parámetros.
Environments conserva las entradas para promover versiones y levantar o retirar
ambientes. Esos workflows pequeños llaman a las piezas comunes de este repositorio.

Un workflow reutilizable no decide por sí solo qué aplicación desplegar ni crea
AWS al publicarse. Los consumidores eligen una referencia revisada y versionada.

## Estructura prevista

```text
.github/workflows/
  ci-node.yml
  ci-java.yml
  ci-go.yml
  build-image.yml
scripts/                  # Solo cuando un procedimiento necesite un auxiliar
```

Los workflows reutilizables estarán directamente en `.github/workflows/`, sin
subcarpetas. Recibirán parámetros mediante `workflow_call`. Crearemos las piezas
al comprobar su uso, comenzando con pruebas y construcción del template Nest.

Consulta el [diseño general](../paideia-infra-platform/docs/arquitectura.md) y el
[flujo de despliegues](../paideia-infra-platform/docs/despliegues.md).
