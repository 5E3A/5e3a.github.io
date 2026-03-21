+++
date = '2026-03-21T19:34:18-03:00'
draft = false
title = 'Git: Cómo modificar el mensaje de un commit'
tags = ['Git']
+++
Puede pasar en ocasiones que el mensaje del commit no es el apropiado para el mismo o hubo un problema de tipeo y quedo mal (no es tan grave), pero si fuera el caso que necesites cambiarlo.

### Commit Remoto

#### No es el último

Este es el caso de "emergencia" cuando el error ya es público. Modificar un commit remoto implica reescribir la historia, lo que significa que el identificador (SHA) del commit cambiará.

**1. Identificar la profundidad**

Si es el último, usamos amend. Si es uno anterior, necesitamos el Rebase Interactivo:

```bash
git rebase -i HEAD~2  # Para ver los últimos 2
```

**2. La edición (Reword)**

En el editor, cambiamos pick por reword en el commit deseado. Guardamos, cerramos y Git nos pedirá el nuevo mensaje.

**3. El Sincronismo Roto**

Al terminar el rebase, Git te dirá que tu rama local y la remota han "divergido". No hagas un git pull, porque eso creará un commit de merge innecesario y duplicará el error.

**4. El "Empujón" Final (Force Push)**

Para que el servidor acepte tu nueva versión de la historia, debés forzar la subida:

```bash
git push origin <tu-rama> --force
```

El uso de --force en ramas compartidas (como main o develop en equipos grandes) puede causar conflictos a tus compañeros, siempre avisá antes de reescribir historia compartida.

**5. Ver el cambio aplicado**

Visualizar en una linea los últimos 5 commits.

```bash
git log --oneline -n 5
```

#### Es el último

```bash
git commit --amend -m "..."
```

```bash
git push --force
```

### Commit Local

#### Es el último

```bash
git commit --amend -m "..."
```

#### No es el último

```bash
git rebase -i HEAD~n
```

En el editor cambiamos pick por reword (o solo r). Al guardar, se abrirá un segundo editor para escribir el nuevo mensaje.

### Resumen de comandos

| Escenario | ¿Es Remoto? | Comando | Acción Final |
| :--- | :--- | :--- | :--- |
| **Último** | No | `git commit --amend -m "..."` | - |
| **Último** | **Sí** | `git commit --amend -m "..."` | `git push --force` |
| **Anterior (n)** | No | `git rebase -i HEAD~n` | (Reword + Save) |
| **Anterior (n)** | **Sí** | `git rebase -i HEAD~n` | `git push --force` |