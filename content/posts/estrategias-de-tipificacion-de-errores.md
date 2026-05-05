---
title: "Estrategias de Tipificación de Errores: Del Caos a la Arquitectura Limpia"
date: '2026-05-05T00:22:00-03:00'
description: "Guía sobre tipificación de errores y el patrón Result para construir software resiliente y desacoplado."
categories: ["Desarrollo", "Arquitectura"]
tags: ["PHP", "TypeScript", "Go", "Clean Architecture", "Patrones de Diseño"]
author: "Sebastian"
---

En el desarrollo de software, existe una tendencia peligrosa a tratar los errores como "accidentes" o eventos excepcionales que simplemente interrumpen el flujo normal. Sin embargo, en sistemas de alta disponibilidad y arquitecturas limpias (como **Hexagonal** o **DDD**), un error no es una interrupción: **es un dato más del dominio**.

Tipificar errores significa dejar de lanzar strings o excepciones genéricas y empezar a modelar los fallos como ciudadanos de primera clase. Si tu código no puede distinguir entre un usuario que no existe y una base de datos que no responde, tu aplicación está operando a ciegas.

## ¿Por qué dejar de usar excepciones para todo?
El uso excesivo de `throw` y `catch` crea un flujo de control "saltarín" que dificulta el seguimiento de la lógica. Además, en muchos lenguajes, generar un *stack trace* es costoso en términos de rendimiento. Aquí es donde entra el **Patrón Result**.

## El Patrón Result: El error como valor de retorno

El patrón **Result** (o *Either* en programación funcional) propone que una función siempre devuelva un contenedor. Este contenedor informa explícitamente si la operación fue un éxito o un fallo, obligando al desarrollador a manejar ambos escenarios.

**Beneficios clave:**
*   **Contrato Explícito:** La firma del método te avisa: "Esto puede fallar".
*   **Semántica Clara:** Permite usar errores específicos como `UserNotFound` en lugar de mensajes genéricos.
*   **Transmisión entre Capas:** Facilita que un error viaje desde el Repositorio hasta el Controlador sin perder su identidad.

## Implementación Unificada: El caso `UserNotFound`

Vamos a ver cómo implementar este patrón de forma equivalente en tres lenguajes distintos, buscando que el error sea predecible y fácil de gestionar.

### 1. PHP (Enfoque Arquitectura Hexagonal)
En PHP, aprovechamos las clases `readonly` (disponibles desde la 8.2) para crear un contenedor inmutable.

```php
<?php
namespace App\Domain\Shared;

readonly class Result {
    private function __construct(
        public mixed $value = null,
        public ?string $error = null,
        public bool $success = true
    ) {}

    public static function ok(mixed $value): self {
        return new self(value: $value);
    }

    public static function fail(string $error): self {
        return new self(error: $error, success: false);
    }
}

// Ejemplo de uso
function findUser(int $id): Result {
    $user = UserRepository::find($id);
    if (!$user) {
        return Result::fail("UserNotFound: El usuario con ID $id no existe.");
    }
    return Result::ok($user);
}

$res = findUser(7);
if (!$res->success) {
    Log::error($res->error);
}
```

### 2. TypeScript (Seguridad de tipos total)
Aquí usamos *Discriminated Unions*, lo que permite que el IDE nos asista perfectamente.

```typescript
type Result<T, E> = 
  | { success: true; data: T } 
  | { success: false; error: E };

interface User { id: number; name: string; }

function findUser(id: number): Result<User, string> {
  const user = db.users.find(u => u.id === id);
  
  if (!user) {
    return { success: false, error: `UserNotFound: ID ${id} no encontrado` };
  }
  
  return { success: true, data: user };
}

const result = findUser(7);
if (result.success) {
  console.log(result.data.name); // Acceso seguro al dato
} else {
  console.error(result.error); // Manejo del error tipificado
}
```

### 3. Go (El estándar del "Error como Valor")
En Go, el Patrón Result es el estándar por defecto del lenguaje. A diferencia de otros lenguajes que requieren contenedores adicionales, Go utiliza el retorno múltiple para tratar el error como un valor de primer orden.

