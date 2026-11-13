# Protocolos – Unidad de Observación (Urgencias)

Web app (PWA) de un solo archivo con los protocolos de la Unidad de Observación del Servicio de Urgencias. El usuario es clínico de Urgencias y escribe en español: responde siempre en español.

## Direcciones
- **Principal:** https://urg-hsa.github.io/protocolos-uo/ (este repositorio, `urg-hsa/protocolos-uo`, rama `main`, GitHub Pages desde la raíz).
- **Antigua:** https://infanteeric03-spec.github.io/protocolos-uo/ (repo `infanteeric03-spec/protocolos-uo`). Solo muestra un aviso rojo de cambio de dirección; ese aviso aparece únicamente cuando `location.hostname === 'infanteeric03-spec.github.io'`. No hace falta seguir publicando allí salvo que el usuario lo pida.

## Estructura
- `index.html` (~0,8 MB): toda la app (HTML + CSS + JS). Es el único archivo que se edita normalmente.
- `pdf/`: PDF originales de cada protocolo (enlazados desde la pestaña «Consultar PDF») y anexos (espirometría, recomendaciones GEA).
- `sw.js`: service worker para uso sin conexión (página: red primero con 4 s de espera; PDF/iconos: caché y actualización en segundo plano). Si se añaden PDF, añadirlos a `PRECACHE` y subir la versión `CACHE`.
- `manifest.json`, `icon-*.png`.
- `tests/avisos.py`: prueba con Playwright que marca criterios en los 15 protocolos y comprueba los avisos de color (debe dar «0 aviso(s) para revisar»). `tests/cifras.py`: compara cifras con unidades de la web con el texto de los PDF.
- `tools/modo_oscuro.py` + `tools/modo_oscuro_base.css`: regeneran el bloque de modo oscuro (`/* ==== MODO OSCURO (auto) ==== */`) a partir del CSS. Ejecutar tras cambios importantes de CSS: `python3 tools/modo_oscuro.py index.html tools/modo_oscuro_base.css` (requiere `tinycss2`).

## Conceptos del código
- Protocolos: `<section class="section" data-protocol="X" id="sec-...">` + botones de navegación `<button data-protocol data-section>`; `enterApp(p)`, `switchProtocol(p)`, `activateNavButton(btn)`, `returnToWelcome()`.
- Al añadir un protocolo hay que actualizar: `<option>` de los selects, `<li data-value>` de los menús, `.reorder-item` de Configuración. Los protocolos ocultos van al final y con estilo atenuado.
- Casillas: `ul.checklist[data-group]`; `data-any-of="true"` = sin «Seleccionar todo» (se añade automáticamente en las demás).
- Avisos de color: `.status-banner` (clases `status-neutral|low|mid|high`). Genéricos: `.status-banner-criteria[data-inc][data-exc][data-mode]` y `.status-banner[data-d-inc][data-d-mode][data-d-exc][data-t-*]` en `evaluateGenericCriteria()`. Los avisos muestran SOLO su frase (sin listas de motivos: el usuario lo pidió así).
- «📝 Generar comentario» (etiqueta Beta): botón bajo el aviso de la pestaña de inclusión/exclusión. Activo en `torax`, `fa`, `asma`, `epoc`, `neumonia` (objeto `CFG` en el script «Generar comentario»: frases por id de casilla + textos de cierre). En el resto de protocolos el botón está en gris (`.is-off`) y muestra que aún no está activo.
- Guía rápida interactiva (spotlight, flechas y mano dibujadas): array `ALL` de pasos; se abre sola la primera visita (`localStorage uoGuideSeen`).
- Botón «Instalar app» flotante, solo en la pantalla de inicio.
- Historial: el botón «atrás» del navegador vuelve a la pestaña/protocolo anterior.
- Contraseña de edición: hash SHA-256 en `<body data-settings-hash>` + código de recuperación en `data-recovery-hash`. No tocar salvo petición expresa.
- Publicación desde la propia web («Publicar en GitHub», token en localStorage): sube `index.html` con el estado del DOM. **Antes de editar, haz `git pull`**: el usuario a veces publica desde la web y podría sobrescribirse un cambio.

## Forma de trabajar acordada
- Probar cada cambio con Playwright (Chromium), en escritorio (1200 px) y móvil (390 px), y también en modo oscuro cuando afecte a colores. Ejecutar `tests/avisos.py` antes de publicar.
- Publicar = commit en `main` + push (el usuario quiere los cambios publicados directamente).
- Regla permanente: todo PDF de protocolo nuevo se añade **oculto**, con casillas en sus criterios, pestaña «Consultar PDF» y copia del PDF en `pdf/`, y se publica.
- Orden de protocolos por especialidad: Cardiología (torax, fa, ic) · Neumología (epoc, asma, neumonia) · Urología (hematuria, pielonefritis) · Digestivo (gea, hda) · Infecciosas (celulitis) · ORL (absceso) · resto (policontusionado, suboclusion, intox).
- Respuestas breves, en español, sin tecnicismos innecesarios.

## Pendiente / ideas
- Extender «Generar comentario» al resto de protocolos (y a las pestañas de alta/ingreso) cuando el usuario lo pida.
- Retirar la dirección antigua (redirección automática) cuando el usuario confirme que todos tienen el enlace nuevo.
