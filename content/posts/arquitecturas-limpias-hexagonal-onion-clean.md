---
title: "Arquitecturas Limpias: Hexagonal, Onion y Clean"
date: 2026-04-11
tags: ["Architecture", "Hexagonal", "DDD", "Clean Architecture", "Software Engineering"]
categories: ["Technical Design"]
author: "Sebastian"
description: "Una inmersión profunda en la genealogía de las arquitecturas desacopladas: de Alistair Cockburn a Uncle Bob."
---
En el desarrollo de software de alto nivel, el éxito no se mide por qué tan rápido entregamos una funcionalidad, sino por qué tan fácil es cambiarla seis meses después. Nuestro trabajo diario es una lucha constante contra el acoplamiento y la entropía del código.

Hoy vamos a desglosar las tres arquitecturas que definieron la ingeniería moderna: **Hexagonal**, **Onion** y **Clean Architecture**. Aunque comparten un ADN común y a menudo se usan como sinónimos, entender sus matices es lo que nos permite diseñar sistemas que no colapsan bajo su propio peso.

## 1. El Núcleo Común: La Regla de Dependencia

Antes de entrar en cada autor, hay un principio universal que las une: **Las dependencias solo pueden apuntar hacia adentro.**

Este concepto rompe con el modelo tradicional de "3 capas" donde la lógica de negocio dependía de la base de datos. Aquí, el "Interior" es el **Dominio** (tus reglas de negocio, tus entidades). El "Exterior" son los detalles técnicos (Bases de datos, APIs de terceros, Frameworks). 

En un sistema bien diseñado, el Dominio es "agnóstico". No sabe si los datos se guardan en un `.txt`, en PostgreSQL o si la petición viene de una API REST o un comando de consola. Esta **Inversión de Dependencia** es la que nos permite testear el negocio sin levantar pesadas infraestructuras.

## 2. Inmersión por Autor y Modelo

### A. Arquitectura Hexagonal (Ports & Adapters) por Alistair Cockburn (2005)

Cockburn propuso este modelo para resolver el problema de las aplicaciones que terminan fuertemente acopladas a sus interfaces de usuario o a sus bases de datos, lo que impedía el testeo automatizado y la evolución tecnológica.

**La Metáfora** 

El Hexágono no implica "seis lados", sino que representa una figura con múltiples caras para conectar diversos "Puertos".

**Puertos (Interfaces)** 

Son los puntos de entrada o salida. Definen el contrato de *qué* necesita la aplicación. Por ejemplo, un puerto `PaymentGateway` dice: "Necesito procesar un pago", pero no sabe cómo.

**Adaptadores (Implementaciones)** 

Son los que contienen la tecnología. Un adaptador de `Stripe` implementa el puerto `PaymentGateway`.

* **Driving Adapters** (Primarios)

    - Otros nombres: Inbound, Input, Left-side, o simplemente Entradas.
    - Se llaman "Driving" (conductores) porque ellos conducen a la aplicación. Son los que inician la acción. La aplicación está "dormida" hasta que uno de estos adaptadores le da una orden.
    - ¿Quiénes son?: Un controlador de una API REST, un comando de CLI, un test unitario, o un proceso de cron.
    - En criollo: Es "lo que dispara" el caso de uso.

* **Driven Adapters** (Secundarios)
    
    - Otros nombres: Outbound, Output, Right-side, o simplemente Salidas (o Infraestructura).
    - Se llaman "Driven" (conducidos) porque son conducidos por la aplicación. La aplicación es la que decide cuándo llamarlos para completar su tarea.
    - ¿Quiénes son?: La base de datos (MySQL, MongoDB), un servicio de mail (SendGrid), una cola de mensajes (RabbitMQ), o una API externa (Stripe).
    - En criollo: Es "lo que la app necesita usar" para terminar el trabajo.
    
La gran diferencia entre ambos es dónde vive la Interface (Puerto):

   - En los Driving, la interfaz suele ser el Caso de Uso mismo (la aplicación expone un método que el controlador llama).
   - En los Driven, la interfaz vive en el Dominio/Aplicación (el puerto), pero la implementación vive en la Infraestructura (el adaptador).

**Diagrama de Flujo: Puertos y Adaptadores**

