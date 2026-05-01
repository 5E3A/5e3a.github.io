---
title: "Arquitectura de APIs Orientadas a Eventos"
date: 2026-04-13
tags: ["Architecture", "Event-Driven", "Microservices", "System Design"]
categories: ["Technical Design"]
author: "Sebastian"
description: "Cómo diseñar sistemas que reaccionan a cambios en tiempo real sin bloquear la ejecución."
---

En los artículos previos sobre **Arquitecturas Limpias** y **DDD**, aprendimos a proteger el núcleo de nuestro negocio. Pero, ¿qué pasa cuando una acción en nuestro sistema debe notificar a otros cinco sistemas? Si usamos llamadas HTTP tradicionales (Request-Response), terminamos con un sistema lento, frágil y altamente acoplado.

Hoy vamos a explorar la **Arquitectura Orientada a Eventos (EDA)**, el patrón que permite que nuestros servicios sean realmente independientes y escalables.

## 1. ¿Qué es una API de Eventos?

A diferencia de una API REST tradicional donde el cliente "pregunta" por el estado, en una API de eventos el sistema "anuncia" que algo ha sucedido.

*   **REST:** "Ey, dame los datos del pedido 123".
*   **Event-Driven:** "¡Atención! El pedido 123 acaba de ser pagado".

Este cambio de paradigma permite que cualquier interesado (otros microservicios, analíticas, logs) reaccione a ese hecho sin que el emisor sepa quiénes son.

## 2. Componentes Clave de la Arquitectura

Para diseñar una API de eventos robusta, necesitamos tres piezas fundamentales:

### A. El Productor (Event Producer)
Es el servicio donde ocurre la acción. Su única responsabilidad es detectar un cambio de estado en su dominio y publicar un mensaje en un bus. Siguiendo lo que vimos en DDD, el productor suele disparar un **Domain Event** desde su Aggregate Root.

### B. El Broker de Mensajería (Event Broker)
Es el cartero del sistema. Recibe los eventos y los distribuye.
*   **Herramientas comunes:** RabbitMQ (ideal para routing complejo), Apache Kafka (ideal para alto volumen y persistencia de eventos) o Redis Pub/Sub (para velocidad extrema).

### C. El Consumidor (Event Consumer)
Son los servicios que "escuchan" temas específicos. Un evento de `OrdenCreada` puede ser consumido simultáneamente por el servicio de Facturación, el de Stock y el de Notificaciones.

## 3. Patrones de Diseño Esenciales

Diseñar una API de eventos no es solo tirar mensajes a una cola. Hay patrones que garantizan que el sistema no explote:

### Event Sourcing
En lugar de guardar solo el estado actual en la base de datos, guardamos la serie completa de eventos que llevaron a ese estado. Esto nos da una trazabilidad (audit log) perfecta por naturaleza. Es decir en lugar de "estado final" (ej: Stock = 5), guardamos cada cambio como un hecho inmutable (ej: Entrada +10, Salida -3, Salida -2).

- **Por qué importa:** El estado actual es solo un derivado. Si perdemos la base de datos, podemos reconstruirla "reproduciendo" los eventos desde el inicio.

- **Utilidad:** Ideal para sistemas financieros o de auditoría donde necesitás saber exactamente cómo y por qué se llegó a un valor, no solo cuál es el valor actual.

### CQRS (Command Query Responsibility Segregation)
Separamos la escritura de la lectura. Los eventos actualizan una "vista" optimizada para consultas, permitiendo que las lecturas sean increíblemente rápidas sin bloquear las transacciones de escritura. En arquitecturas de eventos, el modelo que escribe (Command) suele ser muy complejo. CQRS propone separar físicamente la base de datos de escritura de la de lectura (Query).

- **Mecánica:** Cuando ocurre un evento de escritura, un proceso asíncrono actualiza una "Read Model" (que puede ser un Redis o un Elasticsearch) optimizado para búsquedas rápidas.

- **Beneficio:** Podés escalar las lecturas de forma independiente a las escrituras, eliminando bloqueos en las tablas principales.