```go
package main

import (
	"fmt"
	"errors"
)

type User struct {
	ID   int
	Name string
}

// En Go, el "Result" es la tupla de retorno (T, error)
func FindUser(id int) (*User, error) {
	user, err := db.QueryUser(id)
	if err != nil {
		return nil, fmt.Errorf("UserNotFound: ID %d no encontrado: %w", id, err)
	}
	return user, nil
}

func main() {
	user, err := FindUser(7)
	if err != nil {
		fmt.Println("FALLO:", err.Error())
		return
	}
	fmt.Println("ÉXITO:", user.Name)
}
```
## Otros patrones para la gestión de errores

Aunque el **Patrón Result** es extremadamente versátil, la industria utiliza otros enfoques dependiendo del problema específico que intentamos resolver. Como desarrolladores, nuestra tarea es elegir la herramienta adecuada para cada escenario.

### Null Object Pattern
En lugar de devolver un error o un valor nulo, devolvemos un objeto con comportamiento "neutro". Por ejemplo, un `AnonymousUser` que permite que el sistema siga funcionando sin lanzar excepciones ni llenar el código de condicionales `if ($user === null)`.

**Ejemplos:**

**1. PHP (Uso de Herencia e Interfaces)**

En PHP, este patrón es muy potente para limpiar la capa de servicios.

```php
<?php

interface UserInterface {
    public function getName(): string;
    public function canPost(): bool;
}

class RealUser implements UserInterface {
    public function getName(): string { return "Sebastian"; }
    public function canPost(): bool { return true; }
}

class NullUser implements UserInterface {
    public function getName(): string { return "Invitado"; }
    public function canPost(): bool { return false; }
}

// Uso
function getUser($id): UserInterface {
    $user = UserRepository::find($id);
    return $user ?? new NullUser();
}

// No necesitas chequear si es null
echo getUser(999)->getName(); // Imprime "Invitado"
```

**2. TypeScript (Interfaces y Objetos Literales)**

TypeScript facilita mucho esto mediante el uso de interfaces y objetos predefinidos.

```typescript
interface User {
  name: string;
  permissions: string[];
}

const NullUser: User = {
  name: "Guest",
  permissions: []
};

function findUser(id: number): User {
  const user = db.users.find(u => u.id === id);
  return user || NullUser;
}

// La lógica fluye sin interrupciones
const user = findUser(404);
console.log(user.name); // "Guest"
```

**3. Go (Estructuras y Métodos Receptores)**

En Go, el Null Object se implementa con una estructura que satisface una interfaz pero cuyos métodos tienen implementaciones vacías.
 
```go
package main

import "fmt"

type UserProvider interface {
	GetRole() string
}

type RegisteredUser struct{}
func (u RegisteredUser) GetRole() string { return "Admin" }

type NullUser struct{}
func (u NullUser) GetRole() string { return "Guest" }

/*
En entornos de alta concurrencia o sistemas grandes en Go, a veces se define 
el Null Object como una variable global única (singleton) para ahorrar memoria, 
ya que al ser un objeto sin estado, no necesitas instanciarlo cada vez
*/
var DefaultGuest = NullUser{}

func GetUser(id int) UserProvider {
	if id == 0 {
		return DefaultGuest
	}
	return RegisteredUser{}
}

func main() {
	// No hace falta chequear 'if user != nil'
	user := GetUser(0)
	fmt.Println("Rol del usuario:", user.GetRole()) // "Guest"
}
```

### Notification Pattern
Ideal para procesos de validación masiva (como formularios complejos). En lugar de fallar ante el primer error, el sistema recolecta todos los fallos en una "bolsa de notificaciones" y los devuelve juntos al final del proceso.

### Problem Details (RFC 7807)
Un estándar de la industria para APIs HTTP. Define una estructura JSON común para que cualquier cliente (frontend o móvil) sepa exactamente cómo interpretar un error técnico, usando campos estandarizados como `type`, `title` y `status`.

### Try Pattern (Monad)
Muy común en lenguajes como Scala o Java (vía librerías), donde se encapsula una operación que puede lanzar una excepción en un contenedor que permite encadenar operaciones de forma segura antes de manejar el posible fallo.

