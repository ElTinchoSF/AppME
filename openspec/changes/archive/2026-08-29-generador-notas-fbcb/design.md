# Design: Generador de Notas Formales FBCB-UNL

## Technical Approach

Single-file HTML application (`index.html`) containing semantic HTML, CSS, and vanilla JS encapsulated in an IIFE with `"use strict"`. Strict separation: **pure logic modules** (constants, validators, formatters, generators) vs **UI controllers** (DOM manipulation, event handling, state synchronization). All data (authorities, subject templates, sender profile) are declarative constants. No build step, no ES modules (file:// CORS), no external dependencies — `html2pdf.bundle.min.js` vendored in `lib/`. 100% offline: opens via double-click on `file://`.

## Architecture Decisions

### Decision: Single-file IIFE architecture

**Choice**: One `index.html` with embedded CSS + JS in IIFE  
**Alternatives considered**: Multi-file with ES modules, separate CSS/JS files  
**Rationale**: Requirement: double-click to open, 100% offline, no build. ES modules blocked by file:// CORS. IIFE avoids global pollution while keeping deployment trivial (one file + lib/).

### Decision: Pure logic vs DOM controllers separation

**Choice**: Internal modules for pure functions (validation, formatting, generation, templates); controllers for UI sync  
**Alternatives considered**: Mixed logic in event handlers, class-based components  
**Rationale**: Testability of pure logic without DOM; predictable data flow; matches proposal's "lógica pura vs DOM" requirement. Functions are referentially transparent — easy to unit-test manually in console.

### Decision: 21 authorities as declarative constant array

**Choice**: `AUTHORITIES` array of objects with `{id, nombre, cargo, tratamiento, ambito}`  
**Alternatives considered**: External JSON, derived from cargo at runtime  
**Rationale**: Treatment must be gender-cargo consistent (spec requirement). Pre-computed treatment eliminates runtime derivation bugs. Array enables `.filter()`, `.find()`, `.map()` for UI rendering and search. 10 FBCB + 11 Rectorado UNL from official 2026-2030 roster.

### Decision: `asuntosConfig` with generator functions + dynamic field configs

**Choice**: Object mapping subject key → `{obligatorios, campos[], conDocumentacion?, generar(datos)}`  
**Alternatives considered**: String templates with placeholder replacement, separate template files  
**Rationale**: Generator functions allow conditional paragraphs (motivo, documentación), bullet formatting, and per-subject logic. `campos[]` drives dynamic form rendering declaratively. `conDocumentacion` flag controls doc paragraph inclusion (spec requirement). Adaptable to sender profile via generator parameters.

### Decision: Sender profile with localStorage schema + versioning

**Choice**: Key `fbcb-sender-profile`, shape `{v:1, tipo, nombre, dni, email, telefono}`, validate on read  
**Alternatives considered**: No versioning, IndexedDB, cookies  
**Rationale**: localStorage is synchronous, works file://, survives sessions. Version field enables future migration. Validation on read handles corrupted data gracefully (spec: ignore corrupt, no JS error). Auto-save on `input`/`blur`.

### Decision: Live preview with contenteditable fallback

**Choice**: Read-only preview by default; each section gets `contenteditable="true"` on focus via CSS `:focus-within` or JS toggle  
**Alternatives considered**: Separate edit mode, textarea mirror  
**Rationale**: Spec requires editable preview before export. `contenteditable` preserves formatting; simpler than syncing textarea. Changes captured on `blur`/`input` to update export state.

### Decision: html2pdf configuration for print fidelity

**Choice**: A4 portrait, 20mm margins, scale 2, letter-rendering, `page-break-inside: avoid` on `.firma`, `pdf-render` class to strip UI chrome  
**Alternatives considered**: jsPDF direct, server-side generation  
**Rationale**: Spec mandates these exact options. `pdf-render` class applied/removed around generation ensures preview stays styled while PDF is clean. Blob output for Web Share, `save()` for download.

### Decision: Web Share API with progressive fallback

**Choice**: Detect `navigator.share` + `navigator.canShare({files:[file]})`; fallback to direct download  
**Alternatives considered**: mailto with attachment (impossible), WhatsApp Web deep link (no-goal v1)  
**Rationale**: Spec mandates native share sheet where available, graceful fallback. Level 2 (files) only on mobile; desktop falls back. AbortError on cancel handled silently.

### Decision: Accessibility-first semantic HTML

**Choice**: `<form>`, `<label for>`, `<select>`, `<fieldset>/<legend>` for sender type, `aria-live` on errors, focus management in modal, Escape to close  
**Alternatives considered**: ARIA-only on divs, custom components  
**Rationale**: Spec requires labels, focus, aria states, mobile responsive. Native form elements provide built-in a11y; less code, better screen reader support.

## Data Flow

```
User Input (Form)
       │
       ▼
┌─────────────────────────────────────┐
│  Controller: collectFormData()      │  ──► validates sender profile
│  - sender profile                   │       (DNI regex, email regex, phone)
│  - selected authority               │
│  - selected subject                 │
│  - dynamic fields                   │
└─────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────┐
│  Pure Logic: generateNote()         │  ──► calls authority lookup
│  - formatDate(fecha)                │       subject generator(datos)
│  - renderDestinatario(authority)    │       formatSubjects(list)
│  - generateCuerpo(subject, datos)   │
│  - renderFirma(senderProfile)       │
└─────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────┐
│  Controller: renderPreview(note)    │  ──► updates #v-fecha, #v-destinatario,
│  - populates preview DOM            │       #v-asunto, #v-contenido, #v-firma
│  - attaches contenteditable         │
└─────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────┐
│  User edits preview (contenteditable)│
│  - on blur: update exportState       │
└─────────────────────────────────────┘
       │
       ▼
Export Actions
    ├─► Descargar PDF: generarPDF(save=true) ──► html2pdf().save()
    └─► Compartir: generarPDF(blob=true) ──► navigator.share({files:[blob]})
                                         └─► fallback: save()
```

## File Changes

| File | Action | Description |
|------|--------|-------------|
| `index.html` | Create | Complete application: semantic HTML structure, CSS variables + responsive grid, JS IIFE with modules (AUTHORITIES, ASUNTOS_CONFIG, validators, formatters, generators, controllers, persistence, PDF/share) |
| `lib/html2pdf.bundle.min.js` | Create | Vendored html2pdf v0.10.1 bundle (local, no CDN). Downloaded from official release. |

## Interfaces / Contracts

### Authority Schema
```js
// 21 entries — FBCB (10) + Rectorado UNL (11)
const AUTHORITIES = [
  { id: 'decano-fbcb', nombre: 'Dr. Guillermo Ramos', cargo: 'Decano', tratamiento: 'Señor Decano', ambito: 'FBCB' },
  { id: 'vicedecana-fbcb', nombre: 'Esp. Cecilia Serra', cargo: 'Vicedecana', tratamiento: 'Señora Vicedecana', ambito: 'FBCB' },
  { id: 'sec-adm-fbcb', nombre: 'Téc. Adriana Vidaechea', cargo: 'Secretaria Administrativa', tratamiento: 'Señora Secretaria Administrativa', ambito: 'FBCB' },
  { id: 'sec-acad-fbcb', nombre: 'Dra. María Fernanda Simoniello', cargo: 'Secretaria Académica', tratamiento: 'Señora Secretaria Académica', ambito: 'FBCB' },
  { id: 'dir-est-fbcb', nombre: 'Lic. Rosario Paulini', cargo: 'Directora de Asuntos Estudiantiles', tratamiento: 'Señora Directora de Asuntos Estudiantiles', ambito: 'FBCB' },
  { id: 'sec-cyt-fbcb', nombre: 'Dr. Guillermo Sihufe', cargo: 'Secretario de Ciencia y Técnica', tratamiento: 'Señor Secretario de Ciencia y Técnica', ambito: 'FBCB' },
  { id: 'sec-des-fbcb', nombre: 'CPN María Victoria Luque', cargo: 'Secretaria de Desarrollo Económico', tratamiento: 'Señora Secretaria de Desarrollo Económico', ambito: 'FBCB' },
  { id: 'sec-ext-fbcb', nombre: 'Dra. Sandra Ravelli', cargo: 'Secretaria de Extensión Social', tratamiento: 'Señora Secretaria de Extensión Social', ambito: 'FBCB' },
  { id: 'sec-pos-fbcb', nombre: 'Dra. Gabriela Micheloud', cargo: 'Secretaria de Posgrado', tratamiento: 'Señora Secretaria de Posgrado', ambito: 'FBCB' },
  { id: 'dir-ess-fbcb', nombre: 'Mg. Germán Boero', cargo: 'Director de la Escuela Superior de Sanidad "Dr. Ramón Carrillo"', tratamiento: 'Señor Director de la Escuela Superior de Sanidad', ambito: 'FBCB' },
  // Rectorado UNL (11)
  { id: 'rectora-unl', nombre: 'Prof. Laura Tarabella', cargo: 'Rectora', tratamiento: 'Señora Rectora', ambito: 'Rectorado UNL' },
  { id: 'vicerrectora-unl', nombre: 'CPN Liliana Dillon', cargo: 'Vicerrectora', tratamiento: 'Señora Vicerrectora', ambito: 'Rectorado UNL' },
  { id: 'sec-gen-unl', nombre: 'Arq. Sergio Cosentino', cargo: 'Secretario General', tratamiento: 'Señor Secretario General', ambito: 'Rectorado UNL' },
  { id: 'sec-acad-inov-unl', nombre: 'Lic. María Bárbara Mántaras', cargo: 'Secretaria Académica e Innovación Educativa', tratamiento: 'Señora Secretaria Académica e Innovación Educativa', ambito: 'Rectorado UNL' },
  { id: 'sec-cat-unl', nombre: 'Dra. Larisa Carrera', cargo: 'Secretaria de Ciencia, Arte y Tecnología', tratamiento: 'Señora Secretaria de Ciencia, Arte y Tecnología', ambito: 'Rectorado UNL' },
  { id: 'sec-ext-unl', nombre: 'Ana Indus. María Lucrecia Wilson', cargo: 'Secretaria de Extensión Universitaria', tratamiento: 'Señora Secretaria de Extensión Universitaria', ambito: 'Rectorado UNL' },
  { id: 'sec-vti-unl', nombre: 'Dr. Javier Lottersberger', cargo: 'Secretario de Vinculación Tecnológica e Innovación', tratamiento: 'Señor Secretario de Vinculación Tecnológica e Innovación', ambito: 'Rectorado UNL' },
  { id: 'sec-ft-unl', nombre: 'Arq. Sara Lauría', cargo: 'Secretaria de Fortalecimiento Territorial', tratamiento: 'Señora Secretaria de Fortalecimiento Territorial', ambito: 'Rectorado UNL' },
  { id: 'sec-bien-unl', nombre: 'Ing. Gustavo Menéndez', cargo: 'Secretario de Bienestar Universitario', tratamiento: 'Señor Secretario de Bienestar Universitario', ambito: 'Rectorado UNL' },
  { id: 'sec-cul-unl', nombre: 'Abog. María Lucila Reyna', cargo: 'Secretaria de Cultura y Medios', tratamiento: 'Señora Secretaria de Cultura y Medios', ambito: 'Rectorado UNL' },
  { id: 'sec-adm-pre-unl', nombre: 'CPN Germán Bonino', cargo: 'Secretario de Administración y Gestión Presupuestaria', tratamiento: 'Señor Secretario de Administración y Gestión Presupuestaria', ambito: 'Rectorado UNL' },
];
```

### Subject Config Schema (adapted from AppME 2.0)
```js
const ASUNTOS_CONFIG = {
  'Inscripción Fuera de Término': {
    obligatorios: ['carrera', 'materias'],
    campos: [
      { id: 'carrera', label: 'Carrera', tipo: 'text', placeholder: 'Ej: Licenciatura en Bioquímica', obligatorio: true },
      { id: 'materias', label: 'Asignaturas', tipo: 'textarea', placeholder: 'Liste las asignaturas (una por línea)', obligatorio: true },
      { id: 'motivo', label: 'Motivo de la solicitud (opcional)', tipo: 'textarea', placeholder: 'Explique brevemente los motivos...' }
    ],
    generar: (d, perfil) => `Me dirijo a usted a fin de solicitar ${perfil === 'Alumno' ? 'mi' : 'mi'} inscripción fuera de término...`
  },
  'Extensión de Regularidad': { ... },
  'Solicitud de Licencia': {
    obligatorios: ['tipoLicencia', 'desde', 'hasta'],
    campos: [
      { id: 'tipoLicencia', label: 'Tipo de licencia', tipo: 'text', placeholder: 'Ej: Licencia ordinaria / Estudio / Médica', obligatorio: true },
      { id: 'desde', label: 'Fecha desde', tipo: 'date', obligatorio: true },
      { id: 'hasta', label: 'Fecha hasta', tipo: 'date', obligatorio: true },
      { id: 'motivo', label: 'Motivo (opcional)', tipo: 'textarea', placeholder: '...' }
    ],
    generar: (d, perfil) => `...entre el ${formatDate(d.desde)} y el ${formatDate(d.hasta)} inclusive...`
  },
  'Inscripción a Concurso Docente': {
    obligatorios: ['cargo', 'catedra', 'area'],
    conDocumentacion: true,
    campos: [ ... ],
    generar: (d, perfil) => `...Declaro cumplir con los requisitos...${d.conDocumentacion ? DOC_PARAGRAPH : ''}...`
  },
  'Inscripción a Concurso Alumno': { ... conDocumentacion: true ... },
  'Solicitud de Plan de Estudios y Programas': {
    obligatorios: ['carrera'],
    campos: [
      { id: 'carrera', label: 'Carrera', tipo: 'text', obligatorio: true },
      { id: 'materias', label: 'Asignaturas solicitadas (opcional)', tipo: 'textarea', placeholder: 'Si se omite, se solicitarán los planes de toda la carrera.' }
    ],
    generar: (d, perfil) => d.materias ? `...de las asignaturas...${formatSubjects(d.materias)}` : `...correspondientes a la totalidad del plan de estudios...`
  }
};
```

### Sender Profile Schema (localStorage)
```js
// Key: "fbcb-sender-profile"
// Shape v1:
{
  v: 1,
  tipo: 'Alumno' | 'Docente' | 'Personal' | 'Otro',
  nombre: 'Apellido, Nombre',
  dni: '12345678',           // 7-8 digits only
  email: 'user@domain.com',  // valid email regex
  telefono: '0342-1234567'   // optional, 7-15 chars [\d\s\-+()]
}
```

### Note State (for export)
```js
const exportState = {
  fecha: '15 de marzo de 2026',           // editable
  destinatarioHtml: 'Señor Decano de la<br>FBCB...', // from authority + user edit
  asunto: 'Inscripción Fuera de Término',
  cuerpo: 'Me dirijo a usted...',         // editable, pre-wrap
  firma: {                                // editable per line
    nombre: 'García, Juan',
    dni: 'DNI: 30123456',
    email: 'Email: juan@test.com',
    telefono: 'Tel.: 0342-1234567'
  },
  filename: 'Nota_Inscripción_Fuera_de_Término_García,_Juan.pdf'
};
```

## Testing Strategy

| Layer | What to Test | Approach |
|-------|--------------|----------|
| Unit (pure logic) | `formatDate`, `formatSubjects`, `parrafoMotivo`, `parrafoDocumentacion`, DNI/email/phone validators, authority lookup, `generateCuerpo` for each subject | Open `index.html` in browser, call functions in DevTools console; verify outputs against spec scenarios |
| Integration | Form → preview sync, dynamic fields render on subject change, localStorage save/restore, authority selector groups | Manual: fill form, verify preview updates live; reload page, verify localStorage restores; switch subjects, verify fields |
| E2E | Full flow: select authority → subject → fill dynamic fields → fill sender → edit preview → download PDF → verify PDF content; share native (mobile) → verify share sheet | Manual on desktop + mobile (Chrome Android, Safari iOS); test fallback on desktop Chrome |

## Threat Matrix

N/A — no routing, shell, subprocess, VCS/PR automation, executable-file classification, or process-integration boundary.

## Migration / Rollout

No migration required. New project — no existing users, no data migration. First-load initializes empty form with today's date.

## Open Questions

- [ ] Should `contenteditable` preview sections show visual affordance (e.g., subtle border on hover) to indicate editability?
- [ ] Sender profile: add optional "cargo/función" field for Docente/Personal in v1 or defer to v2? (Spec says "puede incluir en v2" — lean toward defer)
- [ ] PDF filename sanitization: keep accents (UTF-8) or transliterate? Spec says "los caracteres se mantienen" — keep as-is.
- [ ] Modal focus trap: implement full roving tabindex or rely on native `<dialog>`? (Use custom modal per reference; add focus trap in JS)

---

## Implementation Build Order (for sdd-tasks)

1. **HTML skeleton + CSS variables + responsive grid** — semantic structure, panels, preview container, modal
2. **Constants module** — `AUTHORITIES[21]`, `ASUNTOS_CONFIG[6]`, `MESES[12]`, regex constants
3. **Pure logic module** — validators, `formatDate`, `formatSubjects`, `parrafoMotivo`, `parrafoDocumentacion`, authority lookup, `generateCuerpo(subject, datos, perfil)`
4. **Persistence module** — `loadProfile()`, `saveProfile()`, `clearProfile()` with v1 schema + validation
5. **UI Controllers** — `renderAuthoritySelector()`, `renderSubjectSelector()`, `renderDynamicFields()`, `collectFormData()`, `renderPreview(note)`, `syncPreviewEdits()`
6. **Event wiring** — form inputs → `actualizar()`, subject change → dynamic fields, buttons → export flow
7. **PDF generation** — `generarPDF({save, share})` with html2pdf options, `pdf-render` class toggle, blob vs save
8. **Native share** — `compartirNativo()`, detection, `canShare` check, fallback, modal open/close, focus trap, Escape
9. **Accessibility polish** — labels, aria-live errors, focus management, mobile viewport test
10. **Manual verification** — run all spec scenarios, test offline, file:// open, mobile share