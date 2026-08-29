# Generador de Notas — FBCB · UNL

Generador **100% offline** de notas formales para la Facultad de Bioquímica y Ciencias Biológicas
(FBCB) de la Universidad Nacional del Litoral (UNL). Funciona sin conexión a internet, sin
instalación y sin servidores: un solo archivo HTML que abre con doble clic.

El formato de las notas replica la estructura oficial: **fecha → destinatario → Ref.: asunto → cuerpo
→ firma**.

---

## Cómo usar

### 1. Abrir la aplicación

1. Descargue o copie la carpeta completa del proyecto (no mueva `index.html` fuera de la carpeta:
   necesita la librería en `lib/`).
2. Haga **doble clic** en `index.html`. Se abre en el navegador. **No hace falta internet.**

> En algunos navegadores aparece una advertencia de "archivo no seguro" al abrir archivos locales:
> es normal y puede ignorarse. Chrome y Edge funcionan sin problema.

### 2. Generar una nota

1. **Perfil del remitente**: seleccione Alumno/a, Docente, Personal u Otro. Esto ajusta la firma.
2. **Destinatario**: elija la autoridad (10 de FBCB y 11 de Rectorado UNL, nómina 2026-2030).
3. **Asunto**: elija uno de los 6 tipos de nota. Aparecen los campos específicos de cada uno.
4. **Datos del remitente**: nombre completo, DNI, email (obligatorios) y teléfono (opcional).
5. **Vista previa**: se genera al instante a la derecha. **Todo el texto es editable** haciendo clic
   en los campos resaltados al pasar el cursor (fecha, destinatario, referencia, cuerpo y firma).
6. **Nombre del archivo PDF**: se autogenera como `Nota_Asunto_Nombre.pdf`; puede cambiarlo.
7. Pulse **Descargar PDF** o **Compartir** (en el celular abre la hoja de compartir del sistema:
   WhatsApp, Telegram, email, etc.).

### 3. Persistencia

Los datos del remitente quedan guardados en el navegador (localStorage) y se restauran la próxima
vez que abra `index.html` en el mismo navegador. Si los borra del navegador, se pierden — la
aplicación no guarda datos en ningún servidor.

---

## Preguntas frecuentes

### ¿El botón Compartir no hace nada / descarga directo en la PC?

Es esperado. La hoja de compartir del sistema (WhatsApp, Telegram, email) es una función **de
celular**. En computadora, el botón Compartir descarga el PDF directamente como alternativa.

### ¿El PDF sale cortado o con márgenes raros?

La configuración está optimizada para hoja **A4** con márgenes de 20 mm. Si el cuerpo de la nota es
muy largo, el PDF se reparte en varias páginas automáticamente. Evite romper el bloque de la firma
manualmente.

### ¿Puedo escribir un asunto distinto a los 6 ofrecidos?

La vista previa permite editar el texto del cuerpo libremente, así que puede adaptar cualquier nota.
La lista de asuntos y autoridades está fijada para garantizar el formato correcto en la primera
versión.

### Aparece "Error: la librería de PDF no se cargó correctamente"

La carpeta `lib/` falta o el archivo `lib/html2pdf.bundle.min.js` no está junto a `index.html`.
Vuelva a descargar la carpeta completa.

---

## Para desarrolladores

- **Stack**: HTML + CSS + JavaScript vanilla (sin frameworks, sin build, sin dependencias remotas).
- **Estructura**: un solo `index.html` — todo el código está dentro, en una IIFE con `"use strict"`,
  separando lógica pura (validación, formateo, generación de texto) del manejo del DOM.
- **PDF**: `lib/html2pdf.bundle.min.js` (v0.10.1) vendored local, A4 portrait, margen 20 mm, scale 2.
- **Persistencia**: `localStorage` bajo la clave `fbcb-sender-profile` (schema `{v:1}`, validado al leer).
- **Compartir**: Web Share API (Level 2, archivos) con fallback a descarga directa.
- **Verificación**: abrir con doble clic en `file://`; probar en escritorio (Chrome/Firefox/Edge/Safari)
  y en celular (Chrome Android / Safari iOS). No hay suite de tests automatizada en v1.