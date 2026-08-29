# Especificación: sender-profile

## Propósito

Gestión del perfil del remitente: tipo (Alumno/Docente/Personal/Otro), datos personales (nombre, DNI, email, teléfono), validaciones, persistencia en localStorage y adaptación del cuerpo/firma según el tipo.

## Requisitos

### Requirement: Tipo de remitente (perfil)

El sistema **DEBE** permitir seleccionar el tipo de remitente entre: Alumno, Docente, Personal, Otro. Esta selección afecta la redacción del cuerpo de la nota y la firma.

#### Scenario: Selección de tipo Alumno

- DADO que el usuario selecciona "Alumno" como tipo de remitente
- CUANDO se genera una nota de "Inscripción Fuera de Término"
- ENTONCES el cuerpo usa redacción en primera persona de alumno ("Me dirijo a usted a fin de solicitar **mi** inscripción...")
- Y la firma muestra solo datos personales (sin cargo institucional)

#### Scenario: Selección de tipo Docente

- DADO que el usuario selecciona "Docente" como tipo de remitente
- CUANDO se genera una nota de "Solicitud de Licencia"
- ENTONCES el cuerpo usa redacción docente ("Me dirijo a usted a fin de solicitar **mi** licencia...")
- Y la firma puede incluir cargo docente si se proporciona

#### Scenario: Selección de tipo Personal

- DADO que el usuario selecciona "Personal" como tipo de remitente
- CUANDO se genera una nota
- ENTONCES el cuerpo usa redacción de personal administrativo/no docente
- Y la firma puede incluir cargo administrativo

#### Scenario: Selección de tipo Otro

- DADO que el usuario selecciona "Otro" como tipo de remitente
- CUANDO se genera una nota
- ENTONCES el cuerpo usa redacción genérica neutra
- Y la firma muestra solo datos personales básicos

### Requirement: Datos obligatorios del remitente

El sistema **DEBE** requerir y validar: nombre completo, DNI (7-8 dígitos), email (formato válido). El teléfono es opcional pero validado si se completa.

#### Scenario: Nombre completo obligatorio

- DADO que el campo nombre está vacío
- CUANDO el usuario intenta generar/descargar la nota
- ENTONCES la validación falla y se muestra error indicando que el nombre es obligatorio

#### Scenario: DNI obligatorio 7-8 dígitos

- DADO que el DNI tiene 6 dígitos o 9 dígitos o contiene letras
- CUANDO el usuario intenta generar/descargar la nota
- ENTONCES la validación falla con mensaje: "Por favor, ingrese un DNI válido (solo números, 7 u 8 dígitos, sin puntos)."
- Y el foco se posiciona en el campo DNI

#### Scenario: DNI válido 7 dígitos

- DADO un DNI "1234567" (7 dígitos)
- CUANDO se valida
- ENTONCES la validación pasa

#### Scenario: DNI válido 8 dígitos

- DADO un DNI "12345678" (8 dígitos)
- CUANDO se valida
- ENTONCES la validación pasa

#### Scenario: Email obligatorio formato válido

- DADO un email "invalido" o "falta@dominio" o "@dominio.com"
- CUANDO el usuario intenta generar/descargar la nota
- ENTONCES la validación falla con mensaje: "Por favor, ingrese un correo electrónico válido."
- Y el foco se posiciona en el campo email

#### Scenario: Email válido

- DADO un email "usuario@dominio.com" o "nombre.apellido@fbcb.unl.edu.ar"
- CUANDO se valida
- ENTONCES la validación pasa

#### Scenario: Teléfono opcional con validación si se completa

- DADO que el teléfono está vacío
- CUANDO se valida
- ENTONCES la validación pasa (campo opcional)
- DADO un teléfono "0342-1234567" o "+54 342 123 4567" o "3421234567"
- CUANDO se valida (7-15 caracteres, dígitos, espacios, guiones, paréntesis, +)
- ENTONCES la validación pasa
- DADO un teléfono "abc" o "123" (menos de 7 chars válidos)
- CUANDO se valida
- ENTONCES la validación falla con mensaje: "Por favor, ingrese un número de teléfono válido."

