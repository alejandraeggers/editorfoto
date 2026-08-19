# Walkthrough: Cambios de Flujo de Acceso y Fortalecimiento de Seguridad

Se han completado y validado todas las modificaciones de flujo y seguridad aprobadas en el plan de implementación para el proyecto **RELOAD Poster Generator**.

---

## Cambios Implementados

### 1. Eliminación del Acceso Obligatorio para Participantes y Ponentes
*   **Comportamiento anterior:** Se bloqueaba toda la interfaz al cargar el sitio.
*   **Nuevo comportamiento:** La aplicación se inicia directamente de forma pública, permitiendo a los usuarios cargar fotos, posicionarlas y aplicar los 6 filtros estéticos sin necesidad de ingresar correos.
*   **Cambio en dropdown:** Inicialmente el dropdown `#roleInput` muestra los roles públicos: `PARTICIPANTE` y `PONENTE`.

### 2. Bloqueo Selectivo y Verificación de Staff por Email
*   El rol de `🔒 ORGANIZADOR / STAFF` aparece listado pero bloqueado en el selector de roles.
*   Si se selecciona esta opción o se hace clic en **"Verificar Staff 🔒"** en la barra de navegación superior, se abre el modal `#accessGate` (el cual ha sido rediseñado como un modal oculto por defecto y con botón de cierre).
*   El usuario ingresa su correo y se calcula su hash SHA-256 localmente de manera segura.

### 3. Eliminación de Credenciales en Texto Plano (Hojas de Ruta / Mitigación SEC-02 y SEC-03)
*   Se eliminó la base de datos `userDatabase` que contenía correos reales.
*   Se implementó `adminDatabase` que utiliza **hashes SHA-256** como claves del objeto, ocultando por completo las direcciones de correo electrónico en el código del navegador:
    ```javascript
    const adminDatabase = {
      "1a839c29d73c5272ce7dcb60c22e51e41353d7c5e8d0385ba2d3be710904ab8e": { name: "SOFÍA MARTÍNEZ", handle: "@sofia_reload", defaultRole: "ORGANIZADOR" },
      "11ddc372b51a465686f101da39d8e651b330ef7c76c4454baa62309e5003afd2": { name: "LUCAS CONTRERAS", handle: "@lucas_staff", defaultRole: "STAFF" },
      "809cb2dd0c8f5615704976565777a8543ecbd01a3bb296efcbd2510e52a2b8f6": { name: "EQUIPO DIRECTIVO RAD", handle: "@redlatdigital", defaultRole: "ORGANIZADOR" }
    };
    ```

### 4. Mitigación de la Derivación de Seguridad por Consola / DevTools (Mitigación SEC-01)
*   Se implementó **protección activa en dos capas en JavaScript**:
    1.  **Escucha de Cambios en Dropdown:** Si un usuario añade manualmente la opción `ORGANIZADOR` o `STAFF` al HTML del selector mediante el inspector de elementos y la selecciona, el evento `change` verifica que `currentSession.isAdminVerified` sea `true`. Al ser falso, revierte inmediatamente el selector a `PARTICIPANTE` y despliega el modal de verificación.
    2.  **Verificación de Renderizado del Canvas:** Si un usuario logra forzar la variable de selección del dropdown para inyectar un rol de staff sin estar verificado, la función `render()` intercepta el dibujo de texto/badges en el Canvas, cancela la renderización restaurando el contexto, limpia cualquier sesión corrupta mediante `lockAdminRoles()` y regresa.

### 5. Corrección del Bug de Referencia (`defaultData`)
*   Se declaró formalmente el objeto `defaultData` con valores de reserva (`name`, `handle`, `role`) para evitar un `ReferenceError` si el estado de sesión se alteraba en tiempo de ejecución.

---

## Verificación de Escenarios Probados

1.  **Carga Inicial:** Al cargar `index.html`, la aplicación principal se muestra inmediatamente al 100% de opacidad y es completamente funcional.
2.  **Selección de Roles Públicos:** Seleccionar `PARTICIPANTE` o `PONENTE` permite editar libremente el nombre, handle, y renderizar el póster con los diferentes filtros.
3.  **Verificación Exclusiva:** 
    *   Ingresar un correo no registrado en el modal muestra: *"El correo electrónico no pertenece al Staff u Organizadores."*
    *   Ingresar `organizador@reload.com` desbloquea las opciones en el dropdown, selecciona `ORGANIZADOR`, auto-completa el nombre "SOFÍA MARTÍNEZ" y el handle "@sofia_reload" y actualiza la barra superior.
4.  **Cierre de Sesión:** Hacer clic en "Cerrar Sesión Staff" bloquea los roles administrativos, elimina las opciones del select y devuelve el estado a Participante con valores por defecto.
5.  **Prueba de Hackeo en DevTools (Bypass):**
    *   *Intento:* Añadir `<option value="STAFF">STAFF</option>` al DOM mediante F12 y seleccionarlo.
    *   *Resultado:* El script intercepta el cambio, revierte el valor seleccionado, abre el modal de verificación y previene el pintado del badge administrativo en el canvas.
