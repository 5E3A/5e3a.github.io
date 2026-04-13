---
title: "DDD Táctico: El corazón del negocio en el código"
date: 2026-04-12
tags: ["DDD", "Software Design", "Architecture", "Best Practices"]
categories: ["Technical Design"]
author: "Sebastian"
description: "Más allá de las carpetas: cómo el diseño orientado al dominio nos ayuda a domar la complejidad del software."
---

En el [post anterior](/posts/arquitecturas-limpias-hexagonal-onion-y-clean/) hablamos sobre cómo organizar nuestra "casa" usando Arquitecturas Limpias. Si leíste el artículo completo, habrás notado que mencionamos conceptos como **Entidades, Agregados y Eventos**. 

Esos términos no son casualidad: son los pilares de **Domain-Driven Design (DDD)**. Mientras que la arquitectura nos dice dónde poner las piezas para que no se acoplen, DDD nos enseña cómo modelar esas piezas para que el software hable el mismo idioma que el negocio. 

Hoy vamos a hacer un "doble clic" en la parte **táctica** de DDD, profundizando en esas herramientas que mencionamos de pasada y viendo cómo implementarlas de verdad.

## 1. El Lenguaje Ubicuo: El fin de la traducción

El error más común en el desarrollo es la "traducción". El experto de negocio pide un *"Ajuste de stock por merma"* y el programador escribe `updateProductQuantity(-1)`. Seis meses después, nadie sabe por qué ese `-1` está ahí o qué regla de negocio representa.

El **Lenguaje Ubicuo** propone que el código use las mismas palabras que el experto del negocio.
* **Mal:** `user.setStatus(2)`
* **Bien:** `user.markAsDeactivated()`

Si el experto habla de "Suscripciones", "Cuentas" y "Retiros", esas palabras deben ser clases, métodos y eventos en tu código. No traduzcas; integra el lenguaje al código.

## 2. Los Bloques de Construcción: Profundizando en el Modelo

En el post de arquitectura vimos que estas piezas viven en el centro del hexágono o la cebolla. Vamos a ver por qué son tan poderosas:

### A. Value Objects (Objetos de Valor)
Son objetos que se definen por sus atributos, no por una identidad. 
* **Propiedad clave:** Son **Inmutables**. 
* **Ejemplo:** Un `Email` no es solo un string. Es un objeto que se valida a sí mismo al nacer. Si quieres cambiar la dirección de correo, no "editas" el objeto; creas una instancia nueva. Esto elimina los efectos secundarios y hace que tu código sea predecible.

### B. Aggregates (Agregados) y su Raíz
Como vimos anteriormente, un Agregado es un clúster de objetos que se mantienen consistentes. 



* **La Raíz (Aggregate Root):** Es el único punto de contacto. Si tienes un `Pedido` (Order), nadie desde fuera debería poder cambiar el precio de un `OrderItem` directamente. 
* **¿Por qué?** Porque solo el `Pedido` conoce las reglas globales: ¿se puede editar si ya está pagado? ¿aplica un descuento por volumen? El Agregado protege la integridad de los datos.

## 3. Domain Services: La lógica "sin hogar"

A veces hay reglas que involucran a varios Agregados y no "encajan" naturalmente dentro de una sola entidad. 

* **Ejemplo:** *"Transferir dinero entre dos cuentas"*. 
No pertenece a la Cuenta A ni a la Cuenta B. Creamos un **Domain Service** llamado `TransferService`. Es una pieza de lógica pura (sin estado) que reside en el dominio y orquesta esta operación de negocio.

## 4. Domain Events: El lenguaje de lo que ya pasó

Un Evento de Dominio es algo que le importa al experto de negocio y que ya ocurrió: `OrdenCreada`, `PagoRechazado`, `StockAgotado`.

En lugar de que el Caso de Uso de "Crear Orden" tenga que llamar manualmente al servicio de Email, al de Facturación y al de Logística (generando un acoplamiento enorme), el Agregado simplemente dice: **"Ey, esto pasó"**. Otras partes del sistema se suscriben y reaccionan. Esto es lo que permite que tu sistema sea realmente modular.