### Requirement: Persistencia en localStorage

El sistema **DEBE** guardar automáticamente los datos del remitente (tipo, nombre, DNI, email, teléfono) en localStorage y restaurarlos al cargar la aplicación.

#### Scenario: Guardado automático al cambiar datos

- DADO que el usuario modifica cualquier campo del remitente
- CUANDO el campo pierde foco o cambia (oninput)
- ENTONCES los datos se guardan en localStorage bajo clave "fbcb-sender-profile"
- Y el valor almacenado es un objeto JSON con: tipo, nombre, dni, email, telefono

#### Scenario: Restauración al cargar aplicación

- DADO que localStorage tiene datos previos válidos
- CUANDO la aplicación se carga (file://)
- ENTONCES los campos del remitente se poblan con los valores guardados
- Y el tipo de remitente se restaura en el selector

#### Scenario: Limpieza de localStorage inválido

- DADO que localStorage tiene datos corruptos (JSON inválido, estructura incorrecta)
- CUANDO la aplicación intenta restaurar
- ENTONCES se ignora el localStorage corrupto y se muestran campos vacíos
- Y no se produce error de JavaScript

#### Scenario: Sin localStorage previo (primera vez)

- DADO que localStorage no tiene clave "fbcb-sender-profile"
- CUANDO la aplicación se carga
- ENTONCES todos los campos del remitente están vacíos
- Y el tipo de remitente está en valor por defecto (primera opción: "Alumno")

### Requirement: Adaptación de firma según tipo de remitente

El sistema **DEBE** ajustar el bloque de firma según el tipo de remitente seleccionado.

#### Scenario: Firma para Alumno

- DADO tipo "Alumno" con nombre, DNI, email, teléfono completados
- CUANDO se renderiza la firma
- ENTONCES muestra:
  - Línea de firma
  - Nombre completo
  - "DNI: [número]"
  - "Email: [email]"
  - "Tel.: [teléfono]" (solo si se completó)

#### Scenario: Firma para Docente/Personal

- DADO tipo "Docente" o "Personal" con datos completados
- CUANDO se renderiza la firma
- ENTONCES muestra los mismos datos que Alumno
- Y **PUEDE** incluir línea adicional con cargo/función si se implementa campo extra en v2 (fuera de alcance v1)

### Requirement: Validación integrada en flujo de exportación

El sistema **DEBE** validar todos los datos del remitente antes de permitir generación de PDF o compartir.

#### Scenario: Validación exitosa permite exportar

- DADO que todos los campos obligatorios son válidos
- CUANDO el usuario pulsa "Descargar PDF" o "Compartir"
- ENTONCES se procede a generar el PDF

#### Scenario: Validación fallida bloquea exportación

- DADO que algún campo obligatorio es inválido
- CUANDO el usuario pulsa "Descargar PDF" o "Compartir"
- ENTONCES se muestra alerta con el error específico
- Y el foco va al campo inválido
- Y NO se genera el PDF

### Requirement: Placeholders en vista previa sin datos

El sistema **DEBE** mostrar placeholders descriptivos en la firma cuando los datos del remitente no están completados.

#### Scenario: Placeholders en firma vacía

- DADO que el usuario no ha completado ningún dato del remitente
- CUANDO se renderiza la vista previa
- ENTONCES la firma muestra:
  - "[Nombre completo]"
  - "DNI: [---]"
  - "Email: [---]"
  - "Tel.: [---]"

#### Scenario: Placeholders parciales

- DADO que el usuario completó solo nombre y email
- CUANDO se renderiza la vista previa
- ENTONCES la firma muestra:
  - Nombre real
  - "DNI: [---]"
  - Email real
  - "Tel.: [---]"