### Outbox Pattern (Crítico para la consistencia)
¿Qué pasa si guardás en la base de datos pero falla el envío al Broker? El **Outbox Pattern** asegura la atomicidad: guardás el evento en una tabla de "salida" dentro de la misma transacción de tu base de datos y un proceso separado lo envía al Broker. Este patrón resuelve el problema del "Distributed Transaction". Si guardás el pedido en tu DB pero el Broker de mensajes (Kafka/Rabbit) está caído, los sistemas quedan inconsistentes.

- **La Solución:** Guardás el pedido y el mensaje del evento en una tabla llamada Outbox dentro de la misma transacción local.

- **Relay:** Un proceso independiente (como un Debezium o un worker simple) lee esa tabla y publica los mensajes. Si el envío falla, lo reintenta hasta que el Broker responda. No hay pérdida de datos.

### Idempotency (Idempotencia)
En el mundo de los eventos, la red falla y los mensajes se reintentan. Esto significa que un consumidor puede recibir el mismo evento dos veces.

- **El Riesgo:** Si el evento es ProcesarPago, ¡podrías cobrarle dos veces al usuario!

- **La Solución:** Cada evento debe tener un Unique ID. El consumidor guarda los IDs ya procesados y, si recibe uno repetido, simplemente lo ignora. Si no es idempotente, no es seguro para producción.

### Saga Pattern (Transacciones Distribuidas)
Como no tenemos transacciones BEGIN/COMMIT que abarquen múltiples microservicios, usamos Sagas. Es una secuencia de transacciones locales.

- **Coreografía:** Cada servicio realiza su parte y dispara un evento. El siguiente servicio lo escucha y hace lo suyo.

- **Compensación:** Si el paso 3 falla (ej: no hay stock), se dispara un evento de "compensación" (ej: ReembolsarPago) para que los servicios anteriores vuelvan al estado original.

### Dead Letter Queue (DLQ)
¿Qué pasa si un evento llega corrupto o hace que tu consumidor explote? No podés bloquear la cola para siempre.

- **Mecánica:** Después de N reintentos fallidos, el mensaje se mueve a una cola especial llamada DLQ.

- **Utilidad:** Esto permite que el flujo principal siga funcionando mientras vos, como desarrollador, podés inspeccionar manualmente qué pasó con ese mensaje fallido sin afectar al resto del sistema.

## 4. El Contrato de Eventos: Event Schema

Al igual que una API REST tiene Swagger, una API de eventos necesita contratos claros. No podés cambiar la estructura de un mensaje sin romper a todos los consumidores.
*   **Herramientas:** AsyncAPI es el estándar para documentar estos contratos.
*   **Formatos:** JSON es común, pero en sistemas de alto rendimiento se prefiere **Protocol Buffers (Protobuf)** o **Avro** por su tipado fuerte y tamaño reducido.

## 5. Ventajas y Trade-offs

No hay soluciones mágicas.

*   **Ventajas:**
    *   **Escalabilidad:** Los consumidores pueden procesar a su propio ritmo.
    *   **Resiliencia:** Si el consumidor de correos cae, los eventos quedan en la cola y se procesan cuando vuelve a estar online.
    *   **Desacoplamiento:** El servicio de ventas no sabe que existe un servicio de marketing.

*   **Desafíos:**
    *   **Consistencia Eventual:** Los datos pueden tardar milisegundos (o segundos) en propagarse.
    *   **Complejidad de Debugging:** Seguir el rastro de un error a través de múltiples colas es más difícil que en una traza monolítica.

## 6. Caso

1. **El Problema:** El Efecto Dominó (Acoplamiento)
En una arquitectura tradicional (REST/Síncrona), cuando un usuario compra, el Servicio de Pedidos tiene que avisarle a todos:

- Llamar al Servicio de Pago.
- Esperar.
- Llamar al Servicio de Stock.
- Esperar.
- Llamar al Servicio de Envío.
- Si el Servicio de Envío está caído, ¡toda la compra falla! El sistema está "atado".

2. **La Solución:** El Sistema de "Gritos" (Eventos)

En EDA, el Servicio de Pedidos no da órdenes; simplemente anuncia un hecho.

- Hecho: "¡Se ha creado el pedido #505!"

El Servicio de Pedidos publica ese mensaje en un Event Broker (como Kafka o RabbitMQ) y se olvida. No le importa quién lo escucha.

