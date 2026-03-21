+++
date = '2026-03-21T17:35:17-03:00'
draft = false
title = 'Arquitectura de Identidad: De los Fundamentos a los Modelos Modernos'
tags = ["architecture", "security", "identity-management", "auth", "best-practices"]
+++
En el desarrollo de software, solemos agrupar todo bajo el botón de "Login", pero detrás de esa acción existe una jerarquía de conceptos que definen la robustez y escalabilidad de un sistema. Entender la diferencia entre quién sos y qué podés hacer es el primer paso para construir aplicaciones seguras.

### 1. El Cimiento: Autenticación vs. Autorización
Es común confundirlos, pero operan en capas distintas del modelo de seguridad:

- **Autenticación (AuthN - Identity):** Es el proceso de verificar que un usuario es quien dice ser. Se basa en factores de conocimiento (contraseña), posesión (un token físico o el celular) o inherencia (biometría).

- **Autorización (AuthZ - Permissions):** Una vez que sabemos quién sos, la autorización determina a qué recursos tenés acceso y qué acciones podés ejecutar (leer, escribir, borrar).

### 2. Gestión del Estado: Stateful vs. Stateless
Antes de elegir una tecnología, debemos decidir cómo el sistema recordará al usuario.

#### Sesiones Tradicionales (Stateful)
El servidor genera un Session ID que se guarda en una base de datos (o Redis) y se envía al cliente en una cookie.

**Pro**: Es fácil revocar el acceso instantáneamente borrando la sesión del lado del servidor.

**Contra**: Difícil de escalar horizontalmente. Si tenés múltiples servidores, todos deben consultar la misma base de datos de sesiones, creando un cuello de botella.

#### Tokens (Stateless)
El servidor firma un objeto (como un JWT) y se lo entrega al cliente. El servidor no "recuerda" al usuario; confía en la firma digital del token en cada petición.

**Pro**: Escalabilidad total. El servidor no necesita base de datos para validar el token.

**Contra**: La revocación es compleja. Si un token es robado, sigue siendo válido hasta que expire, a menos que se implementen listas de baneo (lo cual vuelve al sistema un poco stateful).

### 3. Modelos de Control de Acceso (AuthZ)
¿Cómo decidimos quién entra a dónde? Aquí es donde la arquitectura se vuelve interesante:

#### RBAC (Role-Based Access Control)
El acceso se basa en Roles (Admin, Editor, Usuario). Es el modelo más común por su simplicidad.

**Ejemplo:** "Si el usuario tiene el rol 'Editor', puede publicar posts".

#### ABAC (Attribute-Based Access Control)
Es la evolución hacia sistemas complejos. El acceso se decide mediante Políticas que evalúan atributos:

- **Sujeto:** ¿Quién es el usuario? (Rol, antigüedad).
- **Recurso:** ¿A qué quiere acceder? (Un archivo contable, un post de blog).
- **Acción:** ¿Qué quiere hacer? (Ver, Editar).
- **Entorno:** ¿Desde dónde y cuándo? (IP de la oficina, horario laboral).

### 4. Estándares Modernos: OAuth 2.0 y OIDC
No reinventes la rueda. La industria ya resolvió cómo delegar acceso de forma segura.

**OAuth 2.0:** Es un framework de Autorización. Permite que una aplicación acceda a recursos en nombre de un usuario sin conocer su contraseña (ej: una app de calendario pidiendo acceso a tus contactos de Google).

**OpenID Connect (OIDC):** Es una capa de Identidad que corre sobre OAuth 2.0. Añade un "ID Token" que estandariza la información del perfil del usuario. Es lo que usamos cuando vemos el botón "Log in with Google/GitHub".

### 5. JWT (JSON Web Tokens): El Vehículo de Información
El JWT es el estándar para transmitir estas afirmaciones (claims) de forma segura. Se compone de tres partes separadas por puntos:

**Header:** Algoritmo y tipo de token.

**Payload:** Los datos del usuario (claims). Nunca pongas contraseñas acá, solo es Base64.

**Signature:** El hash que asegura que el token no fue alterado.

#### Buenas Prácticas de Seguridad
Uso de Cookies Seguras: Almacenar tokens en cookies HttpOnly, Secure y SameSite=Strict para mitigar ataques XSS y CSRF.

**Refresh Tokens:** Usar Access Tokens de corta vida (15 min) y Refresh Tokens de larga vida (7 días) almacenados de forma segura para renovar la sesión sin pedir credenciales constantemente.

**Validación Local vs. Remota:** En microservicios, el Gateway valida el token y los servicios internos confían en esa validación para ganar performance.

### Conclusión
La arquitectura de identidad no se trata de elegir una librería, sino de entender el flujo de confianza. Ya sea que uses sesiones o tokens, RBAC o ABAC, el objetivo siempre es el mismo: garantizar que el recurso correcto sea accedido por la persona correcta, bajo las condiciones correctas.