### Comparativa: ¿Qué patrón elegir según el escenario?

| Patrón | Cuándo usarlo |
| :--- | :--- |
| **Result Pattern** | Cuando el fallo es un resultado esperado del negocio y el sistema **debe** reaccionar a él (ej. "Pago rechazado" o "Stock agotado"). |
| **Null Object Pattern** | Cuando el fallo no es crítico y el sistema puede seguir operando con un comportamiento neutro por defecto (ej. "Usuario invitado"). |
| **Notification Pattern** | Cuando necesitás recolectar **múltiples errores** antes de informar al usuario, evitando que el proceso se detenga en el primero (ej. "Validación de un formulario complejo"). |
| **Problem Details** | Cuando necesitás un **estándar de comunicación externa** para que los clientes de tu API (Frontend, Mobile) entiendan los errores de forma uniforme. |
| **Try Pattern (Monad)** | Cuando querés encadenar varias operaciones que pueden fallar de forma funcional y fluida, abstrayendo los bloques `try-catch` tradicionales. |

## Ubicación

La clave está en entender que cada capa tiene una responsabilidad distinta y, por ende, los errores que nacen en ellas también la tienen. No todos los fallos deben tratarse igual ni guardarse en el mismo sitio.

Aquí detallo cómo se distribuyen los errores según la responsabilidad de cada capa:

### 1. Capa de Dominio (Domain Errors)
Representan violaciones a las reglas de negocio. Son errores que ocurrirían aunque la aplicación no tuviera una base de datos o una interfaz web.
*   **Ejemplos:** `UserNotFoundException`, `InsufficientFundsError`, `InvalidEmailFormat`.
*   **Ubicación:** Dentro de la carpeta o módulo de **Domain**.
*   **Regla de oro:** El dominio no debe conocer conceptos de infraestructura (como HTTP o SQL). Un error de dominio debe ser puro y agnóstico.

### 2. Capa de Aplicación (Application Errors)
Son errores que surgen durante la orquestación de un proceso o caso de uso. No son reglas de negocio per se, sino fallos en el flujo de trabajo.
*   **Ejemplos:** `EmailServiceUnavailable`, `JobExecutionFailed`, `InvalidCommandData`.
*   **Ubicación:** Dentro de la carpeta o módulo de **Application**.
*   **Regla de oro:** Manejan la interacción entre el dominio y los servicios externos. Si un servicio de terceros falla, la aplicación es quien levanta la mano.

### 3. Capa de Infraestructura (Infrastructure Errors)
Son fallos técnicos puros ligados a una implementación específica (base de datos, sistema de archivos, APIs externas).
*   **Ejemplos:** `QueryException`, `RedisConnectionError`, `DiskFullException`.
*   **Ubicación:** Dentro de la carpeta o módulo de **Infrastructure**.
*   **Regla de oro (Mapeo):** Estos errores **nunca** deben llegar a las capas internas. Deben ser capturados en el "adaptador" (por ejemplo, el Repositorio) y convertidos en un error que el dominio o la aplicación sí entiendan.
    *   *Ejemplo:* Capturas un error de conexión a la DB (infra) y devuelves un `ServiceUnavailable` (aplicación).

### Resumen de Distribución

| Capa | Responsabilidad | ¿Dónde se define? |
| :--- | :--- | :--- |
| **Dominio** | Reglas de negocio e invariantes. | `src/Domain/Errors` |
| **Aplicación** | Orquestación de Casos de Uso. | `src/Application/Errors` |
| **Infraestructura** | Implementaciones técnicas y externas. | `src/Infrastructure/Errors` |

### El caso de Vertical Slice Architecture
En **Vertical Slice**, la lógica no se divide por capas técnicas, sino por funcionalidades (slices). 
*   **Ubicación:** Los errores se suelen colocar dentro de la carpeta de la funcionalidad específica (ej. `Features/Users/CreateUser/Errors`).
*   **Diferencia:** Si un error es muy común (como un `UserNotFound`), se puede mover a una carpeta de errores compartidos (`Shared/Errors`), pero la prioridad es mantener todo lo relacionado con una característica en un solo lugar para facilitar el mantenimiento.

