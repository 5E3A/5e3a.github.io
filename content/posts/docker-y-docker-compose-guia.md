---
title: "Docker y Docker Compose: Guía"
date : '2026-05-01T15:40:16-03:00'
tags: ["Docker", "DevOps", "Containers", "Infrastructure"]
categories: ["Technical Guide"]
author: "Sebastian"
description: "De 'en mi máquina funciona' al despliegue profesional: todo lo que necesitás saber sobre Docker."
---

Seguramente escuchaste la frase: *"¡Pero en mi máquina funciona!"*. Docker nació para terminar con ese problema. Como **Developer**, el uso de contenedores no es opcional; es la base de cualquier entorno de desarrollo moderno y seguro.

En este post, vamos a desglosar desde la teoría básica hasta el uso de Docker Compose para orquestar servicios complejos.

## 1. ¿Qué es Docker? (Más allá de la superficie)
Para entender Docker, primero hay que entender el problema que resuelve. Tradicionalmente, si querías aislar una aplicación, usabas una Máquina Virtual (VM). El problema es que cada VM necesita su propio Sistema Operativo completo, lo que consume gigas de RAM y minutos en arrancar.

**Docker cambia las reglas del juego:** En lugar de virtualizar el hardware, Docker virtualiza el sistema operativo. Utiliza características del Kernel de Linux (como namespaces y control groups) para crear "burbujas" aisladas llamadas contenedores.

**Conceptos Clave (El ecosistema)**

- **Imagen (La Plantilla):** Es un paquete de solo lectura que contiene el código, las librerías, las variables de entorno y los archivos de configuración. Pensalo como un Snapshop o un "molde" que nunca cambia.

- **Contenedor (La Instancia):** Es la imagen puesta en marcha. Si la imagen es la clase en POO, el contenedor es el objeto instanciado. Es efímero: podés borrarlo y recrearlo en segundos sin afectar la imagen.

- **Dockerfile (El Manifiesto):** Es un archivo de texto con las instrucciones secuenciales para construir tu imagen. Es la "documentación como código" de tu infraestructura.

- **Docker Registry (El Almacén):** Es donde guardás y compartís tus imágenes. El más famoso es Docker Hub, pero las empresas suelen tener registros privados por seguridad.

**¿Por qué es tan rápido?**

A diferencia de las VMs, cuando lanzás un contenedor, no estás arrancando un sistema operativo desde cero. Estás lanzando un proceso dentro de tu propio sistema operativo, pero que cree que está solo en el mundo. Por eso un contenedor arranca en milisegundos.

## 2. Docker Compose: Orquestación y Ecosistemas

Si un contenedor es una pieza de Lego, Docker Compose es el manual de instrucciones que te dice cómo encajarlas todas para construir un castillo. En proyectos reales, raramente trabajamos con un solo proceso; solemos tener un ecosistema: la API en Go, una base de datos PostgreSQL, un sistema de caché como Redis y quizás un Worker para procesos en segundo plano.

**¿Qué problemas resuelve realmente?**

- **Infraestructura como Código (IaC):** En lugar de recordar una lista interminable de comandos docker run con 20 parámetros cada uno, definís todo en un archivo docker-compose.yml que vive en tu repositorio Git. Esto garantiza que cualquier desarrollador del equipo levante exactamente el mismo entorno que vos.

- **Redes Aisladas Automáticas:** Al levantar un archivo Compose, Docker crea una red virtual privada para esos servicios. Los contenedores pueden hablar entre ellos usando simplemente el nombre del servicio (por ejemplo, tu app se conecta a host: db) sin necesidad de conocer IPs dinámicas.

- **Gestión de Volúmenes y Persistencia:** Te permite declarar de forma centralizada dónde se guardarán los datos de tu base de datos en el host, asegurando que no se pierdan al reiniciar los servicios.

- **Control de Dependencias:** Mediante la instrucción depends_on, podés establecer un orden lógico de arranque, asegurando que la base de datos esté lista antes de que tu aplicación intente conectarse a ella.

- **Reflexión:** Docker Compose no es una herramienta de producción masiva (para eso existe Kubernetes), pero es el estándar absoluto para entornos de desarrollo, testing y CI/CD. Te permite "clonar" la arquitectura de producción en tu máquina local con un solo comando.

## 3. Guía de Comandos

Aquí tenés el "cheat sheet" que todo dev debería tener a mano, con explicaciones claras:

### Gestión de Contenedores (Diagnóstico y Control)

