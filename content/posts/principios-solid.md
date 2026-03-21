+++
date = '2026-03-19T23:18:17-03:00'
draft = false
title = 'Principios SOLID'
tags = ['Clean Code', 'Best Practices', 'Software Architecture']
+++
Es un conjunto de cinco reglas de diseño orientado a objetos. Su objetivo es crear software que sea fácil de mantener, escalar y entender a lo largo del tiempo.

### S: Single Responsibility (Responsabilidad Única)

> "Una clase debería tener una sola razón para cambiar".

Si tenés una clase Factura que guarda datos en la base de datos, envía emails y además calcula el total, está mal. Si el formato del email cambia, tenés que tocar la clase; si la base de datos cambia, también.

```typescript
class Factura {
    calcularTotal() { /* lógica */ }
    guardarEnBaseDeDatos() { /* lógica */ }
    enviarEmail() { /* lógica */ }
}
```

Usando SOLID

```typescript
class Factura {
    calcularTotal() { /* solo lógica de cálculo */ }
}
class FacturaRepository {
    guardar(factura) { /* solo DB */ }
}
class EmailService {
    enviar(factura) { /* solo comunicación */ }
}
```

### O: Open/Closed (Abierto/Cerrado)

> "Las entidades de software deben estar abiertas para la extensión, pero cerradas para la modificación".

No deberías tener que modificar el código que ya funciona para agregar una funcionalidad nueva.

Ejemplo: Si tenés un ProcesadorDePagos que usa if/else para validar el tipo de pago, cada vez que quieras aceptar una nueva forma de pago (como "Crypto"), vas a tener que modificar la clase original. Esto es peligroso porque podrías romper los métodos de "efectivo" o "tarjeta" que ya funcionaban perfectamente.

```typescript
class ProcesadorDePagos {
    procesar(pago) {
        if (pago.tipo === 'efectivo') { /* ... */ }
        else if (pago.tipo === 'tarjeta') { /* ... */ }
        // Si quiero agregar "Crypto", tengo que 
        // MODIFICAR esta clase.
    }
}
```
En lugar de tocar el procesador, creamos clases independientes para cada método de pago. El ProcesadorDePagos ahora es ciego a los detalles: solo sabe que cualquier pago que reciba tiene un método .procesar().Para agregar "Crypto", simplemente creás una clase nueva class PagoCrypto y se la pasás al procesador. Extendiste la funcionalidad del sistema sin modificar ni una sola línea del código que ya era estable.

```typescript
class PagoEfectivo { procesar() { /* lógica efectivo */ } }
class PagoTarjeta { procesar() { /* lógica tarjeta */ } }

class ProcesadorDePagos {
    procesar(pago) {
        // No importa qué medio sea, mientras sepa 
        // procesarse a sí mismo.
        pago.procesar(); 
    }
}
```

### L: Liskov Substitution (Sustitución de Liskov)

> "Los objetos de la clase base deben poder ser sustituidos por objetos de sus subclases sin que el programa falle".

Imagina que tienes una función que espera un Animal (la base/superior):

- **El escenario**: function hacerCaminar(Animal $a) { ... }
- **La prueba de Liskov**: Si tú le pasas un Perro (la subclase) a esa función, el programa no debe romperse.
- **La lógica**: Como el Perro extiende de Animal, debería "saber hacer" todo lo que un Animal hace. Si el Perro rompe alguna promesa de la clase Animal (como lanzar una excepción donde el padre no lo hacía), entonces no es un buen sustituto.

Para que no se te olvide:

- **Clase Base (Padre)**: Es el "molde" o contrato general.
- **Subclase (Hijo)**: Es la implementación específica.

**Liskov dice**: "Si mi código funciona con el Padre, debe seguir funcionando si le doy al Hijo".

Si intentaras hacerlo al revés (meter un Animal donde se espera un Perro), el programa fallaría porque al Animal le faltarían las cosas específicas del Perro (como ladrar). Por eso la sustitución siempre es hacia arriba (el hijo reemplaza al padre).

Si tenés una clase base Archivo. Asumís que todos los archivos se pueden escribir. Pero de repente, necesitás manejar archivos de Solo Lectura (como un archivo de sistema protegido).

```typescript
class Archivo {
    escribir(datos) { 
        return "Datos guardados"; 
    }
}

class ArchivoSoloLectura extends Archivo {
    escribir(datos) {
        // ¡ERROR! Rompés la expectativa del programa 
        // porque este archivo NO puede escribir.
        throw new Error("No tengo permisos de escritura"); 
    }
}

// El programa falla si intenta guardar cambios en una 
// lista de archivos mezclados.
```

