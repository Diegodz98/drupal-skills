# Registro en services.yml y puentes legacy

> **Leer este archivo solo cuando compatibilidad legacy = SÍ.**  
> Si legacy = NO, no registrar la clase de hooks en ningún archivo de servicios.

---

## Registro en `MODULE.services.yml` / `THEME.services.yml`

### Caso básico con autowiring

```yaml
services:
  Drupal\my_module\Hook\MyModuleHooks:
    class: Drupal\my_module\Hook\MyModuleHooks
    autowire: true
```

### Con alias para interfaz no resuelta por autowiring

Si una dependencia inyectada no se resuelve automáticamente, agregar alias:

```yaml
services:
  Drupal\my_module\Hook\MyModuleHooks:
    class: Drupal\my_module\Hook\MyModuleHooks
    autowire: true

  my_module.real_service:
    class: Drupal\my_module\Service\RealService

  Drupal\my_module\Contract\RealServiceInterface: '@my_module.real_service'
```

**Cuándo agregar alias:**
- La clase de hooks inyecta una interfaz en el constructor.
- Esa interfaz no tiene un alias de tipo registrado en el contenedor.
- Sin el alias, Drupal lanzaría error de resolución en el autowiring.

---

## Wrapper procedural con `#[LegacyHook]`

El wrapper reemplaza el cuerpo original y delega al servicio OOP.

### Sin retorno (`void`)

```php
use Drupal\Core\Hook\Attribute\LegacyHook;
use Drupal\my_module\Hook\MyModuleHooks;

/**
 * Implements hook_user_cancel().
 */
#[LegacyHook]
function my_module_user_cancel(array &$edit, UserInterface $account, string $method): void {
  \Drupal::service(MyModuleHooks::class)->userCancel($edit, $account, $method);
}
```

### Con retorno

```php
/**
 * Implements hook_entity_access().
 */
#[LegacyHook]
function my_module_entity_access(EntityInterface $entity, string $operation, AccountInterface $account): AccessResultInterface {
  return \Drupal::service(MyModuleHooks::class)->entityAccess($entity, $operation, $account);
}
```

**Reglas del wrapper:**
- Agregar `#[LegacyHook]` justo antes de la función.
- Mantener el docblock `Implements hook_xxx()`.
- Mantener la firma exacta (mismos tipos, referencias `&`, defaults).
- Si la función original tenía `return`, el wrapper también debe retornar.
- Usar `\Drupal::service(HookClass::class)->metodo(...)` como delegación.

---

## Checklist para escenario legacy

- [ ] Clase de hooks registrada en `MODULE.services.yml` o `THEME.services.yml`.
- [ ] Si hay interfaz sin alias → alias creado en el mismo archivo de servicios.
- [ ] Wrapper procedural con `#[LegacyHook]` por cada hook migrado.
- [ ] Wrapper delega correctamente al servicio.
- [ ] Si hay retorno → wrapper devuelve el resultado de la delegación.
- [ ] `use` de `LegacyHook` y de la clase de hooks presentes en el archivo `.module`.