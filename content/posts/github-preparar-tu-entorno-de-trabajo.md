+++
date = '2026-03-10T23:44:16-03:00'
draft = false
title = 'Github: preparar tu entorno de trabajo'
tags = ['Git', 'Github']
+++
Considerar ya tener una cuenta de GitHub creada, el siguiente paso es: **preparar tu entorno de trabajo**. Configurar Git en tu computadora local no es solo instalar un programa; es establecer una conexión segura para que GitHub sepa quién eres y te permita subir código sin pedirte la contraseña en cada segundo.

### 1\. Instalación de Git


Antes de configurar, necesitamos el motor en nuestra máquina:

*   **Windows:** Descarga el instalador oficial desde [git-scm.com](https://git-scm.com/). Al instalar, recomiendo elegir "Git Bash" como terminal por defecto.
*   **Linux (Debian/Ubuntu):** Abre tu terminal y ejecuta: `sudo apt update && sudo apt install git`.

### 2\. Configurando tu Identidad (Nombre y Email)


Cada commit que hagas llevará una firma. Aquí configuramos cómo te verá el mundo.

#### Privacidad: Ocultar tu email real

Si no queres que tu email personal sea público en los logs de Git, GitHub te ofrece un email "no-reply".

1.  Ve a GitHub -> **Settings** -> **Emails**.
2.  Activa _"Keep my email addresses private"_.
3.  Copia el email que te da GitHub (algo como `ID+username@users.noreply.github.com`).

Ahora, en tu terminal (Git Bash o terminal de Linux), ejecuta:

#### Configura tu nombre (el mismo que en GitHub)
```bash
git config --global user.name "Tu Nombre o Usuario"
```

#### Configura el email (usa el no-reply de GitHub para privacidad)
```bash
git config --global user.email "ID+username@users.noreply.github.com"
```

### 3\. Conexión Segura: Creación de la Llave SSH

Para no escribir tu usuario y contraseña siempre, usamos **SSH (Secure Shell)**. Es una pareja de llaves: una privada (vive en tu PC) y una pública (se la das a GitHub).

#### Paso A: Generar la llave en tu PC

Ejecuta este comando (funciona igual en Windows Git Bash y Linux):

```bash
ssh-keygen -t ed25519 -C "tu-email-de-github@ejemplo.com"
```

Presiona **Enter** a todo para dejar las opciones por defecto. Esto creará dos archivos en una carpeta oculta llamada `.ssh`.

#### Paso B: Copiar la llave pública

*   **Windows:** `cat ~/.ssh/id_ed25519.pub | clip` (lo copia al portapapeles).
*   **Linux:** `cat ~/.ssh/id_ed25519.pub` y copia el texto que empieza con `ssh-ed25519...`.

#### Paso C: Pegar la llave en GitHub

1.  Ve a GitHub -> **Settings** (clic en tu foto arriba a la derecha).
2.  En el menú lateral, busca **SSH and GPG keys**.
3.  Haz clic en **New SSH Key**.
4.  Ponle un título (ej: "Mi Laptop Windows") y pega el contenido en el cuadro **Key**.
5.  Guarda los cambios.

### 4\. Verificando la conexión

Antes asegúrate de que la clave esté cargada (SSH Agent)

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Luego en tu terminal escribe:

```bash
ssh -T git@github.com
```

Si pregunta si confías en el host, escribe `yes`. Deberías ver un mensaje como: _"Hi \[tu-usuario\]! You've successfully authenticated..."_.

### 5\. Primeros pasos: Clonar y Trabajar

Ahora que estás conectado, ya podes traer tus proyectos. Recorda usar siempre la opción **SSH** (no HTTPS) al copiar el link de un repositorio en GitHub.

```bash
git clone git@github.com:usuario/repositorio.git
```

### Conclusión

Configurar Git localmente es el primer paso. Al usar llaves SSH y el email de privacidad, estás protegiendo tu identidad "Hasta donde entiendo" y asegurando que tu flujo de trabajo sea fluido y sin fricciones.

**Tip Extra:** Si trabajas en varias computadoras, repetí el proceso de la llave SSH en cada una. Nunca compartas tu llave privada (el archivo que NO termina en .pub) con nadie.