La solución (con SOLID): No obligues a una clase a heredar comportamientos que no posee. La solución es crear jerarquías que realmente separen las capacidades. Así, el programa solo llamará a .escribir() en objetos que garantizan que pueden hacerlo.

```typescript
class Archivo {
    abrir() { /* Lógica común para todos */ }
}

// Solo los archivos que realmente permiten cambios 
// heredan de esta clase
class ArchivoEditable extends Archivo {
    escribir(datos) { /* Lógica de guardado */ }
}

class ArchivoSoloLectura extends Archivo {
    // Aquí no existe el método escribir, por lo 
    // tanto no hay riesgo de error.
    leer() { /* Lógica de lectura */ }
}
```
- **Seguridad**: Tu código ya no tiene que "adivinar" si un archivo va a lanzar un error al intentar escribir.
- **Contratos claros**: Si una función recibe un ArchivoEditable, tenés la garantía total de que el método .escribir() funciona.

### I: Interface Segregation (Segregación de Interfaz)

> "Es mejor tener muchas interfaces específicas que una sola interfaz general".

No obligues a una clase a implementar métodos que no usa. Si creás una interfaz única AccionesDocumento, obligás a que un PDF de solo lectura tenga que implementar un método para "editar", lo cual no tiene sentido y suele terminar con un error o un método vacío.

```typescript
// Interfaz "gorda" que obliga a todo
class DocumentoPDF {
    leer() { /* abre el archivo */ }
    imprimir() { /* envía a la impresora */ }
    editar() { 
        // ¡ERROR! Un PDF de solo lectura no debería 
        // tener este método.
        throw new Error("Operación no soportada"); 
    }
}
```

La solución: Dividimos las responsabilidades en interfaces pequeñas (en lenguajes como TypeScript usarías interface, en JS usamos clases base o simplemente dividimos la lógica). Así, cada clase solo firma el "contrato" de lo que realmente sabe hacer.

```typescript
// Definimos interfaces específicas: Lectura, 
// Edición, Impresión

class ReporteExcel {
    leer() { /* ... */ }
    editar() { /* ... */ }
    imprimir() { /* ... */ }
    // Este documento implementa las tres porque puede.
}

class ArchivoConfiguracion {
    leer() { /* ... */ }
    editar() { /* ... */ }
    // No implementa 'imprimir' porque no tiene 
    // sentido imprimir un archivo .env o .sys
}

class PDFProtegido {
    leer() { /* ... */ }
    imprimir() { /* ... */ }
    // No implementa 'editar', cumpliendo con la 
    // segregación.
}
```
- **Código más limpio**: No tenés métodos "basura" que solo lanzan errores.
- **Mantenimiento**: Si cambia la forma de imprimir, solo tocás las clases que realmente imprimen, sin riesgo de afectar la lógica de edición.

### D: Dependency Inversion (Inversión de Dependencia)

> "Dependé de abstracciones, no de clases concretas".

Tu código de alto nivel no debería depender de los detalles (bajo nivel).

Ejemplo: Tu aplicación no debería depender directamente de "MySQL". Debería depender de una interfaz "BaseDeDatos". Así, si mañana querés cambiar a MongoDB, solo cambiás el motor sin reescribir toda la lógica de tu negocio.

```typescript
class App {
    constructor() {
        this.baseDatos = new MySQL(); // Estamos "atados" 
        // a MySQL para siempre.
    }
}
```

Código SOLID: Dependemos de una "idea" (interfaz), no de una marca.

```typescript
class App {
    constructor(servicioDB) { 
        this.baseDatos = servicioDB; // Recibo CUALQUIER DB 
        // que cumpla con los métodos.
    }
}
const miApp = new App(new MongoDB()); // Inyectamos la dependencia.
```

### Resumen

> #### 5E3A:
> La memoria nos puede fallar con los nombres técnicos, pero lo importante es entender el concepto de fondo. Yo lo resumo así para tenerlo siempre a mano:
>
> * **S**: Responsabilidad Única.
> * **O**: Abierto para extensión, **cerrado** para modificación.
> * **L**: Contratos claros (Sustitución).
> * **I**: Separación de interfaces (No obligues a implementar lo que no se usa).
> * **D**: Depender de abstracciones, no de implementaciones.