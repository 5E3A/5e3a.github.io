+++
date = '2026-03-22T14:04:18-03:00'
draft = false
title = 'Estrategias de Despliegue (o Deployment Strategies)'
tags = [
  "deployment", 
  "CI-CD", 
  "devops", 
  "infrastructure", 
  "sre", 
  "cloud-computing", 
  "scalability", 
  "zero-downtime"
]
categories = ["Architecture", "DevOps"]
+++
Hacer Deploy no es solo subir archivos por FTP y rezar para que nada se rompa. En entornos profesionales, el despliegue es un arte que busca dos cosas: cero tiempo de inactividad (Zero Downtime) y el mínimo riesgo posible.

### I. Estrategias de Gestión de Tráfico (El "Cómo" llega el usuario)
Aquí agrupamos las técnicas que deciden qué versión del software ve el usuario final.

#### 1. Blue-Green Deployment: El espejo perfecto
Esta estrategia consiste en tener dos entornos idénticos operando al mismo tiempo:

- **Blue (Azul):** Es la versión que está actualmente en producción recibiendo tráfico.
- **Green (Verde):** Es la nueva versión que acabas de subir y estás probando internamente.

Cuando estás seguro de que la versión "Green" funciona perfecto, simplemente cambias el "enrutador" para que todo el tráfico vaya a Green. Si algo falla, vuelves a Blue en un segundo. Es el "rollback" más rápido que existe.

#### 2. Canary Deployment: La prueba de fuego
El nombre viene de los mineros que usaban canarios para detectar gases tóxicos. En software, consiste en lanzar la nueva versión solo para un pequeño porcentaje de usuarios (por ejemplo, el 5%).

Si esos usuarios no reportan errores y los logs se ven bien, vas aumentando el tráfico gradualmente (10%, 25%, 50%...) hasta llegar al 100%. Es ideal para detectar errores que solo ocurren con tráfico real sin afectar a toda tu base de usuarios.

#### 3. Canary Analysis (Automated)
Para empresas de escala masiva, no basta con mirar a ojo el Canary. Se usan algoritmos que comparan las métricas de la versión estable vs. la versión Canary. Si la latencia aumenta un 5% o hay un leve incremento en errores de base de datos, el sistema aborta el deploy solo. Es como tener un robot vigilando el canario las 24 horas.


#### 4. Shadow Deployment: El despliegue fantasma
Consiste en enviar el tráfico real a la nueva versión, pero sin que el usuario vea la respuesta de esta. Básicamente, la versión nueva procesa los datos "en las sombras" para ver cómo se comporta con carga real, pero el usuario sigue recibiendo la respuesta de la versión vieja.

#### 5. A/B Testing vs. Canary
A veces se confunden, pero tienen objetivos distintos:

- **Canary:** Es técnico. Buscamos saber si la App se rompe.
- **A/B Testing:** Es funcional/marketing. Queremos saber si a los usuarios les gusta más el botón verde o el azul.

### II. Mecanismos de Infraestructura (El "Cómo" se mueven los servidores)
Cómo manipulamos los nodos y contenedores durante el proceso.

#### 1. Rolling Update (Despliegue Progresivo)
Es la estrategia por defecto en sistemas como Kubernetes. En lugar de cambiar todo a la vez, se van actualizando las instancias (servidores o nodos) de una en una.

Si tienes 4 servidores, el sistema actualiza el #1. Cuando está listo, pasa al #2, y así sucesivamente. Nunca dejas de tener servidores activos, por lo que el usuario no siente la interrupción.

#### 2. Recreación (Recreate Strategy)
Es la más sencilla pero la más "peligrosa": matas todos los servidores viejos y levantas los nuevos.

- **Ventaja:** No hay problemas de compatibilidad porque nunca conviven las dos versiones.
- **Desventaja:** Tienes Downtime (el sitio queda caído unos segundos o minutos). Solo se usa en entornos de desarrollo o aplicaciones internas donde no importa que se corte el servicio.

#### 3. Infraestructura Inmutable: "No repares, reemplaza"
En el pasado, si un servidor fallaba, entrabas por SSH y lo arreglabas. Hoy usamos Infraestructura Inmutable. Si necesitas actualizar algo, no modificas el servidor actual; creas uno nuevo desde una imagen (Docker/AMI) y destruyes el viejo.

Esto garantiza que tu entorno de producción sea idéntico al de prueba y evita el famoso "en mi máquina funcionaba".

#### 4. Atomic Deploys (Despliegues Atómicos)
Imagina que estás subiendo archivos por FTP y la conexión se corta a la mitad. Tu sitio quedaría con archivos viejos y archivos nuevos mezclados: un desastre total.

Los Deploys Atómicos funcionan creando una carpeta nueva para cada versión. Solo cuando todos los archivos están arriba, se cambia un "enlace simbólico" (Symlink) para que apunte a la carpeta nueva. El cambio ocurre en microsegundos: o sube todo, o no sube nada.

#### 5. GitOps: Tu repositorio es la única verdad
GitOps es una evolución donde el estado de tu infraestructura (servidores, redes, bases de datos) está definido en un archivo dentro de Git.

Si quieres escalar de 2 a 5 servidores, no entras a un panel; cambias un número en un archivo YAML y haces push.
El sistema detecta el cambio y automáticamente ajusta la realidad al código. Esto hace que los deploys sean 100% auditables y fáciles de replicar.

### III. Mitigación de Riesgos y "Safe Guards" (El plan de emergencia)
Herramientas para evitar que un error llegue a mayores.

