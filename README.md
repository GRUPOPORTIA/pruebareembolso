# Formulario de Rendición de Gastos — Grupo Portia

Formulario web para que el personal rinda gastos (viáticos, transporte, notaría, etc.), con generación de PDF, adjuntar facturas, cálculo automático de saldo/anticipo y envío automático a una planilla de Google Sheets + correo a Finanzas.

## Estructura del proyecto

```
index.html   → Frontend completo (HTML + CSS + JS). Este es el archivo que se sube a GitHub.
Code.gs      → Backend. Vive SOLO en Google Apps Script (no en este repo), porque es
               la única forma de conectarse a Google Sheets y enviar correo con MailApp.
```

⚠️ **`Code.gs` no está en este repositorio.** Se administra directamente en el editor de
Apps Script del proyecto vinculado a la planilla de respaldo. Si se pierde o hay que
entregarlo, pedir la copia más reciente a quien mantenga el proyecto.

## Cómo funciona

1. La persona completa el formulario (empresa, centro de costo, período, gastos, etc.).
2. Al elegir un **Centro de Costo**, el buscador solo muestra el nombre (sin el código
   interno) — el código va oculto en `#centroCostoCodigo` y viaja igual al backend.
3. Todos los campos de texto se fuerzan a **MAYÚSCULAS** automáticamente al escribir.
4. Si hay saldo a favor del trabajador, se muestran los campos de cuenta bancaria; si
   hay saldo en contra, se muestran los datos de transferencia de la empresa.
5. Al presionar **"Enviar formulario"**:
   - Se valida que los campos obligatorios estén completos.
   - Se genera un PDF en el navegador con el diseño real del formulario (usa
     [html2pdf.js](https://github.com/eKoopmans/html2pdf.js) desde CDN).
   - Se envía todo (datos + facturas en base64 + PDF) por `POST` al Web App de Apps
     Script (`URL_APPS_SCRIPT` en `index.html`).
   - El backend guarda la rendición en Sheets, genera (si corresponde) el Excel de
     transferencia (TEF) y envía el correo con los adjuntos a Finanzas.
   - Aparece el diálogo de impresión del navegador para que la persona guarde/imprima
     su comprobante. **Apenas se cierra ese diálogo (imprima o cancele), el formulario
     se borra solo** — la rendición ya quedó registrada en el servidor, así se evita
     reenviarla duplicada.

## Configuración necesaria

En `index.html`:

```js
const URL_APPS_SCRIPT = "https://script.google.com/macros/s/AKfycby.../exec";
```

En `Code.gs` (en Apps Script):

```js
var SPREADSHEET_ID = "..."; // ID de la planilla de respaldo
var CORREO_DESTINO = "apasten@grupoportia.cl"; // a quién le llega el correo
```

### Estructura esperada de la planilla de respaldo

La planilla (Google Sheets) debe tener estas dos pestañas exactas:

**`Rendiciones`**

| idRendicion | Empresa | Código CC | Centro de Costo | Nombre | Rut | Mes | Año | Zona | Total | Anticipo | Saldo | Fecha | Cuenta Bancaria | Banco Destino | Código SBIF |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

**`Detalle_Gastos`**

| idRendicion | Empresa | Nombre | Categoría | Detalle | Documento | Valor |
|---|---|---|---|---|---|---|

Si estas pestañas no existen o tienen otro nombre/orden de columnas, `doPost` en
`Code.gs` va a fallar.

## Despliegue

- **Frontend (`index.html`)**: se sube a este repo de GitHub. Actualmente el repo no
  tiene GitHub Pages activado, así que el archivo no tiene un link público propio
  todavía — se comparte/abre como archivo. Si en algún momento se quiere un link web,
  se puede activar GitHub Pages en la configuración del repo.
- **Backend (`Code.gs`)**: se pega directo en el editor de Apps Script del proyecto
  vinculado a la planilla, y se debe **redesplegar** (Implementar → Nueva
  implementación) cada vez que se modifica, para que el link `URL_APPS_SCRIPT` tome los
  cambios (o quedarse con el mismo link si se elige "Administrar implementaciones" →
  editar la implementación existente en vez de crear una nueva).

## Notas para quien reciba el proyecto

- Los cambios al frontend se hacen sobre `index.html` y se suben a este repo.
- Los cambios al backend se hacen en Apps Script; pedir acceso al proyecto de Apps
  Script vinculado a la planilla (`SPREADSHEET_ID` en `Code.gs`) y a la cuenta de
  Google que administra el envío de correos (`CORREO_DESTINO`).
- El listado de centros de costo (`CENTROS_COSTO` en `index.html`) es una copia
  estática de los clientes/empresas que administra Grupo Portia — hay que actualizarlo
  a mano si se agregan o eliminan centros de costo.