| Comando | Descripción |
| :--- | :--- |
| `docker ps` | Lista los contenedores activos. Agregá `-a` para ver los detenidos. |
| `docker run <imagen>` | Descarga (si no existe) y levanta un contenedor. |
| `docker stop <id/nombre>` | Detiene un contenedor de forma segura. |
| `docker rm <id/nombre>` | Elimina un contenedor (debe estar detenido). |
| `docker logs -f <id>` | Sigue los logs de un contenedor en tiempo real. |
| `docker exec -it <id> sh` | Entra a la terminal dentro de un contenedor corriendo. |
| `docker inspect <id>` | Devuelve toda la información técnica (IP, montajes, config) en formato JSON. |
| `docker stats` | Muestra un "top" en tiempo real del consumo de CPU, Memoria y Red de tus contenedores. |
| `docker cp <id>:/ruta/archivo ./` | Copia archivos desde adentro del contenedor hacia tu máquina (o viceversa). |
| `docker pause/unpause <id>` | Pausa todos los procesos del contenedor sin detenerlo por completo. |
| `docker rename <viejo> <nuevo>` | Cambia el nombre de un contenedor existente. |

### Gestión de Imágenes y Mantenimiento

| Comando | Descripción |
| :--- | :--- |
| `docker images` | Lista todas las imágenes descargadas localmente. |
| `docker build -t nombre:tag .` | Construye una imagen a partir de un Dockerfile en el directorio actual. |
| `docker rmi <id_imagen>` | Elimina una imagen local. |
| `docker pull <imagen>` | Descarga una imagen desde Docker Hub sin ejecutarla. |
| `docker system prune` | **El salvador del disco.** Elimina contenedores detenidos, redes no usadas e imágenes huérfanas. |
| `docker image prune -a` | Elimina todas las imágenes que no estén siendo usadas por ningún contenedor. |
| `docker history <imagen>` | Muestra las capas de una imagen para entender por qué pesa lo que pesa. |
| `docker tag <imagen> <nuevo_nombre>` | Crea un alias de una imagen (muy útil antes de hacer un `push`). |

### Docker Compose (El flujo cotidiano)

| Comando | Descripción |
| :--- | :--- |
| `docker-compose up -d` | Levanta todos los servicios en segundo plano (detached). |
| `docker-compose down` | Detiene y elimina los contenedores, redes e imágenes definidos en el YAML. |
| `docker-compose logs -f` | Muestra los logs de todos los servicios combinados. |
| `docker-compose restart` | Reinicia los servicios definidos. |
| `docker-compose up --build` | Fuerza la reconstrucción de las imágenes antes de levantar los servicios. |
| `docker-compose ps` | Lista solo los contenedores asociados al archivo YAML actual. |
| `docker-compose exec <servicio> <comando>` | Ejecuta un comando usando el nombre del servicio (ej: `app`) en lugar del ID. |
| `docker-compose run --rm <servicio> <comando>` | Corre un comando en un contenedor nuevo y lo elimina automáticamente al terminar. |
| `docker-compose config` | Valida y muestra la configuración final del archivo YAML. |

## 4. Ejemplo Práctico: Dockerizando una App

Supongamos que tenés una app simple. Tu `Dockerfile` básico se vería así:

```dockerfile
# Elegimos una imagen base liviana
FROM node:18-alpine

# Directorio de trabajo
WORKDIR /app

# Copiamos archivos de dependencias
COPY package*.json ./
RUN npm install

# Copiamos el resto del código
COPY . .

# Exponemos el puerto
EXPOSE 3000

CMD ["npm", "start"]
```

**Explicación línea por línea:**

- **FROM node:18-alpine:** Define la imagen base. Usamos node en su versión 18. El tag alpine indica que es una versión de Linux ultra ligera (pesa muy pocos MB), lo cual es una buena práctica para que tus imágenes no ocupen gigas.

- **WORKDIR /app:** Crea una carpeta dentro del contenedor y nos "muda" allí. A partir de aquí, todos los comandos se ejecutarán dentro de /app. Evita usar la raíz del sistema de archivos por seguridad.

- **COPY package\*.json ./:** Copiamos solo los archivos de dependencias antes que el resto del código. ¿Por qué? Docker usa un sistema de capas. Si no cambias tus dependencias, Docker usará la caché para este paso y no volverá a descargar todo, ahorrándote mucho tiempo en cada build.

- **RUN npm install:** Ejecuta el comando para instalar las librerías. Esto sucede durante la construcción de la imagen, no cuando el contenedor arranca.