```text
MUNDO EXTERIOR (Entrada)             MUNDO EXTERIOR (Salida)
+-----------------------+           +------------------------+
|                       |           |                        |
|   [ADAPTADORES        |           |   [ADAPTADORES         |
|    PRIMARIOS]         |           |    SECUNDARIOS]        |
|                       |           |                        |
|  - Controlador API    |           |  - Repositorio MySQL   |
|  - Comando CLI        |           |  - Cliente Stripe      |
|  - Test Unitario      |           |  - Servicio de Email   |
|                       |           |                        |
+-----------+-----------+           +-----------+------------+
            |                                   ^
            | (Dispara)                         | (Llama)
            v                                   |
+-----------+-----------------------------------+------------+
|           |          [ EL HEXÁGONO ]          |            |
|           |        (Lógica de Negocio)        |            |
|           v                                   |            |
|     [ PUERTOS DE ]                     [ PUERTOS DE ]      |
|       ENTRADA                            SALIDA            |
|     (Interfaces)                       (Interfaces)        |
|           |                                   ^            |
|           +----------> [ CASO DE ] -----------+            |
|                        [  USO  ]                           |
|                                                            |
+------------------------------------------------------------+
```

1. **Lado Izquierdo (Driving / Primarios):** Son los "Disparadores". El flujo de control va hacia la aplicación. El adaptador primario llama a un puerto de entrada (el Caso de Uso).

**Ejemplo:** El controlador HTTP "conduce" a la aplicación para que cree un pedido.

2. **El Centro (Core):** Aquí están tus reglas. No dependen de nadie. Solo exponen contratos (Interfaces).

3. **Lado Derecho (Driven / Secundarios):** Son los "Colaboradores". El flujo de control sale desde la aplicación. La aplicación llama a un puerto de salida y el adaptador secundario (infraestructura) lo ejecuta.

**Ejemplo:** La aplicación "conduce" al adaptador de MySQL para que guarde los datos.

En conclusión, la arquitectura hexagonal se resume en una máxima: las interfaces hablan con el Core y el Core solo conoce interfaces. Dentro de nuestros Casos de Uso, interactuamos exclusivamente con Entidades (para el estado y reglas críticas) y Puertos (interfaces para la persistencia o servicios externos). Esto garantiza que la lógica de negocio sea un componente puro, testeable y, sobre todo, independiente de si los datos viajan por JSON, se guardan en SQL o se envían por una cola de mensajes.

### B. Onion Architecture por Jeffrey Palermo (2008).

Palermo llevó la idea de Cockburn un paso más allá, enfocándose en la organización interna de las capas y eliminando cualquier dependencia hacia la infraestructura.

**La Metáfora** 

Capas concéntricas. La gran diferencia aquí es el énfasis en las capas de servicios de dominio.

**¿Qué son los Servicios de Dominio?**

En el centro de la cebolla están las Entities (Entidades). Pero a veces tenés lógica que involucra a varias entidades o que no "encaja" naturalmente dentro de una sola. Ahí es donde entran los Domain Services.

- **Ubicación:** Están justo por encima de las Entidades, pero debajo de los Servicios de Aplicación (Casos de Uso).

- **Regla de Oro:** Solo contienen lógica de negocio pura. No conocen la base de datos, no conocen HTTP, no conocen el sistema de archivos.

**Ejemplo Práctico:** Transferencia Bancaria - Imagina que tenés la entidad Account (Cuenta).

**La Entidad (Account):** Sabe cómo aumentar o disminuir su propio saldo. Es lógica interna.

**El Servicio de Dominio (TransferService):** Es el que sabe que para una transferencia entre dos cuentas, una debe tener saldo suficiente, ambas deben estar activas y el movimiento debe ser atómico.

**Por qué es de Dominio:** Porque la regla de "cómo se transfiere" es una regla de negocio que no cambia si usas MySQL o MongoDB.

**El Servicio de Aplicación (ExecuteTransferUseCase):** Es el que usa al servicio de dominio. Se encarga de buscar las cuentas en el repositorio, llamar al TransferService y guardar los cambios.

**Diferencia clave:** Domain Service vs. Application Service

Domain Service, responsabilidad lógica de negocio compleja y multientidad. Es el "Cómo" funciona el negocio. Ejemplo: CalculateOrderDiscount, ValidateUserUniqueness.

Application Service, responsabilidad orquestación, persistencia y seguridad. Es el "Qué" quiere hacer el usuario. Ejemplo: CreateUser, ProcessPayment, SendWelcomeEmail.

**Domain Model**

