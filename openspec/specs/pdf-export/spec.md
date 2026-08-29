# Especificación: pdf-export

## Propósito

Exportación offline a PDF usando html2pdf.bundle.min.js vendored localmente. Genera archivo con nombre automático "Nota_<Asunto>_<NombreApellido>.pdf" editable, configurado para página A4, márgenes adecuados y renderizado fiel de la vista previa.

## Requisitos

### Requirement: Generación de PDF con html2pdf local

El sistema **DEBE** usar la librería html2pdf.bundle.min.js ubicada en `lib/html2pdf.bundle.min.js` (vendored, sin CDN) para generar el PDF completamente offline.

#### Scenario: Librería cargada desde archivo local

- DADO que la aplicación se abre via file://
- CUANDO se carga index.html
- ENTONCES html2pdf está disponible globalmente desde `lib/html2pdf.bundle.min.js`
- Y no se realizan peticiones de red para cargar la librería

#### Scenario: Error si librería no disponible

- DADO que `lib/html2pdf.bundle.min.js` falta o no carga
- CUANDO la aplicación inicializa
- ENTONCES se loggea error en consola: "Error: La librería html2pdf no se cargó correctamente"
- Y los botones de descargar/compartir quedan deshabilitados

### Requirement: Configuración de salida PDF (A4, márgenes, calidad)

El sistema **DEBE** generar PDF con: página A4, orientación retrato, márgenes 20mm, escala 2x para calidad, fondo blanco, letter-rendering activado.

#### Scenario: Parámetros de generación PDF correctos

- DADO que se inicia la generación de PDF
- CUANDO se configura html2pdf
- ENTONCES las opciones incluyen:
  - margin: [20, 20, 20, 20] (mm)
  - image: { type: 'jpeg', quality: 0.98 }
  - html2canvas: { scale: 2, useCORS: true, logging: false, backgroundColor: '#ffffff', letterRendering: true, windowHeight: nota.scrollHeight, scrollY: 0 }
  - jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
  - pagebreak: { mode: ['avoid-all', 'css', 'legacy'] }

### Requirement: Nombre de archivo automático editable

El sistema **DEBE** generar nombre de archivo con patrón "Nota_<Asunto>_<NombreApellido>.pdf" donde espacios se convierten a guiones bajos, y permitir al usuario editarlo antes de descargar.

#### Scenario: Nombre automático con asunto y nombre

- DADO asunto "Inscripción Fuera de Término" y nombre "García, Juan"
- CUANDO se genera el PDF para descarga
- ENTONCES el nombre de archivo propuesto es "Nota_Inscripción_Fuera_de_Término_García,_Juan.pdf"

#### Scenario: Nombre con caracteres especiales sanitizados

- DADO asunto con acentos y nombre con comas/espacios
- CUANDO se genera el nombre de archivo
- ENTONCES los espacios se convierten a "_"
- Y los caracteres se mantienen (html2pdf maneja UTF-8 en nombre)

### Requirement: Clase CSS para renderizado PDF

El sistema **DEBE** aplicar clase `pdf-render` al contenedor de la nota durante la generación para quitar bordes, sombras y border-radius, y removerla al finalizar.

#### Scenario: Clase pdf-render aplicada durante generación

- DADO que se inicia generarPDF()
- CUANDO antes de llamar a html2pdf
- ENTONCES el elemento #nota tiene clase "pdf-render"
- Y la clase quita: border, border-radius, box-shadow

#### Scenario: Clase pdf-render removida al finalizar

- DADO que la generación terminó (éxito o error)
- CUANDO se ejecuta el bloque finally
- ENTONCES el elemento #nota NO tiene clase "pdf-render"

### Requirement: Botones deshabilitados durante generación

El sistema **DEBE** deshabilitar ambos botones (Descargar y Compartir) durante la generación y mostrar estado "Generando..." en el botón Compartir.

#### Scenario: Botones deshabilitados durante generación

- DADO que el usuario pulsa Descargar o Compartir
- CUANDO inicia la generación
- ENTONCES #btnDescargar.disabled = true
- Y #btnCompartir.disabled = true
- Y #btnCompartir.textContent = " Generando..."

