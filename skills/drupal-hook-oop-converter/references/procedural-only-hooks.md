# Hooks solo procedurales en Drupal 11

Estos hooks **no pueden migrarse a OOP** con `#[Hook]`. Si el usuario pide convertirlos, informar que no tienen soporte OOP actualmente y preguntar si quiere mantener el soporte procedural sin cambios.

## Lista completa

| Hook | Motivo |
|---|---|
| `hook_hook_info()` | Solo procedural por diseño del sistema |
| `hook_module_implements_alter()` | Altera el orden de implementaciones, debe ser procedural |
| `hook_install()` | Ciclo de vida del módulo, vive en `.install` |
| `hook_install_tasks()` | Instalación, solo procedural |
| `hook_install_tasks_alter()` | Instalación, solo procedural |
| `hook_post_update_NAME()` | Actualizaciones de datos, vive en `.post_update.php` |
| `hook_removed_post_updates()` | Registro de actualizaciones eliminadas |
| `hook_requirements()` | Verificación de requisitos, solo procedural |
| `hook_schema()` | Definición de esquema de base de datos |
| `hook_uninstall()` | Ciclo de vida del módulo, vive en `.install` |
| `hook_update_N()` | Migraciones de base de datos, vive en `.install` |
| `hook_update_dependencies()` | Dependencias entre updates |
| `hook_update_last_removed()` | Registro de updates eliminados |

## Qué decirle al usuario

Cuando se detecte uno de estos hooks en el código a migrar:

> El hook `hook_xxx` no tiene soporte OOP en Drupal 11 actualmente. ¿Querés mantener el soporte procedural sin modificaciones?

- **SÍ** → dejar la función tal cual, sin cambios, sin agregar `#[LegacyHook]`.
- **NO** → omitir del resultado de la migración e informar que fue excluido.

## Nota sobre `system_theme`

La función `system_theme` también se excluye de la conversión automática por su comportamiento especial en el sistema de temas.