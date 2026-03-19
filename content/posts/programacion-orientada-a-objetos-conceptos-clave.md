+++
date = '2026-03-10T22:04:39-03:00'
draft = false
title = 'Programacion orientada a objetos: Conceptos clave'
tags = ["OOP"]
+++
La **Programación Orientada a Objetos (POO)** es un paradigma de programación que nos permite organizar el código pensando en "objetos" de la vida real en lugar de solo funciones y variables sueltas. Es la base para crear software escalable y fácil de mantener.

En los lenguajes de programación orientado a objetos, un objeto es como una "entidad" que tiene sus propios datos (propiedades) y sus propias acciones (métodos).

* * *

1\. Clases y Objetos: El Molde y el Producto
--------------------------------------------

Imagina que una **Clase** es el plano de una casa, y el **Objeto** es la casa real construida. Puedes construir muchas casas usando el mismo plano.


```php
class Coche {
    public $marca;
    public $color;

    public function encender() {
        return "El coche está encendido.";
    }
}

$miCoche = new Coche();
$miCoche->marca = "Toyota";
echo $miCoche->encender();
```

2\. Abstracción: Solo lo importante
-----------------------------------

La **Abstracción** consiste en ocultar los detalles complejos y mostrar solo lo necesario. Para conducir un coche, no necesitas saber cómo funciona la combustión interna, solo necesitas el volante y los pedales.

3\. Encapsulamiento: Protegiendo los datos
------------------------------------------

El **Encapsulamiento** es como un escudo. Usamos palabras como `private` o `protected` para que nadie pueda modificar datos sensibles desde fuera de la clase sin permiso.

```php
class CuentaBancaria {
    private $saldo = 0;

    public function depositar($monto) {
        if($monto > 0) {
            $this->saldo += $monto;
        }
    }

    public function verSaldo() {
        return "Saldo actual: $" . $this->saldo;
    }
}
```

4\. Herencia: Reutilizando código
---------------------------------

La **Herencia** permite que una clase "hija" herede propiedades y métodos de una clase "padre". Es ideal para no repetir código.

```php
class Vehiculo {
    public function tocarBocina() {
        return "¡Beep beep!";
    }
}

class Moto extends Vehiculo {
    // La moto ya tiene el método tocarBocina() por herencia
}
```

5\. Polimorfismo: Muchas formas
-------------------------------

El **Polimorfismo** significa que diferentes clases pueden responder al mismo mensaje de formas distintas. Por ejemplo, tanto un Perro como un Gato son "Animales", pero el método `hacerSonido()` será diferente para cada uno.

```php
interface Animal {
    public function hacerSonido();
}

class Perro implements Animal {
    public function hacerSonido() { return "Guau!"; }
}

class Gato implements Animal {
    public function hacerSonido() { return "Miau!"; }
}
```

6\. Interfaces: El contrato de tu código
----------------------------------------

Una **Interfaz** es como un contrato legal. No define cómo se hacen las cosas, sino **qué** cosas se deben poder hacer. Es una lista de métodos que cualquier clase que la implemente está obligada a tener.

Las interfaces son fundamentales para el **Polimorfismo** y para que nuestro código no dependa de clases específicas, sino de comportamientos.

```php
interface MetodoPago {
    public function procesarPago($monto);
}

class PagoPaypal implements MetodoPago {
    public function procesarPago($monto) {
        return "Procesando $" . $monto . " a través de PayPal API.";
    }
}

class PagoTarjeta implements MetodoPago {
    public function procesarPago($monto) {
        return "Procesando $" . $monto . " con pasarela de Tarjeta.";
    }
}
```

#### **¿Por qué son tan importantes?** 

Porque podés crear una función que reciba un `MetodoPago`. A esa función no le importa si le pasás PayPal o Tarjeta; ella sabe que, sea lo que sea, tendrá un método llamado `procesarPago()` disponible. Esto hace que tu sistema sea increíblemente fácil de extender: si mañana querés agregar "Pago con Cripto", solo creás la clase y listo, no tenés que tocar el resto del sistema.

#### **¿Por qué las interfaces son el puente a SOLID?** 