Está en el centro absoluto y no tiene dependencias. Define el estado y el comportamiento esencial del negocio.

1. ¿Qué vive dentro del Domain Model?

El modelo de dominio se compone de diferentes elementos que trabajan juntos:

-- **Entities (Entidades):** Objetos que tienen una identidad única que persiste en el tiempo (por ejemplo, un Usuario con un ID o un Pedido con un número de seguimiento). Las entidades contienen lógica de negocio que afecta a sus propios datos.

-- **Value Objects (Objetos de Valor):** Objetos que no tienen identidad propia y se definen solo por sus atributos (ejemplo: un Email, una Dirección o una Moneda). Si dos Email tienen el mismo texto, son el mismo objeto. Son inmutables por naturaleza.

-- **Aggregates (Agregados):** Grupos de entidades vinculadas, lideradas por una Raíz (Aggregate Root), que garantizan la integridad de los datos. El mundo exterior solo interactúa con la Raíz, asegurando que todas las reglas de negocio se validen como una unidad atómica antes de persistir cualquier cambio. Ejemplo: Pedido (Order), el Order es la Raíz del Agregado. Los OrderItems son entidades internas. Si querés cambiar la cantidad de un ítem, no llamás al ítem directamente. Llamás por ejemplo a  ```order.updateItemQuantity(itemId, quantity)```. El Agregado verifica si el pedido no está ya enviado antes de dejarte cambiar la cantidad.

-- **Domain Exceptions:** Errores específicos del negocio (ejemplo: InsufficientFundsException). Estos errores deben nacer aquí porque son reglas de negocio, no fallos técnicos.

-- **Domain Events (Eventos de Dominio):** Bonus track. Son cosas que pasaron en el negocio y que a otras partes del sistema les interesa saber (ej: PedidoConfirmado).

2. **La característica clave: La Pureza**

La regla de oro de Jeffrey Palermo para el Domain Model es que no debe tener dependencias externas.

- **No conoce la Base de Datos:** No importa si usas SQL o JSON.
- **No conoce el Framework:** No sabe qué es Laravel, Gin o Express.
- **No conoce la UI:** No sabe si los datos se mostrarán en una web o en una app móvil.

Esto hace que el Domain Model sea la parte más estable de tu aplicación. Los frameworks cambian cada dos años, pero la regla de "un pedido no puede ser enviado si no está pagado" puede durar décadas.

3. **El problema del "Modelo de Dominio Anémico"**

Este es un concepto que debés evitar. Un modelo es anémico cuando tus entidades son solo bolsas de datos (solo tienen getters y setters) y toda la lógica está en los servicios.

- **Mal (Anémico):** El servicio le cambia el estado a la entidad manualmente.
- **Bien (Rico):** La entidad tiene métodos como $order->confirmPayment() que validan las reglas internas antes de cambiar el estado.

4. **¿Cómo encaja en la Onion?**

Imagina que la aplicación es un organismo vivo:

- **Domain Model:** Es el ADN y los órganos vitales. Define quién es y qué puede hacer el organismo.
- **Domain Services:** Son los procesos fisiológicos (como la digestión) que coordinan varios órganos. Capa que rodea al modelo, donde reside la lógica que involucra a múltiples entidades.
- **Application Services:** Es el sistema nervioso que recibe estímulos externos y decide qué procesos disparar. La capa que orquesta el flujo de datos. Aquí es donde se "abren" las transacciones y se llaman a los servicios externos.

**Inversión Total** 

En el modelo tradicional, la UI dependía de la lógica y la lógica de la DB. En la Onion, **toda la infraestructura depende del centro**.

### C. Clean Architecture por Robert C. Martin "Uncle Bob" (2012).

Es la síntesis definitiva. Uncle Bob tomó "Ports & Adapters" y "Onion" y les dio una estructura jerárquica estricta basada en niveles de abstracción.

**1. Entities (Empresa/Negocio)**

Son las reglas de negocio de alto nivel. Lo que no cambiaría aunque mañana decidas dejar de ser un blog y pases a ser una tienda. Son los objetos que contienen la lógica más pura.

**2. Use Cases (Aplicación/Interactors)**

Aquí es donde reside la lógica específica de tu software actual. Orquestan el flujo de datos desde y hacia las entidades. El corazón de Clean. Contienen la lógica específica de la aplicación. Si el negocio dice "Un usuario solo puede comprar si tiene saldo", esa regla vive aquí. Un punto clave es que la arquitectura debe ser **Screaming** (Gritar su intención): al ver las carpetas, deberías ver `CreateOrder` y `RefundPayment`, no `OrderController`.