- **COPY . .:** Copia todo el resto de tu proyecto al contenedor. Se hace después del install para aprovechar la caché que mencioné arriba.

- **EXPOSE 3000:** Es meramente informativo. Le dice a quien lea el archivo (o a Docker) que la aplicación adentro escucha en el puerto 3000. No abre el puerto al mundo exterior por sí solo; eso lo hace el ports en el Compose o en el run.

- **CMD ["npm", "start"]:** Este es el proceso principal. A diferencia de RUN, esto se ejecuta solo cuando el contenedor se inicia. Solo puede haber un CMD por Dockerfile.

Y tu docker-compose.yml para incluir una base de datos:

```dockerfile
services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: example_password
```

**Explicación línea por línea:**

- **services:**

Es el nodo raíz. Aquí adentro definiremos cada "pieza" de nuestra arquitectura (app, base de datos, redis, etc.).

- **app:** 

Es el nombre que le damos a nuestro servicio. Podríamos ponerle backend o web-api.

- **build: .** 

Le dice a Compose: "No busques una imagen en internet, construí una usando el Dockerfile que está en este mismo directorio (.)".

- **ports:** 

    - **- "3000:3000":** Mapeo de puertos. El formato es HOST:CONTENEDOR.

        - El primero es el puerto de tu computadora (donde vos vas a entrar en el navegador).

        - El segundo es el puerto interno del contenedor (el que definiste en el EXPOSE del Dockerfile).

- **depends_on:**

    - **- db:** Establece el orden de arranque. Le dice a Docker que no intente levantar la app hasta que el servicio db esté iniciado. Esto evita que tu app falle al intentar conectar con una base de datos que aún no existe.

- **db:** 

Nombre del segundo servicio.

- **image: postgres:15-alpine** 

A diferencia de la app, aquí no construimos nada; descargamos una imagen oficial directamente desde Docker Hub.

- **environment:** 

Aquí definimos las Variables de Entorno. En este caso, Postgres necesita obligatoriamente que le definamos una contraseña para el usuario root al arrancar.

En producción nunca deberías dejar las contraseñas en el archivo docker-compose.yml. Lo ideal es usar un archivo .env y referenciarlo. Docker Compose leerá automáticamente las variables de ese archivo, manteniendo tus credenciales fuera del control de versiones (Git).

**Alternativas para manejar secretos en Producción**

Si bien el archivo .env es práctico, tiene un riesgo: las variables de entorno son visibles para cualquier proceso dentro del contenedor y a veces quedan registradas en los logs. Aquí hay formas más seguras:

1. **Docker Secrets (El estándar en Swarm)**

Si usás Docker en modo Swarm, podés usar secrets.

- **Cómo funciona:** El secreto se guarda encriptado en el manejador de Docker y se "monta" como un archivo temporal en /run/secrets/nombre_del_secreto dentro del contenedor.

- **Ventaja:** Nunca se expone como variable de entorno, lo que evita que se filtre en logs de error.

2. **HashiCorp Vault / AWS Secrets Manager**

Para aplicaciones de gran escala, se utilizan Manejadores de Secretos Externos.

- **Cómo funciona:** Tu aplicación, al arrancar, hace una petición a una API externa (Vault, AWS, Google Cloud) usando un token de identidad y descarga las credenciales directamente en memoria.

- **Ventaja:** Permite rotar contraseñas automáticamente sin reiniciar los contenedores.

3. **Montaje de archivos de configuración (Volumes)**

Podés tener un archivo de configuración (ej: config.prod.json) fuera de Docker y montarlo como un volumen de solo lectura.

- **Cómo funciona:** En el docker-compose.yml usas: - ./config.prod.json:/app/config.json:ro.

- **Ventaja:** Separa totalmente los secretos del código y del archivo Compose.

4. **Variables de entorno del Sistema (Shell)**

Docker Compose puede leer variables que ya existen en la máquina donde se ejecuta.

- **Cómo funciona:** Si en tu servidor hacés export DB_PASSWORD=super_secreto, y en el docker-compose.yml solo ponés POSTGRES_PASSWORD: ${DB_PASSWORD}, Docker tomará el valor del sistema.

- **Ventaja:** No hay ningún archivo con claves en el disco del proyecto, ni siquiera un .env.

## 5. Una nota sobre Seguridad: ¿Docker o Podman?

La seguridad es prioritaria. Docker tradicionalmente corre como root. Si estás en un entorno donde la seguridad es crítica, Podman es una excelente alternativa "rootless" que comparte casi todos los comandos de esta guía.

**Docker Rootless: El "Workaround"**