## ¿Cómo se comunican? (Mapeo de Errores)
Para que el sistema sea realmente limpio, necesitas una capa final (usualmente un **Middleware** o un **Global Error Handler** en la capa de entrada) que se encargue de traducir estos errores tipificados a algo que el usuario final entienda:
*   `UserNotFoundException` (Dominio) -> **HTTP 404**
*   `ValidationError` (Aplicación) -> **HTTP 400**
*   Cualquier error no controlado -> **HTTP 500**

De esta manera, tu lógica de negocio permanece intacta y protegida de conceptos externos.

## El rol del espacio Shared

Sí, la existencia de un espacio **Shared** (o `Common`) es prácticamente una necesidad en cualquier arquitectura de mediana a gran escala, aunque su uso y "permisividad" varían según el estilo arquitectónico. 

El objetivo del **Shared** es evitar la duplicación de lógica transversal que no pertenece a un solo dominio o funcionalidad, sino que es una utilidad para todo el sistema.

### 1. En Arquitectura Hexagonal y Clean Architecture
Aquí el **Shared** suele estructurarse también por capas. Es común ver una carpeta `src/Shared` que replica la estructura interna:

*   **Shared/Domain:** Contiene "Value Objects" genéricos (como un `Email`, `UUID` o `Money`), interfaces de repositorios base y, por supuesto, **excepciones de dominio comunes** (como `EntityNotFound` o `AccessDenied`).
*   **Shared/Infrastructure:** Implementaciones de adaptadores que usan múltiples módulos, como un cliente de búsqueda (Elasticsearch) o un Logger.

### 2. En DDD (Domain-Driven Design): El "Shared Kernel"
En DDD, el concepto es más formal y se llama **Shared Kernel** (Núcleo Compartido).
*   Se usa para definir elementos que dos o más **Bounded Contexts** (contextos delimitados) necesitan compartir para interoperar.
*   **Precaución:** El Shared Kernel debe ser pequeño y estable. Si cambia constantemente, rompe todos los módulos que dependen de él. Es el lugar ideal para los **Result Patterns** genéricos o tipos de error base que todos los contextos deben entender.

### 3. En Vertical Slice Architecture
Como esta arquitectura busca reducir el acoplamiento entre "slices" (rebanadas), el **Shared** es un tema de debate.
*   Se usa un **Shared Folder** para componentes de infraestructura (como la configuración de la base de datos) y mediadores de mensajes.
*   Para los errores, si un error es estrictamente de una funcionalidad (ej. `InvalidDiscountCode`), vive en su Slice. Si el error es transversal (ej. `Unauthenticated`), se mueve al **Shared**.

### ¿Qué debería ir en un Shared de Errores?
Independientemente de la arquitectura, para que el **Shared** no se convierta en un "cajón de sastre" (un basurero de código), solo deberías poner:

1.  **Clases Base / Contenedores:** La definición del patrón `Result` o la clase abstracta `AppError`.
2.  **Errores Transversales:** Aquellos que no dependen del contexto de negocio, como `InternalServerError`, `ServiceUnavailable` o `RequestTimeout`.
3.  **Interfaces de Mapeo:** Definiciones de cómo un error debe ser transformado antes de salir al exterior.

### El riesgo del Shared
El mayor peligro es que el **Shared** se vuelva una dependencia circular o que termine conteniendo lógica de negocio que debería estar protegida en un dominio específico. 

> **Regla de oro:** Si para modificar una regla de negocio de "Usuarios" tenés que tocar el `Shared`, entonces esa lógica está mal ubicada. El `Shared` debe ser **infraestructura técnica** o **contratos básicos**, no reglas de decisión.

## Conclusión

Ya sea que estés trabajando en el backend con **PHP**, desarrollando una SPA con **TypeScript** o construyendo microservicios en **Go**, la filosofía es la misma: **El manejo de errores es diseño de software.**

Tipificar tus fallos y usar patrones como **Result** no solo hace que tu código sea más limpio, sino que reduce drásticamente los comportamientos inesperados en producción. Como solemos decir en arquitectura: "Haz que los estados ilegales sean imposibles de representar".