Las interfaces son las protagonistas del quinto principio de SOLID (DIP - Dependency Inversion Principle), que dice que debemos depender de abstracciones (interfaces) y no de implementaciones (clases concretas).

7\. Clases Abstractas: El punto medio
-------------------------------------

Una **Clase Abstracta** es una mezcla entre una clase normal y una interfaz. A diferencia de las interfaces, una clase abstracta **puede tener código real** (métodos con lógica), pero al igual que las interfaces, **no se puede usar para crear objetos directamente**.

Se usan cuando tenés varias clases que comparten mucha lógica, pero querés obligarlas a todas a implementar ciertos métodos específicos.

```php
abstract class Personaje {
    public $nombre;

    // Método con código (todos los personajes saludan igual)
    public function saludar() {
        return "Hola, soy " . $this->nombre;
    }

    // Método abstracto (cada personaje ataca de forma distinta)
    abstract public function atacar();
}

class Guerrero extends Personaje {
    public function atacar() {
        return "Ataca con espada!";
    }
}

class Mago extends Personaje {
    public function atacar() {
        return "Lanza un hechizo!";
    }
}
```

8\. ¿Interfaz o Clase Abstracta?
--------------------------------

Esta es la duda de todo desarrollador. Aquí la regla de oro:

*   Usá una **Interfaz** cuando quieras definir un **comportamiento** (un contrato) que muchas clases distintas pueden tener (ej: "Cualquier cosa que se pueda pagar").
*   Usá una **Clase Abstracta** cuando las clases tengan una **relación de identidad** ("Es un...") y quieras compartir código base entre ellas para no repetir (DRY).

#### ¿Qué significa exactamente "Es un..."?
--------------------------------------

Significa que la clase hija **pertenece a la misma familia** que la clase madre. No solo hace lo mismo, sino que conceptualmente **es** lo mismo.

Aquí tienes algunos ejemplos para que quede clarísimo:

*   **En un sistema de empleados:**
    *   `Gerente` **es un** `Empleado`.
    *   `Programador` **es un** `Empleado`.
    *   La clase abstracta `Empleado` define datos comunes (sueldo, legajo) que todos son y tienen.
*   **En un sistema de archivos:**
    *   `ArchivoPDF` **es un** `Documento`.
    *   `ArchivoWord` **es un** `Documento`.
    *   Ambos comparten la identidad de ser un documento, aunque se abran de forma distinta.

#### La diferencia clave: "Es un..." vs "Puede hacer..."
---------------------------------------------------

Esta es la distinción que separa a los juniors de los seniors:

*   La **Clase Abstracta** define la **identidad** ("Qué es").
*   La **Interfaz** define una **capacidad** ("Qué puede hacer" o "Actúa como").

#### ¿Por qué esto es vital en PHP y POO?
------------------------------------

Si usas mal la herencia (clases abstractas) para cosas que no tienen una relación de identidad, terminas con un código **rígido y frágil**.

**Un error común:** Hacer que una clase `Usuario` herede de una clase `BaseDeDatos`.

*   ¿Un `Usuario` **es una** `BaseDeDatos`? **No.**
*   Un `Usuario` **usa** una `BaseDeDatos`.

Ahí es donde entra la **composición** o las **interfaces**, pero nunca la herencia. La herencia se reserva exclusivamente para cuando la identidad es la misma.

9\. Herencia vs. Composición: ¿Cuál elegir?
-------------------------------------------

En la Programación Orientada a Objetos, tenemos dos formas de darle funcionalidades a nuestras clases. Entender cuándo usar cada una es lo que separa a un programador que "hace que funcione" de uno que "diseña software".

#### La Herencia (Relación "Es un...")
-------------------------------------

Como vimos antes, la herencia crea una relación jerárquica rígida. La clase hija hereda todo de la madre. Es excelente cuando la identidad es la misma, pero tiene un problema: **es frágil**. Si cambias algo en la clase madre, puedes romper todas las hijas sin querer.

```php
// Herencia rígida
class Ave {
    public function volar() {
        return "Estoy volando";
    }
}

class Pinguino extends Ave {
    // ¡Problema! Un pingüino es un ave, pero NO vuela. 
    // Aquí la herencia nos obliga a heredar algo que no necesitamos.
}
```