3. **El Flujo de un E-commerce Paso a Paso**

Diseñaría el sistema para que funcione de la siguiente manera:

A. **La Chispa:** El Evento de Dominio

El usuario hace clic en "Comprar". El Servicio de Pedidos guarda el pedido en su base de datos y dispara un evento: OrderCreated.

B. **Reacciones en Cadena** (Consumidores)

Aquí es donde ocurre la magia. Varios servicios están "suscritos" al tema de pedidos y reaccionan al mismo tiempo:

- **Servicio de Pagos:** Escucha OrderCreated, procesa la tarjeta y dispara PaymentSucceeded.

- **Servicio de Inventario:** Escucha OrderCreated y reserva el stock temporalmente.

C. **La Orquestación** (Sagas)

¿Qué pasa si el pago se confirma?

- El Servicio de Almacén escucha PaymentSucceeded y prepara el paquete.

- El Servicio de Notificaciones escucha PaymentSucceeded y envía el mail de "Gracias por tu compra".

D. **Gestión de Fallos** (Compensación)

Si el pago falla (PaymentFailed), el Servicio de Inventario escucha ese evento y libera el stock automáticamente. Nadie tuvo que llamar al servicio de inventario para decirle "che, cancelá"; él solo reaccionó al fracaso del pago.

4. **Por qué esta arquitectura**

Algunos puntos clave que garantizan la robustez:

- **Escalabilidad Diferenciada:** Si el Black Friday hace que el Servicio de Notificaciones se sature y tarde en enviar los mails, no importa. Los pedidos se siguen procesando y los mails se enviarán en cuanto el servicio se descongestione. La cola guarda el trabajo pendiente.

- **Nuevas Funcionalidades sin tocar el Core:** Si mañana querés agregar un Servicio de Inteligencia Artificial para recomendar productos basados en compras, solo tenés que "enchufarlo" al Broker para que escuche OrderCreated. No tenés que modificar ni una sola línea de código del Servicio de Pedidos original.

- **Resiliencia:** Si el servicio de envíos cae por 10 minutos, los eventos se quedan en el Broker. Cuando el servicio vuelve, retoma exactamente donde dejó. No se perdió ninguna venta.

**Resumen:**
Diseñamos una arquitectura donde los servicios son autónomos. El Pedido no sabe que existe el Envío; solo sabe que su trabajo terminó cuando publicó el evento. Esto nos da un sistema desacoplado, fácil de testear y, sobre todo, preparado para crecer sin dolor.

## 7. Diagrama de Arquitectura: Flujo de un Pedido (EDA)

```
[ CLIENTE ] 
    |
    | (1) POST /orders
    v
+-----------------------+       (2) Transacción Atómica        +-----------------------+
|  SERVICIO DE PEDIDOS  |-------------------------------------->|   BASE DE DATOS       |
| (Aggregate Root)      |       [ Pedido + Outbox Table ]       | (PostgreSQL/MongoDB)  |
+-----------------------+                                       +-----------------------+
    |                                                                      |
    |                                                                      | (3) CDC / Relay
    v                                                                      v
+---------------------------------------------------------------------------------------+
|                                  EVENT BROKER (Kafka / RabbitMQ)                      |
|                           Topic: order.events | Topic: payment.events                 |
+---------------------------------------------------------------------------------------+
    |                               |                               |
    | (4) Evento: OrderCreated      | (4) Evento: OrderCreated      | (4) Evento: OrderCreated
    v                               v                               v
+-----------------------+       +-----------------------+       +-----------------------+
|  SERVICIO DE PAGOS    |       | SERVICIO DE STOCK     |       | SERVICIO DE ENVÍOS    |
| (Procesa Transacción) |       | (Reserva Artículos)   |       | (Prepara Logística)   |
+-----------------------+       +-----------------------+       +-----------------------+
    |                               |                               |
    | (5) Evento: PaymentDone       | (5) Evento: StockReserved     | (5) Espera Pago...
    v                               v                               v
+---------------------------------------------------------------------------------------+
|                                  EVENT BROKER (Bus de Retorno)                        |
+---------------------------------------------------------------------------------------+
    |
    | (6) Reacción: Notificar Usuario / Iniciar Despacho
    v
+-----------------------+
| SERVICIO NOTIFICACIÓN |
| (Envía Email/Push)    |
+-----------------------+
```

