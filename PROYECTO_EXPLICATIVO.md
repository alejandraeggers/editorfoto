# Documento Explicativo del Proyecto: RELOAD Poster Generator

Este documento proporciona una descripción técnica detallada del proyecto **RELOAD Poster Generator**, analizando su propósito, arquitectura, interfaz de usuario, lógica de procesamiento de imágenes y pipeline de renderizado en Canvas.

---

## 1. Descripción General

**RELOAD Poster Generator** (desarrollado en `index.html`) es una aplicación web estática diseñada para que los participantes de un evento (denominado "RELOAD" o "ELAD 2026") puedan crear de forma autónoma posters personalizados para compartir en sus redes sociales (como LinkedIn o X/Twitter). 

El proyecto destaca por realizar todo el procesamiento de imágenes de manera puramente **local y en el navegador del usuario**, garantizando que las fotografías personales nunca se suban a servidores externos.

---

## 2. Arquitectura Tecnológica

La aplicación está construida sobre una arquitectura de frontend monolítica y estática en un solo archivo:

*   **Estructura (HTML5):** Uso de etiquetas semánticas y elementos interactivos para estructurar la aplicación en dos secciones principales: la pantalla de acceso (`accessGate`) y la aplicación del editor principal (`mainApp`).
*   **Estilos y Layout (Tailwind CSS):** Implementado mediante CDN (`https://cdn.tailwindcss.com`). Ofrece una interfaz responsiva tipo dashboard optimizada para dispositivos móviles y de escritorio.
*   **Tipografía y Estética:** Importación de fuentes premium desde Google Fonts:
    *   `Space Grotesk`: Utilizada para títulos con carácter tecnológico/cyberpunk.
    *   `Plus Jakarta Sans`: Utilizada para el cuerpo de texto general y controles.
    *   Filtros visuales CSS personalizados (grano de película, animaciones tipo sacudida (`shake`) para errores).
*   **Lógica y Gráficos (Vanilla JavaScript + HTML5 Canvas):** Toda la manipulación de imágenes, aplicación de filtros de píxeles, posicionamiento y exportación a alta resolución (1080x1080 px) se realiza mediante código JavaScript puro y la API de Canvas 2D.

---

## 3. Componentes de la Interfaz y Flujo de Usuario

### 3.1. Pantalla de Acceso (Access Gate)
*   **Función:** Bloquear el acceso a la aplicación principal hasta que el usuario se verifique.
*   **Modos de Ingreso:**
    1.  **Verificación de Credenciales:** Ingreso de correo electrónico que se coteja contra una base de datos simulada en memoria.
    2.  **Modo Invitado:** Acceso rápido con rol restringido ("PARTICIPANTE") para usuarios no registrados.

### 3.2. Panel de Control y Personalización (Columna Izquierda)
Se divide en dos secciones numeradas secuencialmente para guiar al usuario:
1.  **Información del Participante:** Campos de texto interactivos para modificar el Nombre Completo y el Usuario de Red Social (`handle`). El Rol en el Evento se controla dinámicamente según el nivel de credenciales de Staff/Organizador verificado o Participante público.
2.  **Estilos y Filtros:** 
    *   Composición de imagen (`Media Imagen`, `Línea Horizontal`, `Filtro Global`).
    *   Efectos de color (`RGB Glitch`, `Terminal Ámbar`).
    *   Deslizadores avanzados para controlar el zoom (escala), posición de cortes y desplazamiento horizontal (`X`) y vertical (`Y`) de la fotografía.

### 3.3. Área de Vista Previa y Carga Integrada (Columna Derecha)
*   **Zona Integrada de Carga y Canvas:** Actúa simultáneamente como dropzone interactivo de subida (arrastrar y soltar foto, tocar para elegir o pegar con `Ctrl+V`) y lienzo de renderizado en vivo (1080x1080 px).
*   **Selector de Formato:** Permite alternar la exportación entre PNG en alta definición y JPG optimizado (95%).
*   **Privacidad Local:** Muestra la garantía de seguridad informando que ninguna imagen ni dato personal se almacena ni se transmite a servidores externos.
*   **Canvas Interactivo:** Muestra en tiempo real la composición del poster. Permite arrastrar la imagen directamente con el ratón o el dedo (pantallas táctiles) para acomodar la posición del rostro.
*   **Botón de Descarga:** Exporta la composición actual como un archivo PNG en alta definición (1080x1080 px).
*   **Modal de Éxito:** Notificación que se despliega tras exportar la imagen, brindando recomendaciones para compartirla.

---

## 4. Pipeline de Renderizado y Filtros de Imagen

El núcleo gráfico de la aplicación reside en la función `render()`. Cada vez que el usuario modifica un texto, desliza la escala o cambia un filtro, se limpia el lienzo y se vuelve a dibujar en capas de abajo hacia arriba:

1.  **Fondo Oscuro:** Limpia el canvas con color `#0a090f`.
2.  **Imagen de Usuario:** Si se cargó una imagen, se traslada y rota/escala según los deslizadores o arrastre del ratón.
3.  **Filtros visuales (Canvas manipulation):**
    *   **Media Imagen (`half`):** Genera un degradado de fucsia a morado en la mitad derecha del poster.
    *   **Línea Horizontal (`eyes`):** Oscurece las secciones superior e inferior del poster y mantiene visible una franja central horizontal a la altura de los ojos.
    *   **Filtro Global (`full`):** Aplica un degradado radial de tonos magenta, morado y negro sobre todo el lienzo.
    *   **RGB Glitch (`glitch`):** Divide la imagen en 24 franjas horizontales y desplaza píxeles horizontalmente de forma aleatoria, agregando líneas de escaneo analógicas.
    *   **Entra a la Terminal (`terminal`):** Convierte la imagen a fósforo verde (escala de grises con tinte verde), la mezcla al 65% con la original y dibuja circuitos impresos de cobre interactivos en el canvas.
4.  **Marcas de Calibración:** Esquinas del visor de cámara y tramas de puntos transparentes.
5.  **Textos y Branding:**
    *   Títulos: "I'M PARTICIPATING IN ELAD 2026".
    *   Nombre del participante (con salto de línea si tiene más de 2 palabras).
    *   Red social con prefijo `@` asegurado.
    *   Rol de participación en un recuadro sólido fucsia.
    *   Thumbnail miniatura de la foto en la esquina inferior izquierda.