#### La Composición (Relación "Tiene un...")
-------------------------------------------

La composición consiste en crear clases pequeñas que hacen una sola cosa y luego "armar" tu objeto principal usándolas. En lugar de decir que algo **es**, decimos que algo **tiene** un comportamiento.

Es mucho más flexible porque puedes cambiar las piezas en tiempo de ejecución como si fueran bloques de LEGO.

```php
// Composición flexible
class ComportamientoVuelo {
    public function ejecutar() {
        return "Estoy volando";
    }
}

class Pinguino {
    private $vuelo;

    // El pingüino no hereda "Volar", decide si lo tiene o no.
    public function \_\_construct($comportamiento = null) {
        $this->vuelo = $comportamiento;
    }
}
```

#### ¿Cuál es mejor?
---------------

Existe una máxima en el diseño de software: **"Prefiere la composición sobre la herencia"**.

*   **Usa Herencia** solo cuando la relación es permanente y lógica (Un `Gato` siempre será un `Mamifero`).
*   **Usa Composición** cuando quieras que tu código sea fácil de cambiar, probar y extender sin crear jerarquías gigantescas de clases que nadie entiende.

* * *

10\. Modificadores de Visibilidad: Public, Protected y Private
--------------------------------------------------------------

No todo en una clase debe ser accesible desde afuera. Controlar quién ve qué es la clave del **Encapsulamiento**.

*   **Public:** Es la configuración por defecto. Cualquiera puede ver y modificar esta propiedad o método desde fuera de la clase.
*   **Protected:** Solo la propia clase y sus **hijas** (herencia) pueden acceder. Es ideal para cosas que queremos compartir en la familia pero no con extraños.
*   **Private:** Solo la propia clase puede verlo. Ni siquiera las hijas tienen acceso. Es el nivel máximo de seguridad.

```php
class Base {
    public $abierto = "Todos me ven";
    protected $familia = "Solo mis hijos y yo";
    private $secreto = "Solo yo lo sé";
}
```

11\. El modificador Final: "Hasta aquí llegamos"
------------------------------------------------

A veces querés evitar que otros programadores (o vos mismo en el futuro) extiendan una clase o sobrescriban un método. Para eso existe `final`.

*   **Clase Final:** No puede tener hijas. Se corta la herencia ahí.
*   **Método Final:** Una clase hija puede heredar la clase, pero NO puede cambiar cómo funciona ese método específico.

```php
final class Segura {
    // Nadie puede hacer "class Otra extends Segura"
}

class Padre {
    final public function reglaDeOro() {
        return "Esto no se puede cambiar";
    }
}
```

12\. Static: Lo que le pertenece a la Clase, no al Objeto
---------------------------------------------------------

Normalmente necesitás hacer `new Clase()` para usar sus métodos. Pero los métodos o propiedades **Static** le pertenecen a la clase en sí. No necesitás crear un objeto para usarlos.

```php
class Calculadora {
    public static $pi = 3.1416;

    public static function sumar($a, $b) {
        return $a + $b;
    }
}

// Se usa así, sin el "new":
echo Calculadora::sumar(5, 10);
```

#### ¿Por qué esto es importante para lo que sigue?
----------------------------------------------

*   **Encapsulamiento:** Sin `private` y `protected`, no podrías aplicar el **Principio de Responsabilidad Única (SRP)** de forma segura. Si cualquiera puede meter mano en los datos internos de tu clase desde fuera, pierdes el control sobre cómo y cuándo cambian esos datos.
*   **Static:** Se usa muchísimo en los **Patrones de Diseño** (como el _Singleton_ o las _Factories_). Entender que algo le pertenece a la clase y no a la instancia es clave para herramientas de configuración y utilidades que seguramente tocaremos más adelante en el blog.

* * *

Conclusión: ¿Por qué usar POO?
------------------------------

Usar POO en tus proyectos te permite aplicar los principios de **Clean Code**. Facilita el trabajo en equipo, permite reutilizar lógica y prepara tu aplicación para crecer sin convertirse en un caos de funciones desordenadas.