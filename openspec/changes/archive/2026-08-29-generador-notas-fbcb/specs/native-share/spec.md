# Especificación: native-share

## Propósito

Compartir nativo del sistema operativo mediante Web Share API (Level 2 con soporte para archivos). Incluye detección de disponibilidad, fallback graceful cuando no está soportado, y modal de opciones. Sin integración WhatsApp/email por URL (fuera de alcance v1 según propuesta).

## Requisitos

### Requirement: Detección de Web Share API

El sistema **DEBE** detectar si `navigator.share` y `navigator.canShare` están disponibles en el navegador actual.

#### Scenario: API disponible en navegador compatible

- DADO un navegador que soporta Web Share API Level 2 (Chrome/Edge Android, Safari iOS 15.4+, Firefox Android)
- CUANDO la aplicación carga
- ENTONCES `typeof navigator.share === 'function'` es true
- Y `typeof navigator.canShare === 'function'` es true

#### Scenario: API no disponible en navegador incompatible

- DADO un navegador sin soporte (Firefox Desktop, Safari macOS antiguo, Chrome Desktop sin flag)
- CUANDO la aplicación carga
- ENTONCES `navigator.share` es undefined o `navigator.canShare` es undefined

### Requirement: Verificación de capacidad para compartir archivos

El sistema **DEBE** usar `navigator.canShare({ files: [file] })` para verificar si el sistema puede compartir archivos PDF antes de intentar compartir.

#### Scenario: canShare retorna true para PDF

- DADO que navigator.share está disponible
- CUANDO se crea un File con type 'application/pdf' y se llama navigator.canShare({ files: [file] })
- ENTONCES retorna true en plataformas que soportan compartir archivos (Android, iOS 15.4+)

#### Scenario: canShare retorna false para PDF

- DADO que navigator.share está disponible pero sin soporte de archivos (ej. Chrome Desktop con flag pero sin target apps)
- CUANDO se llama navigator.canShare({ files: [file] })
- ENTONCES retorna false

### Requirement: Compartir nativo con archivo PDF

El sistema **DEBE** compartir el PDF generado como archivo usando `navigator.share({ title, text, files: [file] })` cuando la API está disponible y soporta archivos.

#### Scenario: Compartir nativo exitoso

- DADO que el usuario pulsa "Compartir nativo" en el modal
- CUANDO navigator.canShare({ files: [pdfFile] }) es true
- ENTONCES se llama navigator.share({
    title: "Nota FBCB: [Asunto]",
    text: "Nota generada desde la Facultad de Bioquímica y Ciencias Biológicas - UNL",
    files: [pdfFile]
  })
- Y se muestra el share sheet nativo del SO
- Y el usuario puede elegir app destino (WhatsApp, Telegram, Email, Drive, etc.)

#### Scenario: Compartir nativo cancelado por usuario

- DADO que se muestra el share sheet nativo
- CUANDO el usuario cancela (pulsa X o back)
- ENTONCES la promise de navigator.share rechaza con AbortError
- Y la aplicación maneja el error silenciosamente (no muestra alerta)
- Y los botones se restauran

#### Scenario: Error en compartir nativo

- DADO que navigator.share lanza error distinto a AbortError
- CUANDO se intenta compartir
- ENTONCES se loggea error en consola
- Y se muestra alerta: "Error al compartir: [mensaje]"
- Y los botones se restauran

### Requirement: Fallback cuando Web Share API no soporta archivos

El sistema **DEBE** caer a descarga directa del PDF cuando `navigator.share` existe pero `navigator.canShare({ files: [file] })` retorna false.

#### Scenario: Fallback a descarga cuando canShare es false

- DADO que navigator.share existe pero navigator.canShare({ files: [file] }) es false
- CUANDO el usuario elige "Compartir nativo"
- ENTONCES se muestra alerta: "Tu dispositivo no soporta compartir archivos. Se descargará el PDF."
- Y se ejecuta `pdfGenerate.save()` (descarga directa)
- Y NO se llama navigator.share

### Requirement: Fallback cuando Web Share API no existe

El sistema **DEBE** caer a descarga directa del PDF cuando `navigator.share` no está disponible.

#### Scenario: Fallback a descarga cuando API no existe

- DADO que navigator.share es undefined
- CUANDO el usuario elige "Compartir nativo"
- ENTONCES se muestra alerta: "Tu navegador no soporta compartir nativo. Se descargará el PDF."
- Y se ejecuta `pdfGenerate.save()` (descarga directa)

### Requirement: Modal de opciones de compartir