**3. Interface Adapters (Adaptadores)**

Esta capa es la "aduana". Traduce los datos de los Casos de Uso a un formato que sea conveniente para la base de datos o para la web. Aquí es donde viven los Controllers, Presenters y Gateways.

**4. Frameworks & Drivers (El Mundo Exterior)**

Esta es la capa más externa y volátil. Es donde vive el código que realmente "hace cosas" con el hardware o la red: 
    - El servidor web (Gin, Laravel, Express).
    - El motor de base de datos (PostgreSQL, MongoDB).
    - Herramientas de UI o dispositivos.

**Estructuras de carpetas**

No confundamos Arquitectura con Estructura de Carpetas. La arquitectura define las reglas de dependencia y cómo fluye la información. La estructura de carpetas es cómo organizamos esos archivos para que el equipo sea más productivo. En proyectos grandes, la organización por Features (Vertical Slices) combinada con Clean Architecture es la opción más robusta para evitar el 'código espagueti' entre módulos."

**Organizadas por capas (Package by Layer)**

Se agrupan los archivos según su rol técnico (todas las entidades juntas, todos los casos de uso juntos).

- **Pros:** Es muy fácil entender la arquitectura del sistema con solo mirar las carpetas.
- **Contras:** Cuando el proyecto crece, para modificar una sola funcionalidad (ej: "Pedidos"), tenés que saltar entre 4 o 5 carpetas muy lejanas entre sí.

```
src/
├── Domain/              # CAPA 1: Entidades (Reglas de negocio globales)
│   ├── Entities/        # Objetos con identidad (User, Order)
│   ├── ValueObjects/    # Objetos por valor (Email, Money)
│   └── Exceptions/      # Errores de negocio
├── Application/         # CAPA 2: Casos de Uso (Interactors)
│   ├── UseCases/        # Acciones específicas (CreateOrder, Login)
│   └── Ports/           # Interfaces que la App necesita (Repository, Mailer)
├── InterfaceAdapters/   # CAPA 3: Adaptadores (Traductores)
│   ├── Controllers/     # Reciben la entrada (HTTP, CLI)
│   ├── Presenters/      # Formatean la salida para el cliente
│   └── Gateways/        # Implementan los puertos de la App (RepositoyImpl)
└── Infrastructure/      # CAPA 4: Frameworks y Drivers (Herramientas)
    ├── Database/        # Configuración de MySQL, MongoDB, etc.
    ├── Framework/       # Configuración de Laravel, Gin, Express
    └── ExternalAPIs/    # Clientes de Stripe, AWS, etc.
```

**¿Por qué esta estructura es agnóstica?**

**Independencia de Framework:** La lógica vive en Domain y Application. Si mañana cambiás de lenguaje o de framework, esas carpetas se mantienen casi idénticas en su lógica.

**Separación de "Qué" vs "Cómo":** * En Application/UseCases definís qué hace tu app.

En Infrastructure/ definís cómo lo hace técnicamente.

**Flujo de Dependencia:** Las flechas de dependencia siempre van de afuera hacia adentro. Domain no puede importar nada de Application, y Application no puede importar nada de Infrastructure.

**Estructura por Funcionalidades (Package by Feature)**

Aquí la carpeta raíz no es la capa, sino la funcionalidad del negocio. Dentro de cada "Feature", aplicás la arquitectura (Clean, Hexagonal, etc.).

```
src/
├── Orders/              # Feature de Pedidos
│   ├── Domain/          # Entidades de Pedidos
│   ├── Application/     # Casos de Uso de Pedidos
│   └── Infrastructure/  # Repositorio de Pedidos
├── Users/               # Feature de Usuarios
│   ├── Domain/
│   ├── Application/
│   └── Infrastructure/
└── Shared/              # Código común a todas las features
```

**¿Cuál es mejor?**

La respuesta siempre es "depende", pero hay una tendencia clara:

**Proyectos Chicos/Medianos:** Package by Layer funciona perfecto y mantiene la estructura clara.

**Proyectos Grandes/Microservicios:** Package by Feature es superior porque reduce la carga cognitiva. Si tenés que tocar algo de "Pagos", todo lo que necesitás está dentro de la carpeta Payments/.

**La "Arquitectura Gritona" (Screaming Architecture)**

