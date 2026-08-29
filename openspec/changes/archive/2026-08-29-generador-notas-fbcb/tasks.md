# Tasks: Generador de Notas Formales FBCB-UNL

## Review Workload Forecast

| Field | Value |
|-------|-------|
| Estimated changed lines | 1300–1600 (index.html) + ~3000 vendored (lib/html2pdf.bundle.min.js) |
| 400-line budget risk | High |
| Chained PRs recommended | Yes |
| Suggested split | PR 1: HTML/CSS/Constants → PR 2: Pure Logic → PR 3: UI Controllers/Persistence → PR 4: PDF/Share/Accessibility |
| Delivery strategy | ask-on-risk |
| Chain strategy | feature-branch-chain |

Decision needed before apply: Yes
Chained PRs recommended: Yes
Chain strategy: feature-branch-chain
400-line budget risk: High

### Suggested Work Units

| Unit | Goal | Likely PR | Focused test command | Runtime harness | Rollback boundary |
|------|------|-----------|----------------------|-----------------|-------------------|
| 1 | HTML skeleton, CSS variables, responsive grid, semantic structure, modal, preview container | PR 1 | `open index.html` → verify structure, no JS errors, mobile <600px responsive | file:// double-click, DevTools device toolbar | Revert index.html to empty; lib/ untouched |
| 2 | Constants module: AUTHORITIES[21], ASUNTOS_CONFIG[6], MESES, regex | PR 1 | Console: `AUTHORITIES.length===21`, `Object.keys(ASUNTOS_CONFIG).length===6` | file:// open, DevTools console | Revert constants section only |
| 3 | Pure logic: validators, formatDate, formatSubjects, parrafoMotivo, parrafoDocumentacion, authority lookup, generateCuerpo | PR 2 | Console unit tests per spec scenarios (formatDate, validators, generators) | file:// open, DevTools console | Revert pure logic section only |
| 4 | Persistence: loadProfile/saveProfile/clearProfile v1 with validation | PR 3 | Console: `saveProfile()`, reload → `loadProfile()` restores; corrupt JSON ignored | file:// open, DevTools Application > localStorage | Clear localStorage key; revert persistence section |
| 5 | UI Controllers: renderAuthoritySelector, renderSubjectSelector, renderDynamicFields, collectFormData, renderPreview, syncPreviewEdits | PR 3 | Fill form → preview updates live; switch subject → dynamic fields change | file:// open, interact UI | Revert UI controllers section |
| 6 | Event wiring: form inputs → actualizar(), subject change → dynamic fields, buttons → export flow | PR 4 | Full flow: authority+subject+fields+sender → preview complete → buttons enabled | file:// open, full manual flow | Revert event wiring section |
| 7 | PDF generation: generarPDF(save/share), html2pdf options, pdf-render toggle, blob vs save, filename | PR 4 | Click Descargar → PDF downloads with correct name/content; verify A4/20mm/scale2 | file:// open, verify Downloads folder | Revert PDF generation section |
| 8 | Native share: detect API, canShare check, compartirNativo, modal open/close, focus trap, Escape, fallback | PR 4 | Mobile: share sheet opens; Desktop: fallback alert → download; cancel → silent | Chrome Android / Safari iOS / Chrome Desktop | Revert share section; modal CSS |
| 9 | Accessibility polish: labels, aria-live errors, focus management, viewport meta, print-friendly CSS | PR 4 | Lighthouse/axe: no a11y errors; Tab navigation works; Escape closes modal | file:// open, keyboard only, screen reader | Revert a11y/print CSS additions |
| 10 | Manual verification: all 6 subjects, 21 authorities, offline file://, localStorage persistence, mobile share | PR 4 | Checklist per spec scenarios; file:// double-click; airplane mode test | file://, mobile devices | N/A — final verification |

## Phase 1: Foundation — HTML Skeleton, CSS, Constants