El sistema **DEBE** mostrar un modal con opciones de compartir al pulsar el botón "Compartir", validando primero el formulario.

#### Scenario: Modal se abre tras validación exitosa

- DADO que el formulario es válido
- CUANDO el usuario pulsa botón "Compartir" (btn-compartir)
- ENTONCES el modal #modalCompartir obtiene clase "active" (display: flex)
- Y el modal muestra opciones: "Compartir nativo", y botón "Cancelar"

#### Scenario: Modal NO se abre si validación falla

- DADO que el formulario tiene errores (DNI inválido, email inválido, etc.)
- CUANDO el usuario pulsa botón "Compartir"
- ENTONCES se muestra alerta de validación
- Y el modal NO se abre
- Y el foco va al campo inválido

#### Scenario: Cerrar modal con click en overlay

- DADO que el modal está abierto
- CUANDO el usuario hace click en el fondo oscuro (overlay)
- ENTONCES el modal se cierra (remueve clase "active")

#### Scenario: Cerrar modal con botón Cancelar

- DADO que el modal está abierto
- CUANDO el usuario pulsa botón "Cancelar"
- ENTONCES el modal se cierra

#### Scenario: NO cerrar modal con click en contenido

- DADO que el modal está abierto
- CUANDO el usuario hace click dentro del contenido blanco (.modal-content)
- ENTONCES el modal NO se cierra (event.stopPropagation())

### Requirement: Generación de PDF previa al compartir

El sistema **DEBE** generar el PDF (como blob) antes de intentar cualquier acción de compartir, reutilizando la lógica de generación.

#### Scenario: PDF generado como blob antes de compartir nativo

- DADO que el usuario elige "Compartir nativo"
- CUANDO se procesa la acción
- ENTONCES se llama generarPDF(false, true) → comparteNativo = true
- Y generarPDF produce blob via outputPdf('blob')
- Y el blob se pasa a navigator.share

### Requirement: Nombre de archivo en compartición

El sistema **DEBE** usar el mismo nombre de archivo automático "Nota_<Asunto>_<NombreApellido>.pdf" para el archivo compartido.

#### Scenario: Archivo compartido tiene nombre correcto

- DADO asunto "Extensión de Regularidad" y nombre "López, María"
- CUANDO se comparte nativamente
- ENTONCES el File pasado a navigator.share tiene name "Nota_Extensión_de_Regularidad_López,_María.pdf"
- Y type "application/pdf"

### Requirement: Sin integración WhatsApp Web / email por URL (no-goal v1)

El sistema **NO DEBE** incluir botones ni lógica para "WhatsApp Web" ni "Email por mailto:" con adjunto simulado. Estos estaban en la referencia AppME 2.0 pero son **fuera de alcance** según la propuesta.

#### Scenario: No hay botón WhatsApp Web en modal

- DADO el modal de compartir renderizado
- CUANDO se inspeccionan las opciones
- ENTONCES NO existe botón "WhatsApp Web"
- Y NO existe botón "Correo electrónico" con mailto

#### Scenario: No hay función compartirWhatsApp / compartirEmail

- DADO el código JavaScript
- CUANDO se busca funciones compartirWhatsApp o compartirEmail
- ENTONCES NO existen (fueron removidas de la referencia)

## No-funcionales

### Requirement: 100% Offline - Web Share API es API del navegador

El sistema **DEBE** funcionar offline; Web Share API no requiere red, es una API del navegador/SO que muestra el share sheet nativo.

#### Scenario: Compartir nativo funciona sin internet

- DADO dispositivo en modo avión
- CUANDO el usuario comparte nativamente un PDF
- ENTONCES el share sheet nativo aparece y permite guardar en apps locales (Files, Drive offline, etc.)
- Y no hay peticiones de red

### Requirement: Accesibilidad del modal

El sistema **DEBE** hacer el modal accesible: foco atrapado, aria-labels, tecla Escape para cerrar.

#### Scenario: Modal accesible

- DADO el modal abierto
- CUANDO el usuario navega con Tab
- ENTONCES el foco queda atrapado dentro del modal
- Y pulsar Escape cierra el modal
- Y los botones tienen labels descriptivos

### Requirement: IIFE + "use strict" para lógica de compartir

El sistema **DEBE** encapsular la lógica de compartir en la IIFE principal con "use strict".

#### Scenario: Funciones de compartir en IIFE

- DADO el script principal
- CUANDO se inspeccionan funciones compartirNativo, mostrarOpcionesCompartir, cerrarModalCompartir
- ENTONCES están definidas dentro de la IIFE
- Y no contaminan scope global