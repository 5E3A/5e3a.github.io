+++
date = '2026-03-19T22:12:16-03:00'
draft = false
title = 'Clean Code: El arte de escribir software para seres humanos'
tags = ['Clean Code', 'Best Practices', 'Software Architecture']
+++
**Clean Code** no es una regla técnica, sino un estándar de cortesía y profesionalismo. Es escribir código pensando en que la siguiente persona que lo lea —que bien podrías ser tú dentro de seis meses— no quiera lanzarte el teclado por la cabeza.

En términos sencillos, el código limpio es aquel que se explica a sí mismo. Si tienes que leer un bloque de código tres veces para entender qué hace, o si necesitas llenar todo de comentarios para explicar "el truco" que usaste, entonces no es código limpio.

### 1. Nombres con intención (Intention-Revealing Names)

Es el primer pecado del código sucio. Las variables y funciones deben decirte por qué existen, qué hacen y cómo se usan.

- **Sucio**: Una variable llamada d para guardar días transcurridos.
- **Limpio**: Una variable llamada diasDesdeLaUltimaActualizacion.

> **5E3A**:
>
> Recuerdo a un colega que solía nombrar variables con una sola letra. Si bien es cierto que en scopes muy reducidos donde el uso es inmediato el impacto no es tan grave, lo ideal es siempre buscar la mayor semántica posible. Personalmente, tampoco soy fan de nombres excesivamente largos que dificultan la lectura; el equilibrio está en que el nombre explique el "qué" sin volverse una distracción.

El nombre debe ser tan claro que no necesites un comentario al lado.

### 2. Funciones que hacen una sola cosa (The Single Responsibility)

- **Una función debe ser como un destornillador**: hace una tarea y la hace bien. Si una función valida un usuario, lo guarda en la base de datos y además envía un email de bienvenida, es una "función monstruo".
- **La regla de oro**: Una función debería tener un solo nivel de abstracción y ser pequeña. Si no puedes explicar lo que hace la función sin usar la palabra "y", es que hace demasiadas cosas.

### 3. El código debe leerse como prosa

Un archivo de código debería leerse como un artículo de un diario. Arriba tienes los conceptos de alto nivel (el título y el resumen) y, a medida que bajas, vas encontrando los detalles técnicos y las implementaciones más pequeñas. Esto permite que alguien pueda "escanear" tu código y entender la lógica general rápidamente.

### 4. Evitar el "Ruido" (Comentarios innecesarios)

En Clean Code, un comentario es a menudo un fracaso. Es un fracaso en intentar expresarte claramente a través del código. En lugar de explicar qué hace un bloque complejo con un comentario, lo ideal es mover ese bloque a una función con un nombre descriptivo. El código debe ser narrativo por sí solo.

### 5. La Regla del Boy Scout

"Deja el campamento un poco más limpio de lo que lo encontraste". Si entras a un archivo para arreglar un error pequeño y ves una variable mal nombrada o una función muy larga, límpiala. Si todos aplicamos esto, el código mejora con el tiempo en lugar de pudrirse.

> **5E3A**:
>
> La realidad es que 'código que funciona no se toca'. Se puede aplicar la regla del Boy Scout, pero si es un sistema sin pruebas (Unit Tests), mejor evitar tocarlo. Si surgen problemas de performance, el riesgo podría justificarse, siempre y cuando se coordine previamente con un colega o líder para evaluar el impacto.

### 6. Evitar el "Código Muerto"

El código limpio no guarda "por si acaso". Si una función ya no se usa, se borra. Confía en tu sistema de control de versiones (como Git) para recuperar algo si lo necesitas. El código comentado que "ya no sirve" solo genera ruido visual y duda: "¿Esto no se usa o es que alguien se olvidó de activarlo?".

### 7. Formateo Vertical y Horizontal

La apariencia visual importa. Un archivo de código debe tener una estructura lógica:

- **Densidad vertical**: Las líneas relacionadas deben estar cerca. No obligues al lector a hacer scroll infinito para encontrar una definición.
- **Distancia vertical**: Usa espacios en blanco para separar conceptos, como si fueran párrafos en un libro.

### 8. Manejo de Errores como Flujo Principal

El manejo de errores no es un añadido, es parte de la lógica. Sin embargo, el código limpio prefiere lanzar excepciones claras en lugar de mezclar la lógica principal con cientos de bloques de captura que hacen invisible el camino "feliz" del código.

### 9. DRY (Don't Repeat Yourself)

La duplicación es la raíz de muchos males. Si tienes la misma lógica repetida, tienes dos lugares donde pueden aparecer bugs. El código limpio extrae esa lógica común a un único punto de verdad.

### 10. La importancia de la Simplicidad (KISS)

**KISS**: Keep It Simple, Stupid. A veces queremos demostrar inteligencia con algoritmos sofisticados cuando una solución simple bastaba. El código limpio es humilde; prefiere la solución que cualquier programador junior pueda entender.

### Conclusión: El Código Limpio y la Deuda Técnica

Escribir código limpio es una estrategia de supervivencia. Cuando ignoramos estos principios para "ir más rápido", pedimos un préstamo llamado Deuda Técnica. Los intereses se pagan mañana con errores difíciles de encontrar y una lentitud desesperante para añadir cambios.

Por el contrario, el Clean Code mantiene el costo del cambio bajo. Es lo que permite que el software siga siendo maleable, que el equipo trabaje sin frustración y que el producto evolucione al ritmo que el negocio necesite.