- [x] 1.1 Create `index.html` with DOCTYPE, html[lang="es"], head: charset=utf-8, viewport meta, title "Generador de Notas FBCB-UNL", no external resources
- [x] 1.2 Add embedded `<style>` block: CSS custom properties (colors, spacing, fonts), responsive grid layout (panel formulario left, panel vista-previa right, stacks <600px), #nota A4 print area, .pdf-render class (removes border/radius/shadow), .firma {page-break-inside:avoid}, modal styles (#modalCompartir overlay + .modal-content), focus-visible outlines, aria-live region for errors
- [x] 1.3 Add semantic HTML body structure: `<header>` with h1, `<main>` with two panels: `<section id="panel-formulario">` containing `<form id="form-nota">` (fieldset remitente tipo, inputs nombre/dni/email/telefono, select autoridad, select asunto, div#campos-dinamicos, buttons), `<section id="panel-vista-previa">` with `<article id="nota">` containing editable sections (#v-fecha, #v-destinatario, #v-asunto, #v-contenido, #v-firma), `<footer>` with actions (btnDescargar, btnCompartir)
- [x] 1.4 Add modal #modalCompartir (role="dialog", aria-modal="true", aria-labelledby) with options: button#opcionCompartirNativo, button#opcionCancelar; overlay click closes, content click stops propagation
- [x] 1.5 Add script tag for vendored lib: `<script src="lib/html2pdf.bundle.min.js"></script>` before main IIFE
- [x] 1.6 Create `lib/` directory; download html2pdf.bundle.min.js v0.10.1 from official release (https://github.com/eKoopmans/html2pdf.js/releases/tag/v0.10.1) and save as `lib/html2pdf.bundle.min.js` (verify file exists, non-zero, loads without network)

## Phase 2: Constants & Pure Logic Modules (inside IIFE)

- [x] 2.1 Define `AUTHORITIES` const array with 21 objects per design.md lines 118–141: 10 FBCB + 11 Rectorado UNL, each with id, nombre, cargo, tratamiento, ambito; verify gender-cargo consistency (Señor/Señora)
- [x] 2.2 Define `MESES` const array with 12 Spanish month names for formatDate
- [x] 2.3 Define regex constants: `DNI_RE = /^\d{7,8}$/`, `EMAIL_RE = /^[^\s@]+@[^\s@]+\.[^\s@]+$/`, `PHONE_RE = /^[\d\s\-+()]{7,15}$/`
- [x] 2.4 Define `ASUNTOS_CONFIG` const object with 6 keys per design.md lines 146–182 and spec note-generator scenarios: each with obligatorios[], campos[] (id, label, tipo, placeholder, obligatorio), conDocumentacion? (boolean), generar(datos, perfil) function returning string
- [x] 2.5 Implement pure validators: `validarDNI(dni)` → boolean, `validarEmail(email)` → boolean, `validarTelefono(tel)` → boolean (empty passes), `validarPerfil(perfil)` → {valido, errores[]}
- [x] 2.6 Implement `formatDate(isoString)` → "DD de MMMM de YYYY" or "[fecha]"; handle invalid/empty
- [x] 2.7 Implement `formatSubjects(multilineText)` → string with "• " prefix per non-empty line, trim, ignore blank lines
- [x] 2.8 Implement `parrafoMotivo(motivo)` → string or "" per spec (conditional inclusion)
- [x] 2.9 Implement `parrafoDocumentacion(conDocumentacion)` → string or "" per spec
- [x] 2.10 Implement authority lookup: `buscarAutoridadPorNombre(nombre)` → authority object or undefined; `getAutoridadesPorAmbito(ambito)` → filtered array
- [x] 2.11 Implement `generarCuerpo(asuntoKey, datos, perfil)` → string: delegates to ASUNTOS_CONFIG[asuntoKey].generar(datos, perfil), uses formatSubjects, parrafoMotivo, parrafoDocumentacion
- [x] 2.12 Implement `renderDestinatarioHTML(authority)` → HTML string per spec format (tratamiento + facultad + strong nombre + S/D + small facultad-UNL)

## Phase 3: Persistence & UI Controllers

- [x] 3.1 Implement `loadProfile()` → reads localStorage["fbcb-sender-profile"], parses JSON, validates shape {v:1, tipo, nombre, dni, email, telefono}, returns profile object or null (ignores corrupt/invalid without throwing)
- [x] 3.2 Implement `saveProfile(profile)` → writes JSON to localStorage["fbcb-sender-profile"] with v:1; called on input/blur of sender fields
- [ ] 3.3 Implement `clearProfile()` → removes localStorage key — NO implementada intencionalmente (no-goal v1, sin requisito de spec dependiente; ver SUGGESTION S10 del verify-report round 2)
- [x] 3.4 Implement `renderAuthoritySelector()` → populates #selectAutoridad with default option + 21 options grouped by ambito (optgroup FBCB then Rectorado UNL), label "Nombre — Cargo"
- [x] 3.5 Implement `renderSubjectSelector()` → populates #selectAsunto with default + 6 options from ASUNTOS_CONFIG keys
- [x] 3.6 Implement `renderDynamicFields(asuntoKey)` → clears #campos-dinamicos, creates label+input/textarea per campos[] config, adds required attribute, attaches input listeners to trigger actualizar()
- [x] 3.7 Implement `collectFormData()` → reads all form fields, returns {perfil, nombre, dni, email, telefono, autoridadId, asuntoKey, dynamicFields:object}; validates required fields, returns {valido, errores[], data}
- [x] 3.8 Implement `renderPreview(noteState)` → populates #v-fecha, #v-destinatario (innerHTML), #v-asunto (textContent), #v-contenido (textContent with white-space:pre-wrap), #v-firma lines; attaches contenteditable="true" to each section, adds blur/input listeners to sync edits to exportState
- [x] 3.9 Implement `syncPreviewEdits()` → on blur/input of editable sections, updates exportState (fecha, destinatarioHtml, asunto, cuerpo, firma lines, filename)
- [x] 3.10 Implement `actualizar()` → main UI sync: collectFormData → if valid generate noteState via pure logic → renderPreview → update exportState filename; if invalid show aria-live error

## Phase 4: Event Wiring, PDF Export, Native Share

- [x] 4.1 Wire form events: input/change on sender fields → saveProfile + actualizar(); change on #selectAutoridad/#selectAsunto → actualizar() (subject change also calls renderDynamicFields); input on dynamic fields → actualizar()
- [x] 4.2 Wire #btnDescargar click → validate profile → generarPDF({save:true, share:false})
- [x] 4.3 Wire #btnCompartir click → validate profile → if valid show modal (#modalCompartir.classList.add("active")), focus first button; if invalid show error + focus field
- [x] 4.4 Wire modal: #opcionCompartirNativo click → generarPDF({save:false, share:true}) → cerrarModalCompartir(); #opcionCancelar/overlay click → cerrarModalCompartir(); Escape keydown → cerrarModalCompartir(); focus trap on Tab within modal
- [x] 4.5 Implement `generarPDF({save, share})` → disable buttons + "Generando..." text; #nota.classList.add("pdf-render"); configure html2pdf per spec (margin 20mm, scale 2, A4 portrait, letterRendering, pagebreak avoid-all, backgroundColor white, image jpeg 0.98); if share: .outputPdf('blob').then(blob → compartirNativo(blob)); if save: .save(filename); finally: remove pdf-render class, restore buttons
- [x] 4.6 Implement `compartirNativo(blob)` → create File from blob with name exportState.filename, type application/pdf; if navigator.share && navigator.canShare({files:[file]}): navigator.share({title, text, files:[file]}).catch(err → if AbortError silent else alert+log); else if navigator.share: alert fallback → pdfGenerate.save(); else: alert fallback → pdfGenerate.save()
- [x] 4.7 Implement filename generation: `const filename = 'Nota_' + asuntoKey.replace(/\s+/g, '_') + '_' + nombre.replace(/\s+/g, '_') + '.pdf'` (keep accents/UTF-8 per design)

## Phase 5: Initialization, Accessibility, Verification

- [x] 5.1 IIFE initialization: on DOMContentLoaded → loadProfile() → populate sender fields + tipo selector → set #v-fecha to formatDate(today) → renderAuthoritySelector() → renderSubjectSelector() → actualizar() (renders placeholders per spec)
- [x] 5.2 Add aria-live="polite" region for validation errors; ensure all inputs have <label for>, fieldset/legend for tipo remitente, select has aria-label, buttons have accessible names
- [x] 5.3 Verify focus management: modal open traps focus, Escape closes, btnCompartir returns focus on close; form invalid focuses first invalid field
- [x] 5.4 Verify responsive CSS: <600px panels stack, #nota padding/font-size adjust, .firma line-width 180-220px, panel overflow-x:auto
- [x] 5.5 Create `INSTRUCCIONES.md` in project root: purpose, doble clic index.html, 100% offline, perfiles/autoridades/asuntos, vista previa editable, descargar PDF, compartir nativo, localStorage persistencia, sin build/dependencias externas
- [x] 5.6 Manual verification checklist: all 6 subjects generate correct body + dynamic fields; 21 authorities render correct destinatario block; sender types adapt body/firma; placeholders when empty; localStorage persists across reload; corrupt localStorage ignored; PDF downloads with auto-name, A4/20mm/scale2, firma not split; share native works mobile, fallback desktop; file:// double-click works offline; no console errors; no network requests