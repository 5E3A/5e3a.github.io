+++
date = '2026-03-10T22:37:07-03:00'
draft = false
title = 'Rollinks'
description = ""
tags = ["chrome-extension", "javascript", "productivity", "web-management"]
categories = ["Proyectos", "Herramientas"]
github = "https://github.com/5E3A/chrome-extension-rollinks"
+++
Extensión de Chrome tipo side panel diseñada para capturar, categorizar y filtrar tus links visitados de forma mas ordenada. [PENDIENTE SUBIR PROYECTO A GITHUB]
{{< figure src="screenshot.png" caption="Vista previa extension - light theme. Al hacer hover sobre los enlaces, se habilitan las opciones de edición, eliminación o para marcarlo. Los enlaces que están abiertos actualmente se destacan con una sombra (estado open)." alt="Captura de pantalla" >}}

## Rollinks: Gestión Inteligente de Pestañas

**Rollinks** es una extensión de Chrome diseñada para atacar uno de los problemas más comunes en el flujo de trabajo: el caos de pestañas abiertas y la pérdida de referencias importantes.

### El Problema
El "Tab Hoarding" (acumular pestañas) degrada el rendimiento del navegador y fragmenta la atención. Guardar links en "Favoritos" convencionales suele ser un cementerio de enlaces que nunca se vuelven a consultar por falta de contexto y organización.

### La Solución
Una herramienta ligera que permite categorizar, filtrar y comentar enlaces en tiempo real, transformando una lista caótica en un centro de recursos accionables.

### Características Principales
* **Categorización Dinámica:** Clasificación de enlaces según el contexto del proyecto.
* **Filtros Avanzados:** Búsqueda rápida por tipo de contenido o etiquetas.
* **Notas y Comentarios:** Posibilidad de añadir contexto a cada link para recordar por qué se guardó.

### Stack Técnico
* **UI/Styles:** Tailwind CSS para un diseño responsivo y utilitario.
* **Logic:** JavaScript (Vanilla) enfocado en la manipulación eficiente del DOM.
* **Storage:** Chrome Storage API para persistencia de datos local.
* **API:** Web Extensions API para la integración nativa con el navegador.

### Arquitectura y Componentes (Manifest V3)
Desarrollar bajo el estándar Manifest V3 implicó un cambio de paradigma en la gestión de recursos. Aquí detallo los componentes clave y cómo interactúan en Rollinks:

**Service Workers (Gestión Efímera):** En V3, el archivo background.js funciona como un Service Worker. A diferencia de las versiones anteriores que consumían RAM de forma constante, este componente es efímero: se activa para procesar eventos (como abrir una pestaña) y se suspende automáticamente al terminar. Esto garantiza un impacto mínimo en el rendimiento del sistema.

**Side Panel API (UX Multitarea):** Elegí esta API en lugar de un Popup tradicional para permitir una experiencia persistente. Mientras que un popup se cierra al perder el foco, el panel lateral permite gestionar links y escribir notas mientras navegás activamente por diferentes sitios.

**Chrome Storage & DOM:** La lógica (escrita en Vanilla JS) manipula el DOM (la estructura de la página) para extraer títulos y URLs, persistiendo los datos de forma segura mediante la Chrome Storage API. Esto asegura que tus categorías y comentarios se mantengan incluso si reiniciás el navegador.

**Host Permissions & Privacidad:** El uso de permisos granulares y el acceso a file:// permite que Rollinks sea una herramienta integral. Al declarar solo lo estrictamente necesario, se respeta el Principio de Menor Privilegio, fundamental en la seguridad de extensiones modernas.

### Configuración para Archivos Locales (PDFs)
Para que Rollinks pueda gestionar documentos abiertos desde tu disco duro, Chrome requiere un paso adicional de seguridad por parte del usuario:

1. Abrir la gestión de extensiones: chrome://extensions.

2. Buscar la tarjeta de Rollinks y hacer clic en Detalles.

3. Activar el interruptor: "Permitir acceso a las URLs de archivo".

> Nota de Seguridad: Este permiso es necesario porque, por defecto, los navegadores aislan el sistema de archivos local de las extensiones para proteger tu privacidad. Al activarlo, Rollinks podrá indexar esos manuales o documentación en PDF que consultás offline.

### Próximos Pasos (Roadmap)
Rollinks sigue en evolución constante dentro de mi laboratorio. Estas son las funcionalidades planeadas:

**Exportación de Datos:** Implementar la descarga de la biblioteca en formatos JSON y CSV para asegurar la portabilidad de la información.

**Sincronización en la Nube:** Integración con Google Drive API para respaldar los links y notas de forma persistente.

**Categorización por IA Local:** Implementar sugerencias automáticas de etiquetas basadas en el contenido del sitio visitado utilizando NLP (Natural Language Processing).

### Nota sobre la implementación de IA
Para este desarrollo, he optado por la API de IA integrada en Chrome (Gemini Nano) frente a modelos externos (como OpenAI o Anthropic) por tres pilares fundamentales:

**Privacidad:** Los datos no salen del navegador; el procesamiento es 100% local ("tus datos, tu máquina").

**Latencia Cero:** Al evitar llamadas de red (fetch), la categorización es instantánea.

**Costo:** Aprovecha el hardware del usuario, eliminando la necesidad de infraestructura de servidores o suscripciones.

**Desafíos técnicos a resolver:** Gestionar la disponibilidad del modelo en diferentes versiones de Chrome, manejar los límites de contexto de un modelo compacto (resumiendo el DOM antes del envío) y la configuración de permisos específicos (aiLanguageModel) en el Manifest V3.

### Conclusión: El software como proceso iterativo
Rollinks nació de una necesidad personal de orden, pero se convirtió en el terreno perfecto para experimentar con las capacidades modernas de los navegadores y la IA local.

Este proyecto es un recordatorio de que, a veces, las herramientas más útiles son aquellas que resolvemos para nosotros mismos, aplicando buenas prácticas de arquitectura y manteniendo el foco en la privacidad del usuario. Si te interesa el código o querés proponer una mejora, ¡nos vemos en el repo de GitHub! (PENDIENTE)