### Explicación del diagrama

- **La Chispa (Request):** El cliente inicia la compra. El Servicio de Pedidos actúa como el Aggregate Root.

- **Garantía de Consistencia (Outbox Pattern):** El pedido se guarda en la base de datos junto con el evento a disparar en una sola transacción. Esto evita que el pedido se cree pero el mensaje nunca se envíe.

- **El Mensajero (Relay/CDC):** Un proceso (como Debezium) lee la tabla Outbox y publica el evento OrderCreated en el Event Broker.

- **Paralelismo Real:** Los servicios de Pagos, Stock y Envíos reciben el mensaje al mismo tiempo. No hay esperas. Si el Servicio de Envíos está lento, los demás siguen procesando.

- **Coreografía (Sagas):** Cada servicio realiza su tarea local y emite su propio evento. Por ejemplo, el Servicio de Pagos emite PaymentDone.

- **Desacoplamiento Final:** El Servicio de Notificaciones no sabe nada de pagos o stock; solo escucha el evento de éxito para enviarle el mail al usuario.

¿Por qué este diagrama?

- **Muestra el "Outbox Pattern":** Demuestra cómo evitar la pérdida de datos en sistemas distribuidos.

- **Diferencia el Dominio de la Infraestructura:** Separa claramente dónde vive la lógica y dónde el transporte de mensajes.

- **Maneja la Idempotencia:** Implica que cada servicio receptor debe ser capaz de procesar el evento sin duplicar acciones si el mensaje llega dos veces.

### Otros terminos del diagrama

A. **Ubicación del Aggregate Root**

1. **El Aggregate Root** (Capa de Dominio)

El Aggregate Root (ej: la clase Order) vive en el corazón del hexágono (Core/Domain/Entity).

- **Su función:** No es disparar información a la base de datos, sino validar el negocio.

- **El Evento:** Dentro de la clase Order, cuando algo importante pasa (como confirmar()), la entidad agrega un evento a una lista interna (ej: $this->events[] = new OrderCreated($this->id)). La entidad no envía nada, solo "anota" que algo pasó.

2. **El Repositorio** (Capa de Infraestructura)

Aquí es donde ocurre la magia de la persistencia. El Caso de Uso (Aplicación) llama al Repositorio (Infrastructure/Persistence/SqlOrderRepository) para guardar la entidad.

Es en este método save(Order $order) donde implementás el patrón Outbox:

- **Mapeo:** Convertís tu Entidad de Dominio a tu modelo de Base de Datos.

- **Transacción:** Abrís una transacción de base de datos única.

- **Persistencia Doble:**

    - Insertás el registro en la tabla orders.

    - Insertás los eventos que traía la entidad en la tabla outbox.

    - Commit: Si ambas inserciones funcionan, se cierra la transacción.

**Estructura de Carpetas**

- src/Core/Domain/Order/Order.php -> Aggregate Root (Crea los eventos).

- src/Core/Domain/Order/OrderCreated.php -> Evento de Dominio.

- src/Infrastructure/Persistence/EloquentOrderRepository.php -> Repositorio (Guarda en tabla Order + tabla Outbox).

- scripts/relay-worker.go (o una instancia de Debezium) -> CDC/Relay (Lee la tabla y publica al Broker).

**¿Ves cómo se conecta todo?**

El dominio genera el hecho, la infraestructura lo asegura en disco (Outbox) y el relay lo comunica al mundo. De esta forma, tu código de negocio sigue siendo "puro" y no sabe nada de brokers o colas.

B. **¿Qué es un Topic (Tópico)?**

Si el Event Broker es el edificio de correos, los Topics son las diferentes categorías o etiquetas que le ponemos a los mensajes para que lleguen a la persona adecuada.

Imagina que el Broker es como una radio con muchas frecuencias:

- **Frecuencia A:** Solo pasa noticias de "Pedidos".

- **Frecuencia B:** Solo pasa noticias de "Pagos".

- **Frecuencia C:** Solo pasa noticias de "Stock".