Para que Docker corra sin root, tenés que instalar un script adicional (dockerd-rootless-setuptool.sh), configurar variables de entorno manualmente (PATH, DOCKER_HOST) y, lo más molesto, lidiar con las limitaciones:

- **Puertos privilegiados:** No podés mapear puertos menores al 1024 (como el 80 o 443) sin tocar configuraciones del kernel (sysctl).

- **Almacenamiento:** El driver de almacenamiento suele ser más lento o complejo de configurar en modo usuario.

- **Carga inicial:** El demonio de Docker (el proceso que siempre corre de fondo) sigue existiendo, solo que ahora corre bajo tu usuario.

**Podman: El enfoque "Secure by Design"**

A diferencia de Docker, Podman fue diseñado desde el día 1 para ser rootless y daemonless.

- **Sin demonio:** No hay un proceso "padre" corriendo siempre como root esperando órdenes. Cuando corrés un contenedor, Podman simplemente lanza el proceso.

- **Integración nativa:** En sistemas como Ubuntu o Fedora, Podman ya sabe cómo manejar los espacios de nombres de usuario (user namespaces) sin instalaciones raras.

- **Seguridad:** Si un atacante logra "escapar" del contenedor, solo tiene los permisos de tu usuario limitado, no los de root del servidor.

**Cuadro comparativo**

| Característica | Docker | Podman |
| :--- | :--- | :--- |
| **Arquitectura** | Basada en un **Demonio** (proceso central `dockerd` siempre activo). | **Daemonless** (cada contenedor es un proceso independiente). |
| **Seguridad (Rootless)** | Posible, pero requiere configuración manual y tiene limitaciones de red/puertos. | **Nativa**. Diseñado para correr sin privilegios desde el primer momento. |
| **Compatibilidad** | El estándar de la industria con el ecosistema más amplio. | Casi 100% compatible (podés usar el alias `alias docker=podman`). |
| **Orquestación** | Docker Compose (requiere herramienta externa o plugin). | Soporte nativo para Compose y generación de YAMLs para Kubernetes. |
| **Consumo de recursos** | El demonio consume RAM constantemente, incluso en reposo. | Solo consume recursos cuando hay un contenedor ejecutándose. |

## 6. Buenas Prácticas

- **Usá .dockerignore:** Al igual que el .gitignore, sirve para que Docker no copie carpetas pesadas o innecesarias (como node_modules, .git o archivos de log) al contenedor. Esto hace que el BUILD sea mucho más rápido.

- **Imágenes livianas (Alpine):** Siempre que puedas, usá versiones alpine de las imágenes base. Menos peso significa despliegues más rápidos y menos superficie de ataque.

- **Una sola responsabilidad:** Cada contenedor debería correr un solo proceso. No intentes meter la base de datos y la app en el mismo contenedor; para eso inventaron Docker Compose.

- **Idempotencia en los scripts:** Asegurate de que si corrés tu docker-compose up diez veces, el resultado sea siempre el mismo y no rompa nada.

- **Utilizá el "Principio de Menor Privilegio" (USER):** Por defecto, Docker corre los procesos como root dentro del contenedor. Una práctica de industria es crear un usuario sin privilegios en el Dockerfile (ej: useradd -m developer) y usar la instrucción USER developer. Así, si alguien vulnera tu app, no tiene acceso total al sistema.

- **Multi-Stage Builds (Construcciones Multi-etapa):** Esta es clave para producción. Usas una imagen pesada para compilar (con todas las herramientas de build) y luego copias solo el binario o los archivos compilados a una imagen final mucho más pequeña. **Ejemplo:** Compilas tu app de Go o React en una imagen de 1GB, pero tu imagen final de producción solo pesa 20MB.

- **No guardes datos persistentes dentro del contenedor:** Los contenedores son efímeros (pueden morir y ser recreados en cualquier momento). Cualquier archivo que tu app genere y deba persistir (fotos, logs, bases de datos) debe ir en un Volume o un Bind Mount. Si no lo hacés, cuando borres el contenedor, perdés los datos.

- **Limitá los recursos (CPU y RAM):** En la industria, nunca dejás que un contenedor use todo lo que quiera. En el docker-compose.yml, se suelen definir limits de memoria (ej: 256mb) para evitar que un bug o un "memory leak" de un solo servicio te tire abajo todo el servidor.

## Conclusión

Docker no es solo una herramienta, es una mentalidad de empaquetado. Al dominar Docker y Compose, garantizás que tu software sea portable, escalable y fácil de mantener para cualquier equipo.