#### Scenario: Botones restaurados al finalizar

- DADO que la generación terminó (éxito o error)
- CUANDO se ejecuta el bloque finally
- ENTONCES #btnDescargar.disabled = false
- Y #btnCompartir.disabled = false
- Y #btnCompartir.textContent = "🔗 Compartir"

### Requirement: Descarga directa de PDF

El sistema **DEBE** permitir descargar el PDF directamente al disco del usuario al pulsar "Descargar PDF".

#### Scenario: Descarga exitosa

- DADO que la validación pasa
- CUANDO el usuario pulsa "Descargar PDF"
- ENTONCES se genera el PDF y se dispara la descarga del navegador con el nombre automático
- Y el archivo se guarda en la carpeta de descargas del usuario

#### Scenario: Descarga fallida muestra error

- DADO que html2pdf lanza error (ej. memoria insuficiente, elemento no encontrado)
- CUANDO se intenta descargar
- ENTONCES se muestra alerta: "❌ Error al generar el PDF: [mensaje de error]"
- Y el error se loggea en consola
- Y los botones se restauran

### Requirement: Preparación de blob para compartir nativo

El sistema **DEBE** generar el PDF como Blob (outputPdf('blob')) cuando el flujo es "Compartir nativo" para pasarlo a Web Share API.

#### Scenario: Generación de blob para compartir

- DADO que el usuario elige "Compartir nativo" desde el modal
- CUANDO se genera el PDF
- ENTONCES se llama html2pdf()...outputPdf('blob')
- Y el blob resultante se almacena en variable para Web Share API
- Y NO se dispara descarga automática

### Requirement: Manejo de salto de página en firma

El sistema **DEBE** evitar que el bloque de firma se parta entre dos páginas usando CSS `page-break-inside: avoid`.

#### Scenario: Firma no se parte entre páginas

- DADO que la nota es larga y la firma caería al final de página
- CUANDO se genera el PDF
- ENTONCES el bloque .firma tiene `page-break-inside: avoid`
- Y html2pdf respeta la regla moviendo la firma completa a la siguiente página si no cabe

### Requirement: Viewport y CSS print-friendly

El sistema **DEBE** incluir meta viewport y CSS que asegure renderizado correcto en móviles y escritorio para el PDF.

#### Scenario: Meta viewport presente

- DADO el HTML generado
- CUANDO se inspecciona el head
- ENTONCES existe `<meta name="viewport" content="width=device-width, initial-scale=1.0">`

#### Scenario: CSS responsive para nota

- DADO el CSS de #nota
- CUANDO se renderiza en móvil (< 600px)
- ENTONCES #nota tiene padding reducido, font-size 12-13px, y min-width 320px con overflow-x auto en panel
- Y la firma tiene ancho de línea reducido (180-220px)

## No-funcionales

### Requirement: 100% Offline - Sin peticiones de red

El sistema **DEBE** funcionar completamente sin conexión a internet. Ninguna petición de red para librerías, fuentes, imágenes o APIs externas durante la generación de PDF.

#### Scenario: Generación PDF sin red

- DADO que el dispositivo está en modo avión (sin internet)
- CUANDO el usuario genera y descarga un PDF
- ENTONCES el PDF se genera y descarga correctamente
- Y no hay errores de red en consola

### Requirement: Sin módulos ES (type="module" prohibido)

El sistema **NO DEBE** usar `<script type="module">` ni imports/exports ES modules, ya que file:// bloquea CORS para módulos.

#### Scenario: Script principal sin type="module"

- DADO el index.html
- CUANDO se inspecciona la etiqueta script principal
- ENTONCES NO tiene atributo type="module"
- Y el código usa IIFE + "use strict"

### Requirement: IIFE + "use strict"

El sistema **DEBE** encapsular toda la lógica en una IIFE (Immediately Invoked Function Expression) con "use strict" para evitar contaminación del scope global.

#### Scenario: Código encapsulado en IIFE

- DADO el script principal
- CUANDO se inspecciona la estructura
- ENTONCES todo el código está dentro de `(function() { "use strict"; ... })();`
- Y no hay variables/funciones globales innecesarias (solo las estrictamente necesarias para handlers inline como onclick)