#### 1. Feature Flags: El interruptor maestro
Esta es la técnica favorita de empresas como Facebook o Netflix. Consiste en subir el código de una funcionalidad nueva pero mantenerla "apagada" mediante un condicional en el código (un if).

- Te permite hacer deploy un viernes (si eres valiente) y activar la función el lunes sin subir código nuevo.
- Si la función falla, la apagas instantáneamente desde un panel de control sin necesidad de hacer un rollback completo.

#### 2. Rollbacks: El plan de evacuación
Un deploy profesional no está completo sin un plan de Rollback automatizado. No es "borrar y volver a subir"; es tener una versión anterior lista para ser reactivada en milisegundos.

En estrategias Canary, el rollback puede ser automático: si la tasa de errores (500) sube del 1%, el sistema corta el despliegue y vuelve a la versión estable sin intervención humana.

#### 3. Health Checks: El latido del sistema
Ninguna estrategia de deploy funciona sin un Health Check. Es un endpoint (ej: /api/health) que el balanceador de carga consulta constantemente.

Si durante un Rolling Update el nuevo servidor levanta pero la base de datos no conecta, el Health Check fallará y el sistema detendrá el deploy automáticamente antes de afectar a los usuarios.

#### 4. El desafío de la Base de Datos: Migraciones
Cambiar el código es fácil, pero cambiar la estructura de una base de datos con millones de registros es peligroso. En despliegues como Blue-Green, la base de datos suele ser compartida.

**Regla de oro:** Los cambios en la base de datos siempre deben ser compatibles hacia atrás. Primero agregas la columna nueva, y solo cuando el código nuevo es estable, borras la vieja.

#### 5. Dark Launches: Probando en producción sin que se note
Similar al Canary, pero más sutil. El Dark Launching consiste en desplegar una funcionalidad y hacer que el Frontend realice llamadas a esa nueva API por debajo, pero sin mostrar los resultados al usuario final.

Sirve para estresar el servidor y ver si la base de datos aguanta la nueva carga antes de "abrir el grifo" oficialmente.

### IV. Confiabilidad y Cultura Post-Deploy (El mantenimiento del modelo)
Lo que pasa una vez que el código ya está "afuera".

#### 1. Monitoreo Post-Deploy
El deploy no termina cuando el código llega al servidor. Necesitas herramientas de monitoreo (como Grafana, New Relic o Datadog) para observar en tiempo real:

- Uso de CPU y Memoria.
- Latencia de las respuestas.
- Porcentaje de errores HTTP.

**Consejo:** "Deploy early, deploy often". Cuanto más pequeños sean tus despliegues, menos riesgo tienen. Es preferible hacer 10 deploys pequeños al día que uno gigante una vez al mes.

#### 2. Chaos Engineering: Probar el deploy rompiendo cosas
Empresas como Netflix usan lo que llaman el Chaos Monkey. Es un software que, de forma aleatoria, "mata" servidores o tira abajo bases de datos en producción justo después de un deploy.

¿Para qué? Para asegurar que sus estrategias de Rollback y Health Checks funcionen de verdad. Si tu sistema puede sobrevivir a un ataque aleatorio mientras estás desplegando, entonces tu estrategia de deploy es verdaderamente irrompible.

#### 3. El factor humano: "Post-Mortems"
Incluso con la mejor estrategia, algo puede fallar. Las empresas de alto rendimiento practican los Blameless Post-Mortems: reuniones donde se analiza qué falló en el deploy para mejorar la automatización, sin buscar culpables, sino buscando debilidades en el proceso.

**El peligro del "Configuration Drift"**

Ocurre cuando alguien cambia algo "a mano" en el servidor de producción y se olvida de replicarlo en el código de infraestructura. En el próximo deploy, ese cambio desaparecerá y todo se romperá. Por eso, todo cambio debe pasar por el pipeline de CI/CD.

### Conclusión: ¿Qué estrategia elegir?

No existe una “bala de plata”; la elección depende de tu arquitectura, tu presupuesto y tu tolerancia al riesgo:

* **Blue-Green:** Si tenés presupuesto para duplicar infraestructura y necesitás un rollback instantáneo.
* **Canary:** Si tenés una gran base de usuarios y querés validar estabilidad con riesgo controlado.
* **Rolling Update:** Si buscás eficiencia de recursos y estabilidad constante sin duplicar servidores.
* **Shadow / Dark Launch:** Si el sistema es crítico y necesitás probar performance con carga real.

### Conceptos Clave: La base de todo es CI/CD

Para que estas estrategias funcionen sin errores humanos, necesitamos **CI/CD (Continuous Integration / Continuous Deployment)**. No son solo siglas; es la cultura de automatizar el ciclo de vida del código:

1.  **CI (Integración Continua):** Cada vez que hacés un `git push`, un robot (runner) testea y buildea tu código.
2.  **CD (Despliegue Continuo):** El paso final donde el sistema ejecuta automáticamente la estrategia elegida.

Herramientas como **GitHub Actions**, **GitLab CI** o **Jenkins** son los motores que permiten que el despliegue sea un proceso de un solo clic.

### Conexión con la Disponibilidad

Como analicé en mi post sobre [Uptime Modeling](/posts/git/uptime-modeling-guia-sre), estas técnicas son las herramientas que reducen el **MTTR** (Mean Time To Recovery) y protegen nuestro **Error Budget**:

- * Un **Rollback automático** en un Canary Deployment salva tu SLA en segundos.
- * Un **Feature Flag** permite apagar una función rota sin necesidad de un nuevo despliegue.

**En resumen:** El despliegue moderno se trata de gestionar el riesgo para que los **“Nueves”** de nuestra infraestructura nunca dejen de brillar.