Uncle Bob (Clean Architecture) es un gran defensor de que la estructura de carpetas debe gritar qué hace la aplicación. Si abrís un proyecto y ves carpetas como Controllers, Models y Views, la arquitectura grita "¡Soy una app de Rails o Laravel!".

Pero si abrís el proyecto y ves Orders, Catalog y Shipping, la arquitectura grita "¡Soy un E-commerce!". Eso es organizar por Features.

**Estructura Híbrida: Features + Clean Architecture**

```
src/
├── Billing/              # Dominio de Facturación (Feature)
│   ├── Domain/           # Entidades y Reglas de Facturación
│   ├── Application/      # Casos de Uso (Interactors)
│   └── Infrastructure/   # Implementaciones (DB Facturación, PDFs)
│
├── Catalog/              # Dominio de Catálogo (Feature)
│   ├── Domain/           # Entidades (Product, Category)
│   ├── Application/      # Casos de Uso (SearchProducts, UpdateStock)
│   └── Infrastructure/   # Adaptadores (ElasticSearch, MySQL)
│
├── Shared/               # Lo que todos necesitan (Núcleo común)
│   ├── Domain/           # Value Objects comunes (Money, Email)
│   └── Infrastructure/   # Logger, Bus de Eventos, Framework Config
│
└── UserInterface/        # Capa de entrada global (Driving Adapters)
    ├── Http/             # Rutas y Controladores que llaman a las Features
    └── Console/          # Comandos de CLI
```

**¿Por qué esto es "Clean"?**

Aunque los archivos no están agrupados por "todas las entidades del sistema en una sola carpeta", la Regla de Dependencia se sigue cumpliendo a rajatabla dentro de cada feature:

- **Aislamiento:** Si tenés un bug en Facturación, no tenés que tocar nada en Catálogo.
- **Screaming Architecture:** Al abrir src/, ves Billing y Catalog. El proyecto te está "gritando" que es un sistema comercial.
- **Facilidad de Extracción:** Si mañana la feature de Billing crece mucho, es muy fácil sacarla a un microservicio aparte porque ya está físicamente separada del resto.

## 3. Cuadro Comparativo de Referencia

| Característica | Hexagonal (Cockburn) | Onion (Palermo) | Clean (Uncle Bob) |
| :--- | :--- | :--- | :--- |
| **Foco** | Simetría Entrada/Salida. | Capas concéntricas estables. | Casos de Uso y Abstracción. |
| **Elemento Clave** | Puertos y Adaptadores. | Domain Services. | Entities e Interactors. |
| **Flujo de Control** | A través de Puertos. | Hacia el centro. | Regla de Dependencia estricta. |
| **Nivel de Rigidez** | Flexible (estilo libre). | Moderado (capas definidas). | Alto (estándar visual). |

## 4 . Resumen
Si tuviera que darte una regla rápida para decidir cómo organizar tus archivos:

**Hexagonal:** Organizá por "Dirección" (¿Quién inicia a quién?). Es ideal para sistemas donde tenés múltiples puntos de entrada (Web, Worker, Cron) y múltiples salidas. Úsala en microservicios o aplicaciones donde necesites cambiar de adaptadores frecuentemente.

**Onion:** Organizá por "Nivel de Abstracción" (¿Qué tan cerca del negocio está esto?). Es ideal para dominios ricos donde la lógica es la protagonista. Úsala cuando tu dominio es extremadamente complejo y el estado de los objetos es la prioridad.

**Clean:** Organizá por "Intención" (¿Qué acción está realizando el usuario?). Es ideal para sistemas grandes donde necesitás encontrar un flujo de negocio (un Caso de Uso) sin navegar por 20 controladores. Úsala en sistemas de largo aliento donde la claridad de los casos de uso sea vital para la entrada de nuevos desarrolladores.

Lo importante no es el nombre, sino que el negocio sea el rey. Si podés cambiar tu base de datos de MySQL a MongoDB o tu framework de Laravel a Symfony (o de Gin a Echo) sin tocar una sola línea de tu lógica de negocio, entonces has alcanzado la maestría arquitectónica.

Como siempre recomiendo en este espacio, validá estos conceptos con la documentación oficial de los autores y adaptalos a las necesidades específicas de tu arquitectura actual. No hay una "bala de plata", pero sí hay una estructura que hará que tu "yo del futuro" no te odie cuando tenga que hacer un cambio.