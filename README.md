AdsCreative Generator
Web App de una sola página que genera hasta 10 ideas de creativos para campañas de venta por mensajes en Meta Ads (WhatsApp y Messenger). Cada idea usa un sesgo cognitivo y un ángulo de venta distintos, y se estructura en Hook, Cuerpo y CTA.
Es un sitio 100 % estático: no necesita servidor, build ni dependencias. Se aloja gratis en GitHub Pages.
---
1. Cómo está construido
Estructura del proyecto
```
/
├── index.html   ← toda la aplicación (HTML + Tailwind + JavaScript)
└── README.md    ← este archivo
```
Tecnologías
Pieza	Detalle
HTML5	Estructura de la página
Tailwind CSS	Cargado por CDN (`cdn.tailwindcss.com`), con `darkMode: 'class'`
JavaScript (Vanilla)	Lógica integrada en `index.html`, sin frameworks
API de Gemini	Modelo `gemini-2.5-flash`, llamado con `fetch` desde el navegador
Secciones de la interfaz
Cabecera con título y botón para cambiar entre modo oscuro (por defecto) y claro.
Panel de control: campo de API Key (tipo password), selector de nicho (con opción "Otro" para escribirlo) y botón Generar Ideas de Creativos con spinner de carga.
Contenedor de resultados (`#results`): aquí se inyecta el HTML que devuelve Gemini.
Esquema de Meta Ads: sección estática con la estructura recomendada: Campaña de Interacción → Conjunto de Anuncios segmentado → 3 a 5 Anuncios con distintos ángulos.
Flujo de la lógica (JavaScript)
El usuario ingresa su API Key y elige un nicho.
`buildPrompt(niche)` construye el prompt con las instrucciones de Media Buyer y neuromarketing, y le añade una semilla aleatoria y ángulos sugeridos al azar para que cada generación sea distinta.
`callGemini()` hace un `POST` a:
```
   https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent
   ```
La clave se envía en la cabecera `x-goog-api-key` (no en la URL).
`temperature: 1.4`, `topP: 0.95`, `topK: 64` para favorecer la creatividad.
Timeout de 90 segundos con `AbortController`.
La respuesta (HTML con clases de Tailwind) pasa por `sanitizeHTML()`, que elimina `<script>`, `<iframe>`, atributos `on*` y enlaces `javascript:`, y luego se inyecta en `#results`.
Los errores (clave inválida, cuota, timeout, sin conexión, respuesta bloqueada) se muestran como notificaciones en pantalla.
Parámetros que puedes modificar
Están al inicio del bloque `<script>` de `index.html`:
```js
const MODEL = 'gemini-2.5-flash';  // modelo de Gemini a usar
const TIMEOUT_MS = 90000;          // tiempo máximo de espera (ms)
```
Para añadir nichos, agrega una línea `<option>` dentro del `<select id="niche">`.
---
2. Cómo obtener y usar la API Key de Gemini
Entra a Google AI Studio con tu cuenta de Google.
Pulsa Create API key y copia la clave (empieza con `AIza...`).
Abre la web app, pega la clave en el campo API Key de Gemini.
Opcional: marca Recordar en este navegador para guardarla en `localStorage` y no tener que pegarla cada vez.
Seguridad de la clave
La clave no se escribe en el código ni se sube a GitHub: cada usuario pega la suya en el navegador.
Se envía únicamente a `generativelanguage.googleapis.com`.
Nunca pegues tu clave directamente en `index.html`: al ser un sitio estático y público, cualquiera podría verla y usarla.
Si activas "Recordar", la clave queda guardada en ese navegador. No lo uses en equipos compartidos.
Si crees que tu clave se filtró, elimínala en Google AI Studio y crea otra.
---
3. Cómo subir el sitio a GitHub Pages
Opción A: desde la web de GitHub (sin instalar nada)
Crea una cuenta en github.com si aún no tienes una.
Pulsa New repository (el botón verde o el `+` arriba a la derecha).
Ponle un nombre (por ejemplo `adscreative-generator`), déjalo en Public y pulsa Create repository.
Pulsa uploading an existing file, arrastra `index.html` y `README.md`, y pulsa Commit changes.
Ve a Settings → Pages.
En Build and deployment → Source, elige Deploy from a branch.
En Branch, selecciona `main` y la carpeta `/ (root)`, y pulsa Save.
Espera 1 a 2 minutos. Tu sitio quedará en:
```
   https://TU-USUARIO.github.io/adscreative-generator/
   ```
Opción B: desde la terminal con Git
```bash
# 1. Dentro de la carpeta con index.html y README.md
git init
git add .
git commit -m "Primera versión de AdsCreative Generator"

# 2. Conecta con tu repositorio (créalo antes vacío en GitHub)
git branch -M main
git remote add origin https://github.com/TU-USUARIO/adscreative-generator.git
git push -u origin main
```
Después sigue los pasos 5 a 8 de la Opción A para activar Pages.
Actualizar el sitio después
Edita `index.html`, y luego:
```bash
git add .
git commit -m "Describe tu cambio"
git push
```
GitHub Pages se actualiza solo en uno o dos minutos.
---
4. Probar en local
No necesitas servidor: haz doble clic en `index.html` y se abre en el navegador. Si prefieres un servidor local:
```bash
python3 -m http.server 8000
# abre http://localhost:8000
```
---
5. Solución de problemas
Mensaje / síntoma	Causa probable	Solución
"API Key inválida"	Clave mal copiada o eliminada	Copia de nuevo la clave desde AI Studio
"Acceso denegado" (403)	La clave no tiene permisos o tiene restricciones	Revisa las restricciones de la clave en Google Cloud
"Límite de uso alcanzado" (429)	Cuota gratuita agotada o demasiadas solicitudes	Espera unos minutos o revisa tu cuota
"Modelo no encontrado" (404)	El nombre del modelo cambió o fue retirado	Cambia `MODEL` por un modelo vigente de Gemini
"Tiempo de espera agotado"	La API tardó más de 90 s	Inténtalo de nuevo o sube `TIMEOUT_MS`
"No se pudo conectar con la API"	Sin internet o un bloqueador de anuncios/extensión interfiere	Revisa la conexión o desactiva extensiones
"La respuesta no tuvo el formato esperado"	Gemini no devolvió las 10 tarjetas en HTML	Pulsa de nuevo el botón para regenerar
El diseño se ve sin estilos	El CDN de Tailwind no cargó	Revisa tu conexión a internet
---
6. Notas
Tailwind por CDN es ideal para prototipos y sitios ligeros. Para un uso intensivo en producción, conviene compilar Tailwind y servir el CSS propio.
El resultado depende del modelo: revisa siempre las ideas y adáptalas a tu oferta y a las políticas de publicidad de Meta antes de usarlas.
