# Estructura de clase OOP para hooks

## Estructura mínima

```php
<?php

declare(strict_types=1);

namespace Drupal\my_module\Hook;

use Drupal\Core\Hook\Attribute\Hook;

/**
 * Hook implementations for my_module.
 */
final class MyModuleHooks {

  #[Hook('nombre_del_hook')]
  public function nombreDelMetodo(/* firma exacta del hook */): void {
    // lógica
  }

}
```

**Reglas:**
- Namespace: `Drupal\<modulo>\Hook`
- Archivo: `src/Hook/<ClassName>.php`
- Docblock de clase: `Hook implementations for <module>.`
- Siempre `use Drupal\Core\Hook\Attribute\Hook;`
- Clase `final`

---

## Con inyección de dependencias (módulos)

```php
<?php

declare(strict_types=1);

namespace Drupal\my_module\Hook;

use Drupal\Core\Hook\Attribute\Hook;
use Drupal\my_module\Service\MyDomainService;

/**
 * Hook implementations for my_module.
 */
final class MyModuleHooks {

  public function __construct(
    private readonly MyDomainService $myDomainService,
  ) {}

  #[Hook('user_cancel')]
  public function userCancel(array &$edit, UserInterface $account, string $method): void {
    $this->myDomainService->handleUserCancel($account, $method);
  }

}
```

**Reglas de DI en módulos:**
- Preferir constructor con propiedades tipadas `private readonly`.
- Métodos de hook delgados: delegar lógica a servicios inyectados.
- No usar `\Drupal::service()` / `\Drupal::entityTypeManager()` / `\Drupal::currentUser()` en métodos OOP.

---

## Sin DI (themes o módulos simples)

```php
final class MyThemeHooks {

  #[Hook('preprocess_node')]
  public function preprocessNode(array &$variables): void {
    // Se puede usar \Drupal::service() directamente en themes.
    $service = \Drupal::service('my_theme.helper');
    $variables['extra'] = $service->compute($variables['node']);
  }

}
```

---

## Variantes de atributo permitidas

### 1. Atributo en método (más común)

```php
#[Hook('user_cancel')]
public function userCancel(...): void {}
```

### 2. Atributo en clase con `method` explícito

```php
#[Hook('user_cancel', method: 'userCancel')]
final class MyModuleHooks {
  public function userCancel(...): void {}
}
```

### 3. Clase invocable con `__invoke`

```php
#[Hook('user_cancel')]
final class UserCancelHook {
  public function __invoke(...): void {}
}
```

### 4. Hook de otro módulo

```php
#[Hook('user_cancel', module: 'other_module')]
public function userCancel(...): void {}
```

---

## Reglas de conversión de nombres

| Procedural | OOP |
|---|---|
| `my_module_user_cancel` | `userCancel` |
| `my_module_form_node_article_form_alter` | `formNodeArticleFormAlter` |
| `my_module_entity_presave` | `entityPresave` |

- Eliminar el prefijo del módulo (`my_module_`).
- Convertir `snake_case` restante a `camelCase`.
- Si `__FUNCTION__` aparece en el cuerpo, reemplazar por el nombre original de la función como string.

---

## Firma exacta

La firma del método OOP debe ser idéntica a la del hook:

- Mismos tipos de parámetros.
- Mismas referencias `&`.
- Mismos valores por defecto.
- Mismo tipo de retorno.

**Ejemplo:**

```php
// Procedural
function my_module_token_info_alter(array &$data): void {}

// OOP — firma idéntica
#[Hook('token_info_alter')]
public function tokenInfoAlter(array &$data): void {}
```