Los servicios no se hablan entre ellos directamente; simplemente se "sintonizan" en el tópico que les interesa.

**¿Cómo funciona en la práctica?**

- **Publicar:** El servicio de pedidos grita en la frecuencia de "Pedidos": "¡Se creó la orden #505!".

- **Suscripción:** Cualquier otro servicio (Pagos, Envío, Notificaciones) que esté escuchando esa frecuencia recibirá el mensaje automáticamente.

- **La clave es el desacoplamiento:** El que publica el mensaje no necesita saber cuánta gente lo está escuchando, ni quiénes son. Solo lo deja en el tópico correspondiente y sigue con su trabajo.

**¿Por qué es útil para tu API?**

Sin tópicos, tendrías una sola "bolsa" de mensajes mezclados donde todos tendrían que leer todo para encontrar lo que les sirve, lo cual sería un caos. Los tópicos permiten que tu arquitectura sea ordenada y eficiente, enviando la información solo a quien realmente la necesita.

C. **¿Qué es el CDC / Relay?**

Es un proceso independiente que "observa" tu base de datos y se encarga de mover los datos de la tabla local al sistema de eventos. Hay dos formas principales de implementarlo:

- 1. **Transactional Relay** (El "Worker" simple)

Es un servicio (puede ser un pequeño proceso en Go o un Job de Laravel) que consulta la tabla Outbox cada N milisegundos.

**Flujo:** Busca registros con estado PENDING, los publica en el Topic correspondiente y luego los marca como PROCESSED o los borra.

**Pros:** Muy fácil de implementar.

**Contras:** Genera una carga extra de lectura en la base de datos (polling).

2. **CDC (Change Data Capture)** - El estándar Senior

Es una técnica más avanzada que no "pregunta" a la tabla, sino que escucha los logs de transacciones de la base de datos (como el Binlog en MySQL o el WAL en PostgreSQL).

**Herramienta líder:** Debezium.

**Flujo:** Debezium detecta que se insertó una fila en la tabla Outbox e inmediatamente la transforma en un evento de Kafka o RabbitMQ.

**Pros:** Es casi en tiempo real, no sobrecarga la base de datos con consultas y es extremadamente confiable.

**¿Por qué no enviar el evento directamente desde el código?**

Esta es la pregunta que te harían en una arquitectura de alto nivel. Si envías el mensaje directamente desde tu código (publishEvent()) después de guardar en la DB, te expones a este fallo:

- Guardás el Pedido en la DB (Éxito).

- Se corta la luz o falla la red justo antes de enviar al Broker.

- Resultado: Tenés un pedido cobrado pero el depósito nunca se enteró porque el evento jamás salió.

Con el CDC / Relay + Outbox Table, esto es imposible. Si la transacción de la DB se completó, el evento ya está escrito en el disco y el Relay lo enviará tarde o temprano, apenas el sistema se recupere.

**En resumen**

El CDC / Relay es el garante de la entrega. Asegura que la comunicación entre servicios no dependa de la suerte o de que la red funcione perfecto en ese microsegundo, sino de una persistencia sólida que sobrevive a caídas del sistema.

D. **La función del "Bus de Retorno"**

Imagina que el Servicio de Pedidos publica un evento de OrderCreated. El Servicio de Pagos lo escucha, procesa la tarjeta y obtiene un resultado. ¿Cómo se entera el Servicio de Pedidos de que el pago fue exitoso?

El Bus de Retorno es el canal (tópico o cola) donde los servicios secundarios publican los resultados de sus tareas para que el orquestador (o el servicio interesado) pueda cerrar el ciclo del negocio.

**Por qué se usa (El patrón de Respuesta Asíncrona)**

En el mundo síncrono (REST), hacés una petición y esperás la respuesta en la misma conexión. En eventos, la conexión se cierra apenas publicás el mensaje. El "Bus de Retorno" permite:

- **Notificar Éxito/Fracaso:** El servicio de pagos avisa si el cobro salió bien o mal.

- **Actualizar el Estado:** El Servicio de Pedidos escucha ese bus para cambiar el estado de la orden de PENDING a PAID.