## 5. El Modelo Rico vs. El Modelo Anémico

Este es el punto de inflexión. 
* **Modelo Anémico:** Clases que son solo bolsas de datos (getters/setters). La lógica está "desparramada" en servicios externos. Es difícil de mantener y fácil de romper.
* **Modelo Rico:** Las entidades tienen comportamiento. El objeto `Order` sabe cómo validarse, cómo calcular su total y cómo cambiar su estado.

DDD nos empuja hacia el **Modelo Rico**. El objetivo es que el código se explique a sí mismo.


## 6. El desafío de la Persistencia: Repositorios y Mapeadores

En las arquitecturas limpias, la base de datos es un "detalle de implementación". Para que esto sea verdad en la práctica, debemos resolver dos problemas comunes:

**A. El desacoplamiento total (Mappers)**

Muchos desarrolladores dicen aplicar Clean Architecture, pero terminan usando las anotaciones o decoradores del ORM (como Hibernate, GORM o Eloquent) dentro de sus Entidades de Dominio.

**El pecado:** Si tu entidad tiene un @Column(name="user_id"), tu dominio ya está "sucio" con detalles de la base de datos.

**La solución:** Mantener tres modelos de datos separados:

* **Request/Response DTOs:** Para la comunicación con el exterior (API).
* **Domain Entities:** Representan el negocio y viven en Core/Domain. No conocen nada técnico.
* **Database Entities/Schemas:** Representan las tablas y viven en Infrastructure/Persistence.

**Aquí entra el Mapeador:** un traductor que vive en la capa de Infraestructura. Su único trabajo es convertir una fila de la base de datos en un objeto de dominio rico, y viceversa. Es más código, pero es lo que te permite cambiar de motor de base de datos sin tocar tu lógica de negocio.

**B. La Regla de "Un Repositorio por Agregado"**

Un error clásico es crear un repositorio por cada tabla de la base de datos.

**El concepto DDD:** Solo la Raíz del Agregado (Aggregate Root) debe tener un repositorio.

**Ejemplo:** Si tenés un Pedido con Items, no hacés un ItemRepository. Hacés un OrderRepository que gestione todo el paquete.

**¿Por qué?** Porque esto garantiza la atomicidad. Al guardar el pedido, el repositorio se asegura de que los ítems también se guarden correctamente, manteniendo la integridad del negocio en una sola operación.

## 7. El precio de la pureza (Trade-offs)

No todo es color de rosa. Implementar DDD táctico y Arquitecturas Limpias tiene un costo que debemos evaluar:

**Indirección:** A veces, seguir el rastro de una acción implica saltar por 5 archivos (Controller -> UseCase -> DomainService -> Repository -> Entity).

**Boilerplate:** Vas a escribir más código inicial (mapeadores, DTOs, interfaces) que en un modelo tradicional de 3 capas.

**Curva de aprendizaje:** No es fácil para un equipo junior entrar en un proyecto con estas reglas sin una buena mentoría.

**Mi consejo:** Si estás haciendo un CRUD simple, no mates una mosca con un cañón. Pero si el negocio es complejo y va a crecer, esta inversión se paga sola en tres meses.

## Conclusión: ¿Por dónde empezar?

Implementar DDD táctico no requiere que cambies todo tu sistema mañana. Puedes empezar por:
1. Identificar un **Value Object** (como una moneda o un rango de fechas) y extraerlo.
2. Buscar un **Agregado** que esté sufriendo problemas de consistencia y cerrar sus fronteras.
3. Lo más importante: **Hablar con los expertos de negocio** y asegurarte de que tu código refleja su realidad.

DDD es el complemento indispensable de las Arquitecturas Limpias. Sin una buena estructura, el dominio se ensucia; sin un buen dominio, la estructura está vacía.

Como siempre, te recomiendo validar estos patrones con la biblia de Eric Evans o el libro rojo de Vaughn Vernon según las necesidades de tu proyecto.