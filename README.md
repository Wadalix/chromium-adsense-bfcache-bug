# chromium-adsense-bfcache-bug
Chromium mobile back/forward cache bug (Blink) freezing Google AdSense scripts and web layouts, causing dynamic text stalls and stuck vignette hashes.

## Technical Incident Report: Chromium Bfcache Restoration of Stale AdSense Vignette State

### Bug Reference Data
- **Chromium Bug Tracker ID:** 544217146
- **Affected Component:** `Blink>History>BackForwardCache`
- **Impacted Services:** Google AdSense (Mobile Vignette Ads), Core Web Vitals (INP, CLS), Dynamic Site Scripts.
- **AdSense Support Thread:** https://support.google.com/adsense/thread/447696714/bug-doble-en-intersticiales-bfcache-aria-hidden-rompe-anuncios-m%C3%B3viles?hl=es


### Real Bug Mechanism & Technical Breakdown
There is a fundamental state-restoration flaw when a user navigates away from a page via an AdSense mobile vignette link and subsequently triggers a Back/Forward Cache (bfcache) hit in modern mobile Chromium.

1. **Initial Trigger:** Clicking a dynamic element (e.g., a job posting link) initializes the AdSense vignette overlay. At this precise point, the console triggers a red warning: *"Blocked aria-hidden on a <body> element because it would hide the entire accessibility tree"*. 
2. **Navigation:** Upon closing/progressing past the ad, the DOM is correctly cleaned up, the `aria-hidden` attribute disappears from the `<body>`, and the user is redirected to a completely clean internal destination URL. No further console errors are thrown.
3. **The Deadlock on Back Navigation:** When the user clicks the browser's "Back" button, Chromium bypasses a clean network reload and fetches the previous page state directly from the `bfcache` snapshot. 

**The Root Cause Failure:** Chromium caches the state of the homepage *prior* to the complete programmatic teardown of the vignette infrastructure. Upon restoration, the homepage wakes up entirely frozen. The browser address bar remains permanently hijacked with the uncleaned `/#google_vignette` hash, dynamic text items/feeds stall without fetching updates, and all standard responsive and sticky ad units freeze in place, generating an absolute drop in mobile RPM impressions.

---

### Status and Publisher Blockade
Official Sellside engineering channels have privately confirmed that their monetization scripts were architected under a hard "NO BFCACHE" assumption. They have classified this systemic failure as a "long-term" problem. 

Publishers are currently left in a complete deadlock: attempting to patch the layout or forcefully reload the window via automated script manipulation directly breaches Google AdSense policies regarding ad code alteration and invalid traffic generation, leading to immediate automated account bans. Leaving the code unmodified forces the mobile UI into absolute zero-responsiveness, destroying real-world user metrics and mobile ad impressions.


---

# 🇪🇸 Versión en Español: Informe de Incidente Técnico

## Informe de Incidente Técnico: Restauración de Estado Corrupto de la Viñeta de AdSense en el Bfcache de Chromium

### Datos de Referencia del Bug

*   **Chromium Bug Tracker ID:** 544217146
*   **Componente Afectado:** Blink>History>BackForwardCache
*   **Servicios Impactados:** Google AdSense (Anuncios de Viñeta Móvil), Core Web Vitals (INP, CLS), Scripts Dinámicos del Sitio.
*   **Hilo de Soporte de AdSense:** https://support.google.com/adsense/thread/447696714/bug-doble-en-intersticiales-bfcache-aria-hidden-rompe-anuncios-m%C3%B3viles?hl=es

### Mecanismo Real del Bug y Desglose Técnico

Existe un fallo fundamental en la restauración del estado cuando un usuario navega fuera de una página a través de un enlace de viñeta móvil de AdSense y, posteriormente, provoca un impacto en la caché de navegación atrás/adelante (bfcache) en las versiones modernas de Chromium móvil.

1. **Detonante Inicial:** Al hacer clic en un elemento dinámico (por ejemplo, un enlace de oferta de empleo), se inicializa la capa superpuesta de la viñeta de AdSense. En este punto preciso, la consola dispara una advertencia roja: *"Blocked aria-hidden on a element because it would hide the entire accessibility tree"* (Bloqueado aria-hidden en un elemento porque ocultaría todo el árbol de accesibilidad).
2. **Navegación:** Al cerrar o avanzar más allá del anuncio, el DOM se limpia correctamente, el atributo aria-hidden desaparece del `<body>` y el usuario es redirigido a una URL de destino interna completamente limpia. No se lanzan más errores en la consola.
3. **El Bloqueo en la Navegación Atrás:** Cuando el usuario hace clic en el botón "Atrás" del navegador, Chromium se salta una recarga limpia desde la red y recupera el estado anterior de la página directamente desde la captura guardada en el bfcache.

**La Causa Raíz del Fallo:** Chromium guarda en la caché el estado de la página de inicio (homepage) *antes* de que se complete el desmontaje programático de la infraestructura de la viñeta. Al restaurarse, la página de inicio se despierta completamente congelada. La barra de direcciones del navegador queda permanentemente secuestrada con el hash `/#google_vignette` sin limpiar, los elementos/fuentes de texto dinámicos se paralizan sin recuperar actualizaciones, y todas las unidades de anuncios estándar adaptables y fijas se congelan en su lugar, generando una caída absoluta en las impresiones de RPM móvil.

### Estado Actual y Bloqueo para el Editor

Los canales oficiales de ingeniería de Sellside han confirmado de forma privada que sus scripts de monetización fueron arquitectónicamente diseñados bajo la premisa estricta de "NO BFCACHE". Han clasificado este fallo sistémico como un problema a "largo plazo".

Los editores nos encontramos actualmente en un callejón sin salida absoluto: intentar parchear el diseño o forzar la recarga de la ventana mediante la manipulación automatizada de scripts infringe directamente las políticas de Google AdSense sobre la alteración del código de anuncios y la generación de tráfico inválido, lo que lleva a la suspensión automatizada e inmediata de la cuenta. Dejar el código sin modificar obliga a la interfaz de usuario móvil a una falta total de respuesta absoluta, destruyendo las métricas de usuario del mundo real y las impresiones de anuncios móviles.
