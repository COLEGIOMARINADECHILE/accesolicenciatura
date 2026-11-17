# Control de acceso · Licenciatura 2026

Sistema de invitaciones con **código QR** y control de ingreso en tiempo real (110 estudiantes · 550 invitaciones · 5 por estudiante).

```
Invitación (QR "A00101-9SUE3")  →  Teléfono con docs/index.html  →  Apps Script  →  Google Sheet
```

- Cada invitación tiene un QR único de **uso único**: `CÓDIGO-TOKEN` (el token aleatorio evita que alguien invente códigos).
- Al escanear: verde = ingreso registrado (muestra «3 de 5» del alumno), amarillo = ya usada, rojo = inválida.
- Panel en vivo con totales, avance por curso, últimos movimientos y detalle por alumno (con opción de anular/marcar manualmente).

## Estructura

| Carpeta | Qué contiene | ¿Va a GitHub? |
|---|---|---|
| `docs/index.html` | Página de escaneo + panel (se publica con GitHub Pages) | Sí |
| `docs/generador.html` | Genera las invitaciones con QR para imprimir | Sí |
| `docs/fondo.jpg`, `docs/config.js` | Diseño de la invitación y configuración | Sí |
| `2_APPS_SCRIPT/Code.gs`, `appsscript.json` | Backend (Google Apps Script) | Sí |
| `3_PRIVADO_NO_SUBIR/` | CSV con nombres + tokens y el PDF final | **NO** (en `.gitignore`) |

> **Importante:** los tokens y nombres de estudiantes (menores de edad) no deben subirse a un repositorio público. Quedan solo en tu Google Sheet y en `3_PRIVADO_NO_SUBIR/`.

## Puesta en marcha

### 1. Google Sheet + Apps Script
1. Crea una Google Sheet nueva (ej. «Control Licenciatura 2026»).
2. Menú **Extensiones → Apps Script**. Borra el contenido y pega `2_APPS_SCRIPT/Code.gs`. (En ⚙ Configuración del proyecto puedes activar «Mostrar appsscript.json» y pegar el de esta carpeta.) Guarda y recarga la hoja: aparece el menú **Control Licenciatura**.
3. **Control Licenciatura → 1. Preparar hojas** (autoriza los permisos).
4. En la hoja **Invitaciones**: **Archivo → Importar → Subir** `3_PRIVADO_NO_SUBIR/invitaciones_con_tokens.csv` → «Reemplazar datos en la hoja actual» y **desmarca «Convertir texto en números, fechas y fórmulas»**.
5. **Control Licenciatura → 2. Definir PIN de operadores** (la clave que usarán quienes escaneen).
6. **Implementar → Nueva implementación → Aplicación web**: ejecutar como *Yo*, acceso *Cualquier usuario*. Copia la URL que termina en `/exec`.

### 2. Publicar la página en GitHub
1. Crea un repositorio en GitHub y sube el contenido de esta carpeta (arrastrando archivos en la web o con `git`). No subas `3_PRIVADO_NO_SUBIR/`.
2. **Settings → Pages → Deploy from a branch → `main` / carpeta `/docs`**.
3. Tu página quedará en `https://TU-USUARIO.github.io/NOMBRE-REPO/` (HTTPS, necesario para usar la cámara).
4. Abre esa URL en el teléfono → pestaña **Ajustes** → pega la URL `/exec`, la PIN y tu nombre → **Guardar y probar**. (Opcional: pega la URL en `docs/config.js` para que venga precargada.)

### 3. Imprimir las invitaciones
- Ya está generado `3_PRIVADO_NO_SUBIR/Invitaciones_QR_Licenciatura_2026.pdf` (138 páginas A4, 4 por hoja, mismo diseño con QR).
- Para regenerarlo: abre `docs/generador.html` en Chrome/Edge (con `3_PRIVADO_NO_SUBIR/datos.js` presente, o cargando el CSV) → **Imprimir / Guardar PDF** → A4, márgenes «Ninguno», «Gráficos de fondo» activado.

