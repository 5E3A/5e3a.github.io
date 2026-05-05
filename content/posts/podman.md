---
title: "Desarrollo Local Seguro: Entorno Full-Stack con Podman"
date : '2026-05-04T22:15:16-03:00'
tags: ["Podman", "DevOps", "Containers", "Security", "Infrastructure", "Go", "React"]
categories: ["Technical Guide"]
author: "Sebastian"
description: "Guía completa para configurar un entorno de desarrollo local seguro y 'rootless' usando Podman en Windows y Linux."
---

En el desarrollo profesional, Docker es el estándar de la industria. Sin embargo, para proyectos personales, **Podman** ofrece una capa de seguridad superior. Al ser **daemonless** (sin un demonio central) y **rootless** (se ejecuta sin privilegios de administrador), protegés tu sistema host de posibles vulnerabilidades en la cadena de suministro (*supply chain*) de paquetes como `npm` o `pip`.

> **Nota:** Es cierto que Docker permite un modo **Rootless**, pero requiere una configuración manual más tediosa. Podman nos da esa seguridad "de fábrica", permitiéndonos enfocarnos en el código y no en la administración del motor de containers. Articulo sobre docker <a href="/posts/docker-y-docker-compose-guia/">aquí</a>.

## 1. Instalación del Entorno

### Opción A: Linux (Ubuntu/Debian)

```bash
# Actualizar repositorios e instalar Podman y su orquestador
sudo apt update
sudo apt install podman podman-compose -y
```

### Opción B: Windows (WSL2)
En Windows, Podman utiliza el motor de **WSL2** para ejecutar los contenedores de forma aislada:

1.  **Habilitar WSL2:** Abrí PowerShell como administrador y ejecutá `wsl --install`.
2.  **Instalar Podman Desktop:** Descargá el ejecutable desde [podman-desktop.io](https://podman-desktop.io/).
3.  **Inicializar el entorno:**
    ```powershell
    podman machine init
    podman machine start
    ```
## 2. Estructura del Proyecto "Hola Mundo"
Para mantener la modularidad y el aislamiento, organizamos el proyecto evitando instalar dependencias directamente en el host:
```text
mi-proyecto/
├── api/              # Backend (ej: Go, Node, Python)
|    └── Dockerfile
├── web/              # Frontend (ej: React + TS, Vue)
|    └── Dockerfile
├── compose.yaml      # Orquestación de servicios
└── .env              # Variables de entorno
```

```dockerfile
# Dockerfile api
FROM golang:1.22-alpine
WORKDIR /app

# Copiamos los archivos de módulos
COPY go.mod ./
# Si ya tenés un go.sum, descomentá la siguiente línea:
# COPY go.sum ./

RUN go mod download

COPY . .

RUN go build -o main .

EXPOSE 8080

CMD ["./main"]
```

```dockerfile
# Dockerfile web
FROM node:lts-alpine
WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5173

# El flag --host es fundamental para que Vite 
# acepte conexiones externas al contenedor
CMD ["npm", "run", "dev", "--", "--host"]
```
### El archivo de Orquestación (`compose.yaml`)
Este archivo le indica a Podman cómo levantar cada pieza de forma segura y conectarlas en una red privada.
```yaml
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD:-password123}
    volumes:
      - pgdata:/var/lib/postgresql/data:Z # El flag :Z es clave para SELinux en Linux
    networks:
      - secure-network

  backend:
    build: ./api
    ports:
      - "8080:8080"
    depends_on:
      - db
    networks:
      - secure-network

  frontend:
    build: ./web
    ports:
      - "5173:5173"
    networks:
      - secure-network

networks:
  secure-network:
    driver: bridge

volumes:
  pgdata:
```
## 3. Seguridad en la Cadena de Suministro
Al usar Podman, mitigás riesgos críticos:
*   **Aislamiento de Privilegios:** Si un paquete malicioso intenta escalar privilegios dentro del contenedor, se encontrará con que el proceso corre con los permisos de tu usuario estándar, no como `root` del sistema.
*   **Host Limpio:** No necesitás tener instalados compiladores o runtimes (Node, Go, etc.) en tu sistema principal, evitando que scripts de instalación (como por ejemplo `postinstall` en npm) ejecuten código fuera del contenedor.

## 4. Comandos para Levantar y Trabajar

Una vez que tengas tus archivos listos, el flujo de trabajo es idéntico en Windows y Linux.

### Levantar el proyecto
Desde la raíz de tu carpeta, ejecutá:
```bash
# Levantar todos los servicios en segundo plano
podman-compose up -d
```

Si al levantar tu entorno recibís un error de Port already in use, verificá quién tiene el puerto ocupado:

* **En Linux/WSL2:** `ss -tulpn | grep LISTEN`

* **En Windows (PowerShell):** `netstat -ano | findstr LISTENING` o `netstat -abno | findstr LISTENING`

### Comandos útiles de Workflow
Para mantener la filosofía de **"nada en el host"**, usá estos comandos para interactuar con el entorno:

*   **Instalar librerías en el frontend:**
    `podman exec -it <nombre_contenedor_frontend> npm install axios`
*   **Ver logs en tiempo real:**
    `podman-compose logs -f`
*   **Detener todo el entorno:**
    `podman-compose down`
*   **Verificar que todo corre como tu usuario (y no root):**
    `podman unshare cat /proc/self/uid_map`
*   **Levantar el proyecto con un solo comando:** -d para detached (segundo plano), --build para asegurar que compile cambios
    `podman-compose up -d --build`

> **Tip:** Si venís de usar Docker, podés añadir un alias en tu `.bashrc` o perfil de PowerShell: `alias docker=podman`. Esto te permite seguir usando tus comandos habituales mientras aprovechás la seguridad de Podman por debajo.

## 5. Seguridad Avanzada

Si bien ser *rootless* da una ventaja enorme, se puede elevar la vara con estos dos conceptos:

*   **Solo Lectura (Read-Only FS):** Podés configurar tus contenedores (especialmente el frontend) para que el sistema de archivos sea de solo lectura. Si un atacante logra entrar, no podrá escribir scripts ni persistir malware.
    *   En el `compose.yaml` se agrega: `read_only: true`.
*   **Capacidades Limitadas (Drop Capabilities):** Por defecto, un contenedor tiene ciertos "permisos" de kernel. Podés ser paranoico y quitarlos todos, dejando solo los necesarios para que el proceso corra.
    *   Se usa: `cap_drop: [ "all" ]`.
*   **Escaneo de Vulnerabilidades nativo:** Podman se integra muy bien con herramientas como **Trivy**. Podés correr `podman run --rm -v /var/run/podman.sock:/var/run/podman.sock aquasec/trivy image <tu_imagen>` para analizar tus paquetes antes de levantarlos.

## 6. ¿Es transparente pasar de Podman a Docker?

La respuesta corta es **sí, casi al 100%**. 

Podman fue diseñado para ser un reemplazo *drop-in* de Docker. De hecho, cumple con los estándares de la **OCI (Open Container Initiative)**, lo que garantiza que las imágenes y los archivos de configuración sean compatibles.

### Puntos a tener en cuenta en la migración

*   **Imágenes:** Una imagen construida en Podman se puede subir a Docker Hub (o cualquier registry) y Docker la podrá descargar y ejecutar sin problemas.
*   **Docker Compose:** Podman utiliza la misma sintaxis que Docker para el archivo `compose.yaml`. Si el proyecto escala y necesitás pasarlo a un entorno laboral donde solo usan Docker, solo tenés que cambiar el comando de ejecución.
*   **El Socket:** La principal diferencia técnica es que Docker depende de un socket (`/var/run/docker.sock`). Si usás herramientas que interactúan directamente con ese socket, en Podman deberás habilitar el servicio de `podman.socket` para "emular" ese comportamiento.

### El Alias
Para que la transición sea invisible en tu día a día:
```bash
# En tu .bashrc o .zshrc
alias docker='podman'
alias docker-compose='podman-compose'
```
Aunque vale aclarar que forzar un alias puede ser contraproducente si te toca saltar entre proyectos personales (donde usás Podman por seguridad) y proyectos laborales o de terceros que dependen estrictamente de Docker.