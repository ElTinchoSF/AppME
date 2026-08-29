# Propuesta: Generador de Notas Formales FBCB-UNL

## Propósito

Generador offline de notas formales para la Facultad de Bioquímica y Ciencias Biológicas (FBCB-UNL) que replica el formato de "AppME 2.0": fecha → destinatario → "Ref.: [asunto]" → cuerpo → firma. Resuelve la necesidad de docentes, alumnos y personal de emitir notas institucionales válidas sin depender de red, backends o instalaciones.

## Alcance

### En alcance
- Perfil remitente: Alumno / Docente / Personal / Otro (afecta firma y redacción)
- Datos remitente: nombre, DNI (7-8 dígitos), email (válido), teléfono (opcional)
- 21 autoridades oficiales 2026-2030 (10 FBCB + 11 Rectorado UNL) con tratamiento derivado del cargo
- 6 asuntos con plantillas y campos dinámicos: Inscripción Fuera de Término, Extensión de Regularidad, Solicitud de Licencia, Inscripción a Concurso Docente, Inscripción a Concurso Alumno, Solicitud de Plan de Estudios y Programas
- Vista previa en vivo editable (fecha/destinatario/Ref/cuerpo/firma)
- localStorage para persistencia de datos del remitente
- Exportación PDF (html2pdf local) + Web Share API nativo
- Nombre archivo auto: "Nota_<Asunto>_<NombreApellido>.pdf"

### Fuera de alcance (no-goals v1)
- Nuevos asuntos más allá de los 6
- Backend, sincronización, login, PWA/service worker
- Embeber fuentes (fuentes del sistema)

## Capacidades

### Nuevas capacidades
- `note-generator`: motor de generación de notas formales con plantillas por asunto
- `authority-registry`: catálogo de 21 autoridades FBCB/UNL con tratamientos formales
- `sender-profile`: gestión de perfil remitente (tipo, datos, validación, localStorage)
- `pdf-export`: exportación offline a PDF vía html2pdf vendored
- `native-share`: compartir nativo del sistema (Web Share API)

### Capacidades modificadas
- Ninguna (proyecto nuevo, sin specs previas)

## Enfoque técnico

Single-file HTML (index.html) con IIFE + "use strict". Separación estricta: lógica pura (validación, formateo fecha, generación texto, plantillas) en módulos internos; manipulación DOM en controladores UI. Datos declarativos (autoridades, asuntos, plantillas) como constantes. html2pdf.bundle.min.js en lib/. Sin build, sin módulos ES (file:// CORS), 100% offline.

## Áreas afectadas

| Área | Impacto | Descripción |
|------|---------|-------------|
| `index.html` | Nuevo | Aplicación completa (HTML semántico, CSS, JS IIFE) |
| `lib/html2pdf.bundle.min.js` | Nuevo | Librería vendored para PDF |

## Riesgos

| Riesgo | Probabilidad | Mitigación |
|--------|--------------|------------|
| Web Share API no disponible en algunos navegadores | Media | Fallback: solo botón Descargar PDF; detectar `navigator.share` |
| Validación DNI/email demasiado estricta | Baja | Regex permisivas, permitir corrección manual en vista previa |
| html2pdf renderizado incorrecto en móviles | Baja | Probar viewport meta, CSS print-friendly, página A4 |

## Plan de rollback

Eliminar `index.html` y `lib/html2pdf.bundle.min.js`. Sin migraciones, sin base de datos, sin estado server-side. localStorage se limpia manualmente o expira.

## Dependencias

- html2pdf.bundle.min.js (vendored local, sin CDN)
- Web Share API (navegador, opcional)

## Criterios de éxito

- [ ] Abre con doble clic en file:// y funciona 100% offline
- [ ] Genera 6 tipos de nota con campos dinámicos correctos
- [ ] 21 autoridades con tratamiento formal correcto (género-cargo)
- [ ] Vista previa editable antes de exportar
- [ ] PDF descargable con nombre auto-generado editable
- [ ] Compartir nativo funciona donde está disponible
- [ ] localStorage persiste datos remitente entre sesiones
- [ ] Código en IIFE + "use strict", lógica pura separada del DOM
- [ ] HTML semántico, accesible (labels, foco, estados)