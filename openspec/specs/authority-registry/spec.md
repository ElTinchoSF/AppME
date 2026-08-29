# Especificación: authority-registry

## Propósito

Catálogo de 21 autoridades oficiales FBCB-UNL (período 2026-2030) con tratamientos formales derivados del cargo, garantizando consistencia género-cargo. 10 autoridades de FBCB + 11 de Rectorado UNL.

## Requisitos

### Requirement: Catálogo de autoridades FBCB (10)

El sistema **DEBE** incluir las 10 autoridades de la Facultad de Bioquímica y Ciencias Biológicas con su tratamiento formal, nombre completo, cargo y facultad de pertenencia.

#### Scenario: Autoridades FBCB listadas correctamente

- DADO el registro de autoridades cargado
- CUANDO se consultan las autoridades de FBCB
- ENTONCES se encuentran exactamente 10 entradas
- Y cada entrada tiene: nombre completo, tratamiento formal, cargo, facultad "FBCB"
- Y los tratamientos respetan género del cargo (ej. "Señor Decano", "Señora Secretaria", "Señor Jefe", "Señora Jefa")

#### Scenario: Tratamientos formales FBCB consistentes

- DADA la autoridad "Decano" (masculino)
- CUANDO se obtiene su tratamiento
- ENTONCES es "Señor Decano"
- DADA la autoridad "Secretaria Administrativa" (femenino)
- CUANDO se obtiene su tratamiento
- ENTONCES es "Señora Secretaria Administrativa"
- DADA la autoridad "Jefe de Personal" (masculino)
- CUANDO se obtiene su tratamiento
- ENTONCES es "Señor Jefe de Personal"
- DADA la autoridad "Jefa de Alumnado" (femenino)
- CUANDO se obtiene su tratamiento
- ENTONCES es "Señora Jefa de Alumnado"

### Requirement: Catálogo de autoridades Rectorado UNL (11)

El sistema **DEBE** incluir las 11 autoridades del Rectorado de la Universidad Nacional del Litoral con su tratamiento formal, nombre completo, cargo y facultad de pertenencia "Rectorado UNL".

#### Scenario: Autoridades Rectorado UNL listadas correctamente

- DADO el registro de autoridades cargado
- CUANDO se consultan las autoridades de Rectorado UNL
- ENTONCES se encuentran exactamente 11 entradas
- Y cada entrada tiene: nombre completo, tratamiento formal, cargo, facultad "Rectorado UNL"
- Y los tratamientos respetan género del cargo

### Requirement: Total de 21 autoridades

El sistema **DEBE** exponer exactamente 21 autoridades en el catálogo completo (10 FBCB + 11 Rectorado UNL).

#### Scenario: Conteo total de autoridades

- DADO el catálogo completo cargado
- CUANDO se cuenta el total de autoridades
- ENTONCES el total es 21

### Requirement: Renderizado de bloque de destinatario

El sistema **DEBE** formatear el bloque de destinatario en la nota como:
```
[Tratamiento formal] de la
[Facultad]
<strong>[Nombre completo]</strong>
S / D
<span style="font-size:12px;color:#666;">([Facultad] — UNL)</span>
```

#### Scenario: Renderizado destinatario FBCB

- DADA una autoridad FBCB con tratamiento "Señor Decano", nombre "Dr. Jorge G. Ramos", cargo "Decano"
- CUANDO se renderiza en la vista previa
- ENTONCES el HTML incluye:
  - "Señor Decano de la<br>Facultad de Bioquímica y Ciencias Biológicas<br><strong>Dr. Jorge G. Ramos</strong><br>S / D<br><span style="font-size:12px;color:#666;">(FBCB — UNL)</span>"

#### Scenario: Renderizado destinatario Rectorado UNL

- DADA una autoridad Rectorado con tratamiento "Señor Rector", nombre "Dr. Enrique Mammarella", cargo "Rector"
- CUANDO se renderiza en la vista previa
- ENTONCES el HTML incluye:
  - "Señor Rector de la<br>Rectorado de la Universidad Nacional del Litoral<br><strong>Dr. Enrique Mammarella</strong><br>S / D<br><span style="font-size:12px;color:#666;">(Rectorado UNL — UNL)</span>"

### Requirement: Selección de destinatario en UI

El sistema **DEBE** poblar el selector de destinatario con todas las 21 autoridades, agrupadas por facultad (FBCB primero, luego Rectorado UNL), mostrando "Nombre — Cargo" como etiqueta.

#### Scenario: Selector muestra todas las autoridades agrupadas

- DADO el selector de destinatario renderizado
- CUANDO el usuario lo abre
- ENTONCES ve 21 opciones más la opción por defecto "-- Seleccione una autoridad --"
- Y las primeras 10 corresponden a FBCB
- Y las siguientes 11 corresponden a Rectorado UNL
- Y cada opción muestra "Nombre — Cargo"

### Requirement: Búsqueda de autoridad por nombre

El sistema **DEBE** permitir obtener la información completa de una autoridad (tratamiento, cargo, facultad) dado su nombre exacto como clave.

#### Scenario: Búsqueda de autoridad existente

- DADO el nombre "Dr. Jorge G. Ramos"
- CUANDO se busca en el registro
- ENTONCES se devuelve: { tratamiento: "Señor Decano", cargo: "Decano", facultad: "FBCB" }

#### Scenario: Búsqueda de autoridad inexistente

- DADO un nombre que no está en el registro
- CUANDO se busca
- ENTONCES se devuelve undefined/null (sin error)

### Requirement: Consistencia género-cargo en tratamientos

El sistema **DEBE** garantizar que todos los tratamientos formales usan el género correcto según el cargo: "Señor" para cargos masculinos, "Señora" para cargos femeninos, aplicado consistentemente en todas las 21 autoridades.

#### Scenario: Verificación de consistencia en todo el catálogo

- DADO el catálogo completo de 21 autoridades
- CUANDO se verifica cada tratamiento
- ENTONCES ningún tratamiento usa "Señor" para cargo femenino ni "Señora" para cargo masculino
- Y todos los tratamientos siguen el patrón "Señor/Señora [Cargo formal]"

## Datos de referencia (extraídos de AppME 2.0 y propuesta)

### Autoridades FBCB (4 en referencia, 10 totales según propuesta)
| Nombre | Tratamiento | Cargo | Facultad |
|--------|-------------|-------|----------|
| Dr. Jorge G. Ramos | Señor Decano | Decano | FBCB |
| Tec. Adriana Vidaechea | Señora Secretaria Administrativa | Secretaria Administrativa | FBCB |
| Ing. Patricio Gómez | Señor Jefe de Personal | Jefe de Personal | FBCB |
| Gabriela Gentina | Señora Jefa de Alumnado | Jefa de Alumnado | FBCB |
| *+ 6 autoridades FBCB adicionales por completar según nómina oficial 2026-2030* | | | |

### Autoridades Rectorado UNL (11 según propuesta)
| Nombre | Tratamiento | Cargo | Facultad |
|--------|-------------|-------|----------|
| *Por completar según nómina oficial 2026-2030 (Rector, Vicerrectores, Secretarios Generales, etc.)* | | | Rectorado UNL |

> **Nota**: La propuesta especifica 21 autoridades totales. La referencia AppME 2.0 solo muestra 4 de FBCB. Las 6 autoridades FBCB restantes y las 11 de Rectorado UNL deben completarse según la nómina oficial 2026-2030 durante la fase de diseño/implementación.