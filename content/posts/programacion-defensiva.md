+++
date = '2026-03-19T21:40:15-03:00'
draft = false
title = 'Programación defensiva'
tags = ['Backend', 'Golang', 'Programming Patterns', 'Reliability', 'Software Architecture']
+++
En un mundo ideal, cada función que llamamos responde instantáneamente. En el mundo real, los servidores se caen, las redes se saturan y los procesos se bloquean. Sin patrones defensivos, una sola llamada lenta puede generar un efecto dominó que tumbe todo tu sistema. Aquí es donde entra el Timeout.

### 1. ¿Qué es la Programación Defensiva?
Es un enfoque de diseño donde el código se escribe bajo la premisa de que "si algo puede salir mal, saldrá mal". No se trata de confiar en que los datos de entrada serán perfectos o que los servicios externos siempre estarán disponibles, sino de construir "escudos" alrededor de nuestra lógica de negocio.

### 2. El concepto de Timeout: El límite de la paciencia
Un Timeout es un patrón que establece un tiempo máximo de espera para una operación. Si la operación no termina en ese plazo, el programa interrumpe la ejecución y recupera el control.

¿Por qué es vital? Sin un timeout, un hilo (thread) o goroutine puede quedarse esperando una respuesta para siempre (Hanging). Si tienes 100 usuarios esperando y 100 hilos bloqueados, tu servidor se queda sin recursos y deja de responder a todos los demás.

### 3. Tipos de Timeouts en la Industria
Connection Timeout: El tiempo máximo para establecer el "apretón de manos" (handshake) inicial con un servidor.
Read/Request Timeout: Una vez conectados, es el tiempo máximo que esperamos por la respuesta completa.
Idle Timeout: Tiempo máximo que una conexión puede estar abierta sin transmitir datos.

### 4. Patrones Defensivos Relacionados

#### A. Fail-Fast (Fallo Rápido)
Es mejor devolver un error en 200ms que una respuesta correcta en 30 segundos. El timeout permite que el sistema falle rápido, liberando recursos y permitiendo que el usuario o el proceso de rellamada (retry) decida qué hacer.

#### B. Graceful Degradation (Degradación Elegante)
Si una llamada con timeout falla, el programa no tiene por qué morir. Puede devolver un valor por defecto, usar una caché antigua o mostrar un mensaje amigable.
Ejemplo: Si el servicio de "Recomendaciones" falla por timeout, el sitio de e-commerce muestra "Productos Populares" en su lugar.

### 5. Implementación Técnica (Ejemplo en Go)
En Go, el estándar para manejar la programación defensiva y los timeouts es el paquete context.

```golang
func peticionSegura() {
  // Definimos un escudo: solo esperamos 2 segundos
  ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
  defer cancel()

  req, _ := http.NewRequestWithContext(
    ctx, "GET", "https://api.lenta4321.com", nil)

  client := &http.Client{}

  resp, err := client.Do(req)
  if err != nil {
      if ctx.Err() == context.DeadlineExceeded {
          fmt.Println("Escudo activado: La API tardó demasiado.")
      } else {
          fmt.Println("Error de conexión:", err)
      }
      return
  }
  defer resp.Body.Close()
}
```