## Uso el día del evento
1. Cada operador abre la página, ingresa PIN y nombre, y pulsa **Iniciar cámara**.
2. Apunta al QR. Si no se lee, usa **Ingreso manual** con el código impreso (ej. `A00103`).
3. Mira el **Panel** para el conteo total y por curso, y **Alumnos** para ver cuántos de los 5 invitados han entrado.
4. Varios teléfonos pueden escanear a la vez: el servidor usa bloqueo para impedir ingresos dobles.

## Acceso por PIN personal (administrador y funcionarios)
- La página pide solo una **PIN**; al entrar muestra el nombre y el perfil de la persona.
- La clave del **Administrador** es `Licenciatura2026` (constante `CLAVE_ADMIN` en `Code.gs`; no distingue mayúsculas). Se escribe en la misma pantalla de acceso. Cámbiala editando esa línea y publicando una nueva versión.
- En la pestaña **Equipo** (solo administradores) se crean funcionarios con nombre, correo y perfil. El sistema genera una PIN única de 6 caracteres y **la envía por correo** (hoja «Funcionarios»). Se puede reenviar, regenerar, desactivar o eliminar.
- Funcionario: escanear, panel, alumnos y deshacer su propio ingreso por 3 minutos. Administrador: además anular cualquier ingreso y gestionar el equipo.
- La primera vez, ejecuta `autorizarCorreo` en el editor de Apps Script para permitir el envío de correos, y publica una **nueva versión** de la implementación.

## Invitaciones adicionales y especiales (administrador)
Pestaña **Admin → Invitaciones**: crea un *invitado especial* (código V00001…) o invitaciones *adicionales* de un alumno (A00106…), con nombre y cargo opcionales. Se guardan en la hoja «Invitaciones» (columnas Invitado, Cargo, Tipo), aparecen en el listado y en el PDF, y se descargan como imagen (digital, para enviar por WhatsApp) o PDF 11×15 cm (impresa). Al escanearlas, el funcionario ve el nombre y el cargo.

## Exportar listado en PDF
Pestaña **Alumnos** (solo administradores): «Lista de invitaciones» o «Con asistencia» (marca en verde las ya utilizadas). Genera un PDF por curso con los 5 códigos de cada alumno; no incluye los tokens secretos.

## Script independiente (error «Cannot call SpreadsheetApp.getUi()»)
Ese error aparece cuando el script se creó en script.google.com y no desde la hoja. Dos soluciones: (a) crear el script desde la hoja con **Extensiones → Apps Script**, o (b) en `Code.gs` completar `SHEET_ID` (ID de la hoja) y `PIN_INICIAL`, y ejecutar la función **configurarTodo**. `onOpen` ya no falla en ningún caso.

## Alumno de prueba
`3_PRIVADO_NO_SUBIR/invitaciones_prueba_ALARCON.csv` trae 5 invitaciones (T00101–T00105, curso «Prueba») para ALARCÓN MEDINA JOSÉ ALONZO; su PDF es `Invitaciones_PRUEBA_ALARCON.pdf`. Impórtalas **agregando** filas al final de la hoja «Invitaciones». Para borrarlas después, elimina esas 5 filas.

## Revertir un ingreso
Tres formas: botón **Deshacer este ingreso** en la pantalla verde tras escanear; botón **Revertir** con el código en la pestaña Escanear; o **Anular** en la pestaña Alumnos. Queda anotado como ANULADO en la hoja «Registro».

## Ícono de la app
`docs/icons` + `manifest.webmanifest`: al abrir la página en el celular, «Agregar a pantalla de inicio» crea el ícono **Licenciatura 2026** con la insignia del colegio.

## Modo demo
En **Ajustes → Modo demo** (o `index.html?demo=1`) puedes probar todo sin conexión; códigos de prueba `A00101-DEMO1` … `A00105-DEMO1`.

## Notas técnicas
- QR en modo alfanumérico, corrección de errores Q, versión 1 (21×21 módulos): módulos grandes y lectura rápida en celular.
- La PIN viaja como parámetro de la consulta HTTPS; es suficiente para un evento escolar, pero no reutilices una clave personal.
- Hoja **Registro** guarda cada escaneo (ingreso, duplicado, inválido, anulado) con hora y operador.


