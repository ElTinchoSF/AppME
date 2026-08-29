# Especificación: note-generator

## Propósito

Motor de generación de notas formales para FBCB-UNL con plantillas por asunto. Produce la estructura completa: fecha → destinatario → "Ref.: [asunto]" → cuerpo → firma, permitiendo edición en vivo de la vista previa antes de exportar.

## Requisitos

### Requirement: Generación de estructura base de nota

El sistema **DEBE** generar una nota con la estructura canónica: fecha (editable, default hoy), bloque de destinatario (tratamiento formal + nombre + cargo + facultad), línea de referencia "Ref.: [asunto]", cuerpo de texto justificado, y bloque de firma (línea + datos del remitente).

#### Scenario: Generación de nota vacía sin datos

- DADO que el usuario abre la aplicación sin seleccionar destinatario ni asunto
- CUANDO se renderiza la vista previa inicial
- ENTONCES la fecha muestra la fecha de hoy en formato "DD de MMMM de YYYY"
- Y el destinatario muestra placeholder "[Seleccione un destinatario]"
- Y la referencia muestra "[Seleccione un asunto]"
- Y el cuerpo muestra "[Seleccione un asunto para generar el contenido]"
- Y la firma muestra placeholders "[Nombre completo]", "DNI: [---]", "Email: [---]", "Tel.: [---]"

#### Scenario: Generación de nota con datos completos

- DADO que el usuario seleccionó destinatario, asunto, completó campos dinámicos y datos del remitente
- CUANDO se actualiza la vista previa
- ENTONCES la fecha muestra la fecha seleccionada (editable)
- Y el destinatario muestra tratamiento formal, nombre, cargo y "Facultad de Bioquímica y Ciencias Biológicas\nS / D\n(FBCB — UNL)"
- Y la referencia muestra "Ref.: [Asunto seleccionado]"
- Y el cuerpo muestra el texto generado por la plantilla del asunto con los campos dinámicos interpolados
- Y la firma muestra nombre, DNI, email y teléfono del remitente

### Requirement: Plantillas por asunto (6 asuntos oficiales)

El sistema **DEBE** incluir plantillas para los 6 asuntos oficiales con sus campos dinámicos obligatorios y opcionales, y funciones generadoras que produzcan texto formal en español neutro.

#### Scenario: Inscripción Fuera de Término

- DADO que el usuario selecciona "Inscripción Fuera de Término"
- CUANDO completa carrera, materias (lista multilínea) y motivo opcional
- ENTONCES el cuerpo incluye: solicitud de inscripción fuera de término, carrera, listado de materias con viñetas "• ", párrafo de motivo si se completó, y cierre formal "Agradeciendo anticipadamente su atención, saludo a usted atentamente."

#### Scenario: Extensión de Regularidad

- DADO que el usuario selecciona "Extensión de Regularidad"
- CUANDO completa carrera, materias (lista multilínea) y motivo opcional
- ENTONCES el cuerpo incluye: solicitud de extensión de regularidad, carrera, listado de materias con viñetas, párrafo de motivo si se completó, frase "Quedo a disposición para cualquier consulta o requerimiento adicional.", y cierre formal

#### Scenario: Solicitud de Licencia

- DADO que el usuario selecciona "Solicitud de Licencia"
- CUANDO completa tipo de licencia, fecha desde, fecha hasta, y motivo opcional
- ENTONCES el cuerpo incluye: solicitud del tipo de licencia por el período "entre el [fecha desde] y el [fecha hasta] inclusive", párrafo de motivo si se completó, frase "Quedo a disposición para cualquier consulta.", y cierre formal

#### Scenario: Inscripción a Concurso Docente

- DADO que el usuario selecciona "Inscripción a Concurso Docente"
- CUANDO completa cargo, cátedra, área
- ENTONCES el cuerpo incluye: solicitud de inscripción al concurso para el cargo en la cátedra del área, declaración de cumplimiento de requisitos, frase "Adjunto la documentación respaldatoria correspondiente y quedo a disposición para cualquier consulta o requerimiento adicional.", y cierre formal

#### Scenario: Inscripción a Concurso Alumno

- DADO que el usuario selecciona "Inscripción a Concurso Alumno"
- CUANDO completa cargo/función, cátedra/sector, área
- ENTONCES el cuerpo incluye: solicitud de inscripción al concurso para el cargo en la cátedra/sector del área, declaración de cumplimiento de requisitos, frase de documentación adjunta, y cierre formal