- **Continuar la Saga:** Es la señal para que el siguiente paso de la coreografía comience (por ejemplo, avisar al depósito para que prepare el paquete).

**Implementación Técnica**

Físicamente, es el mismo Kafka, RabbitMQ o Redis que ya estás usando, pero organizado de forma lógica:

- **Tópicos de Salida (Forward):** Donde se publican las intenciones de acción (ej: order.created, shipment.requested).

- **Tópicos de Retorno (Reply):** Donde se publican los resultados (ej: payment.confirmed, inventory.allocated).

**Ejemplo del diagrama de E-commerce**

En el esquema que armamos, el Bus de Retorno es el que recibe el mensaje (5) Evento: PaymentDone.

- El Servicio de Pagos termina su laburo.

- Publica en el "Bus de Retorno" (Tópico de eventos de pago).

- El Servicio de Pedidos (o el de Notificaciones) está escuchando ese canal específico para saber que ya puede mandarle el mail al usuario diciendo: "¡Pago recibido!".

A veces, para evitar que todos los servicios escuchen todo, se implementa lo que se conoce como "Reply-To Header". El mensaje original lleva una etiqueta que dice: "Cuando termines, respondeme en este canal específico". Esto es muy común cuando usás RabbitMQ para simular un comportamiento de "Pedido-Respuesta" sobre eventos.

E. **Diferencia entre ese "Job" y la herramienta de "CDC"**

1. **El Worker como un "Job"** (Polling)

Si lo hacés de forma manual (como un proceso en Go o un Job de Laravel/Node), es efectivamente un loop infinito.

- **Cómo funciona:** El worker hace un SELECT * FROM outbox WHERE status = 'pending' LIMIT 100.

- **Configuración:** En tu código, vos decidís qué enviar. Tomas la fila, extraes el JSON del evento y lo publicas al tópico.

- **Punto débil:** Si el worker consulta cada 1 segundo, tenés 1 segundo de latencia. Si lo hacés cada 10ms, podés estresar la base de datos con consultas vacías.

2. **El CDC** (Debezium / Herramientas de Log)

Aquí no hay un "SELECT". La herramienta se conecta a la base de datos como si fuera un nodo de réplica más.

- **Cómo funciona:** Mira el Transaction Log (WAL en Postgres o Binlog en MySQL). Cada vez que la base de datos escribe en el disco, el CDC lo detecta al instante.

- **La configuración que mencionás:** ¡Es tal cual lo sospechás! No se manda todo el log de la base de datos (que sería un caos de tráfico).

**¿Cómo se configura el CDC para no enviar "basura"?**

Herramientas como Debezium se configuran mediante filtros. Vos le decís:

- **Whitelisting de Tablas:** "Solo mirá la tabla outbox de la base de datos ecommerce_db". Ignora todo lo demás (usuarios, productos, etc.).

- **Filtro de Operaciones:** "Solo avisame cuando haya un INSERT". No me interesan los UPDATE o DELETE sobre esa tabla.

- **Transformaciones (SMT):** Podés configurar una "Transformación Simple de Mensaje". Por ejemplo, que el mensaje que llegue al Broker solo contenga el campo payload de la tabla, quitando metadatos de la base de datos como el ID de la fila o el timestamp de inserción.

**¿Cuál elegir?**

**El Worker/Job:** Es la "vieja confiable". Fácil de debugear, control total del código, pero menos eficiente a gran escala.

**El CDC (Debezium):** Es la solución de "clase mundial". Es la que usan empresas con millones de eventos porque es casi en tiempo real y no afecta el rendimiento de la base de datos con consultas constantes.

En el mundo real, muchas veces empezamos con un Worker simple en Go (que es súper liviano) y, cuando el sistema crece y la latencia de 1 segundo molesta, migramos a Debezium.

## Conclusión

Migrar hacia una arquitectura orientada a eventos es un paso natural cuando tu sistema crece y el acoplamiento empieza a doler. Es el complemento externo ideal para el diseño interno que logramos con Clean Architecture.

Si lográs que tus servicios se comuniquen mediante hechos (eventos) en lugar de órdenes (comandos directos), habrás alcanzado un nivel de madurez técnica que permite que tu arquitectura evolucione casi sin fricción.