#### Scenario: Solicitud de Plan de Estudios y Programas

- DADO que el usuario selecciona "Solicitud de Plan de Estudios y Programas"
- CUANDO completa carrera y opcionalmente materias específicas
- ENTONCES SI se completaron materias: el cuerpo solicita planes "de las asignaturas que se detallan a continuación" con listado con viñetas
- ENTONCES SI NO se completaron materias: el cuerpo solicita planes "correspondientes a la totalidad del plan de estudios de la carrera"
- Y EN AMBOS CASOS incluye: propósito "fines académicos/personales" y cierre formal

### Requirement: Edición en vivo de la vista previa

El sistema **DEBE** permitir que el usuario edite directamente cualquier sección de la vista previa (fecha, destinatario, referencia, cuerpo, firma) y que los cambios persistan al exportar PDF.

#### Scenario: Edición de fecha en vista previa

- DADO que la vista previa muestra la fecha de hoy
- CUANDO el usuario modifica el texto de la fecha en la vista previa
- ENTONCES el PDF exportado incluye la fecha editada por el usuario

#### Scenario: Edición de cuerpo en vista previa

- DADO que la vista previa muestra el cuerpo generado por plantilla
- CUANDO el usuario modifica párrafos, agrega o elimina texto en el cuerpo
- ENTONCES el PDF exportado refleja exactamente el texto editado

#### Scenario: Edición de firma en vista previa

- DADO que la vista previa muestra los datos del remitente en la firma
- CUANDO el usuario modifica nombre, DNI, email o teléfono en la firma
- ENTONCES el PDF exportado incluye los datos editados

### Requirement: Formateo de lista de asignaturas

El sistema **DEBE** convertir texto multilínea de asignaturas en lista con viñetas "• " (una por línea no vacía), ignorando líneas en blanco.

#### Scenario: Formateo de múltiples asignaturas

- DADO un texto con 3 líneas no vacías y 1 línea en blanco
- CUANDO se formatea la lista
- ENTONCES se producen 3 líneas precedidas por "• " sin línea en blanco extra

#### Scenario: Formateo con líneas vacías al inicio/final

- DADO un texto con líneas vacías al inicio y final
- CUANDO se formatea la lista
- ENTONCES las líneas vacías se ignoran y solo se procesan líneas con contenido

### Requirement: Formateo de fecha en español

El sistema **DEBE** formatear fechas como "DD de MMMM de YYYY" usando nombres de meses en español (enero, febrero, ..., diciembre).

#### Scenario: Formateo de fecha válida

- DADA una fecha "2026-03-15"
- CUANDO se formatea
- ENTONCES el resultado es "15 de marzo de 2026"

#### Scenario: Formateo de fecha inválida o vacía

- DADA una fecha vacía o inválida
- CUANDO se formatea
- ENTONCES el resultado es "[fecha]"

### Requirement: Párrafo de motivo condicional

El sistema **DEBE** incluir el párrafo "La presente solicitud se funda en los siguientes motivos: [motivo]." solo cuando el campo motivo tiene contenido no vacío.

#### Scenario: Motivo proporcionado

- DADO un motivo "Motivos personales"
- CUANDO se genera el párrafo
- ENTONCES el resultado incluye "La presente solicitud se funda en los siguientes motivos: Motivos personales."

#### Scenario: Motivo vacío

- DADO un motivo vacío o solo espacios
- CUANDO se genera el párrafo
- ENTONCES el resultado es cadena vacía

### Requirement: Párrafo de documentación condicional

El sistema **DEBE** incluir el párrafo "Adjunto la documentación respaldatoria correspondiente y quedo a disposición para cualquier consulta o requerimiento adicional." solo para asuntos que lo requieran (concursos).

#### Scenario: Asunto con documentación requerida

- DADO un asunto configurado con `conDocumentacion: true`
- CUANDO se genera el cuerpo
- ENTONCES el párrafo de documentación adjunta se incluye

#### Scenario: Asunto sin documentación requerida

- DADO un asunto configurado sin `conDocumentacion`
- CUANDO se genera el cuerpo
- ENTONCES el párrafo de documentación adjunta NO se incluye