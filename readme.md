# ARIAE — Centro de Soporte Técnico y Seguimiento de Incidencias

## Documentación técnica del sistema

**Proyecto:** ARIAE — Motores Aeronáuticos con Realidad Aumentada  
**Institución:** Facultad Politécnica — Universidad Nacional de Asunción (FP-UNA)  
**Tipo de solución:** Aplicación web de soporte, registro y seguimiento de incidencias  
**Frontend:** HTML5, CSS3 y JavaScript  
**Hosting:** GitHub Pages  
**Backend y persistencia:** Supabase (PostgreSQL, Auth, Row Level Security y Edge Functions)

---

## 1. Introducción

El Centro de Soporte Técnico ARIAE fue desarrollado como una interfaz web complementaria a la aplicación didáctica ARIAE. Su finalidad es proporcionar un mecanismo sencillo y centralizado para registrar fallas, consultar el estado de una incidencia y permitir al equipo docente gestionar los reportes recibidos.

La solución se diseñó con una arquitectura separada en dos partes:

1. **Frontend estático**, responsable de la interfaz de usuario, validaciones básicas, interacción y presentación de la información.
2. **Backend administrado**, responsable de la persistencia de datos, autenticación, autorización y lógica de servidor.

Esta separación permite mantener una página web liviana y fácil de publicar, sin necesidad de administrar un servidor web tradicional.

---

## 2. Objetivo general

Desarrollar un sistema web de soporte técnico que permita registrar, rastrear y administrar incidencias relacionadas con el uso de ARIAE, garantizando separación de responsabilidades, persistencia de datos y controles de acceso adecuados para usuarios públicos y docentes.

### 2.1. Objetivos específicos

- Permitir que un alumno o usuario registre una incidencia sin necesidad de crear una cuenta.
- Generar un código único y fácil de conservar para cada incidencia.
- Permitir la consulta del estado de un ticket mediante su código.
- Permitir que un docente autenticado visualice todos los reportes.
- Permitir que el docente cambie el estado de una incidencia y agregue una respuesta oficial.
- Proteger los datos internos mediante políticas de seguridad en la base de datos.
- Evitar que el navegador tenga acceso a credenciales administrativas.
- Mantener un frontend responsivo y compatible con dispositivos móviles y de escritorio.
- Facilitar el despliegue y mantenimiento del proyecto utilizando servicios administrados.

---

## 3. Arquitectura general

La arquitectura implementada puede representarse de la siguiente manera:

```text
                        USUARIO / ALUMNO
                              │
                              ▼
                    ┌───────────────────┐
                    │   GitHub Pages    │
                    │ HTML + CSS + JS   │
                    └─────────┬─────────┘
                              │
             ┌────────────────┴────────────────┐
             │                                 │
             ▼                                 ▼
     Edge Function                       Edge Function
      create-ticket                       track-ticket
             │                                 │
             └────────────────┬────────────────┘
                              ▼
                    ┌───────────────────┐
                    │     Supabase      │
                    │    PostgreSQL     │
                    │ tabla: tickets    │
                    └─────────┬─────────┘
                              ▲
                              │
                     Supabase Auth + RLS
                              │
                    ┌─────────┴─────────┐
                    │  PANEL DOCENTE    │
                    │ usuario autentic. │
                    └───────────────────┘
```

### 3.1. Separación entre acceso público y acceso docente

El sistema contempla dos tipos de interacción:

**Acceso público**

- Crear un ticket.
- Consultar un ticket por código.

Estas acciones no acceden directamente a la tabla `tickets`. Se realizan mediante las Edge Functions `create-ticket` y `track-ticket`.

**Acceso docente**

- Iniciar sesión.
- Visualizar todos los tickets.
- Cambiar el estado.
- Agregar o modificar la respuesta oficial.
- Exportar los reportes a CSV.

Estas acciones requieren una sesión válida de Supabase Auth y están sujetas a las reglas de Row Level Security (RLS) de PostgreSQL.

---

## 4. Tecnologías utilizadas y justificación

## 4.1. HTML5

Se utilizó HTML5 para definir la estructura semántica de la aplicación.

Entre los elementos empleados se encuentran:

- `header`
- `nav`
- `main`
- `section`
- `form`
- `table`
- `aside`
- `footer`

La utilización de etiquetas semánticas permite que la estructura de la página sea más clara tanto para mantenimiento como para tecnologías de asistencia y motores de búsqueda.

También se incorporaron atributos como:

- `aria-label`
- `aria-labelledby`
- `aria-live`
- `role="dialog"`
- `role="status"`

Esto mejora la accesibilidad y permite que los cambios dinámicos de estado puedan ser interpretados por lectores de pantalla.

---

## 4.2. CSS3

CSS3 se utiliza para la presentación visual, adaptación responsiva y separación entre estructura y estilo.

Los estilos se encuentran divididos en módulos:

```text
css/
├── base.css
├── header.css
├── main.css
├── footer.css
└── styles.css
```

`styles.css` funciona como archivo maestro e importa los demás módulos.

Esta separación fue elegida para evitar un único archivo excesivamente grande y facilitar el mantenimiento.

### Variables CSS

El proyecto utiliza custom properties en `:root` para centralizar colores, tipografías, radios y sombras.

Ejemplo conceptual:

```css
:root {
  --color-aero-blue: #1E3A8A;
  --color-aero-cyan: #06B6D4;
  --color-success: #10B981;
  --radius-md: 14px;
}
```

La ventaja es que un cambio de identidad visual puede realizarse desde un único punto.

### Diseño responsivo

Se emplean media queries para reorganizar formularios, botones, encabezado y panel administrativo según el ancho disponible.

Esto permite que la aplicación sea utilizada desde:

- teléfonos móviles;
- tablets;
- notebooks;
- computadoras de escritorio.

---

## 4.3. JavaScript sin framework

La lógica del frontend fue implementada con JavaScript nativo.

Se eligió esta alternativa porque el alcance del sistema no requiere un framework completo como React, Angular o Vue.

El proyecto necesita principalmente:

- manipulación del DOM;
- formularios;
- cambio de pestañas;
- llamadas HTTP mediante el SDK de Supabase;
- manejo de sesiones;
- renderizado de resultados;
- exportación de CSV.

Utilizar JavaScript nativo reduce dependencias, evita un proceso adicional de compilación y mantiene el proyecto compatible con el modelo de publicación estática de GitHub Pages.

---

## 4.4. GitHub Pages

GitHub Pages se utiliza para alojar el frontend.

GitHub Pages es apropiado para el proyecto porque puede publicar directamente archivos estáticos HTML, CSS y JavaScript almacenados en un repositorio Git.

### Motivos de elección

- integración directa con el repositorio;
- despliegue sencillo;
- no requiere administrar Apache, Nginx o un servidor propio;
- resulta adecuado para una interfaz construida con HTML, CSS y JavaScript;
- facilita el control de versiones del frontend;
- permite mantener el código del proyecto y la versión publicada dentro del mismo flujo de trabajo.

### Limitación asumida

GitHub Pages no ejecuta lógica de servidor propia del proyecto. Por este motivo, las operaciones sensibles se delegaron a Supabase Edge Functions.

Esta característica influyó directamente en la arquitectura seleccionada: GitHub Pages se ocupa de la capa de presentación y Supabase se ocupa de la capa de datos y servidor.

Documentación oficial:

https://docs.github.com/pages/getting-started-with-github-pages/what-is-github-pages

---

## 4.5. Supabase

Supabase se utiliza como plataforma backend.

La plataforma proporciona, dentro del mismo proyecto:

- base de datos PostgreSQL;
- autenticación;
- Row Level Security;
- API de datos;
- Edge Functions;
- herramientas de monitoreo y logs.

Esta integración reduce la necesidad de configurar y mantener múltiples servidores independientes.

### Motivos de elección

Se eligió Supabase porque el proyecto necesitaba simultáneamente:

1. persistencia relacional;
2. autenticación docente;
3. autorización;
4. lógica de servidor;
5. integración sencilla con JavaScript;
6. posibilidad de administrar el sistema desde un Dashboard.

Al estar basado en PostgreSQL, el sistema conserva conceptos tradicionales y ampliamente utilizados de bases de datos relacionales: tablas, claves primarias, índices, restricciones, roles y políticas.

Documentación oficial:

https://supabase.com/docs

---

## 4.6. PostgreSQL

La información se almacena en una tabla relacional denominada `tickets`.

PostgreSQL permite definir integridad directamente en la base de datos mediante:

- tipos de datos;
- `NOT NULL`;
- restricciones `CHECK`;
- claves `UNIQUE`;
- claves primarias;
- índices;
- políticas RLS.

La validación de datos no depende exclusivamente del navegador.

---

## 4.7. Supabase Auth

Supabase Auth se utiliza exclusivamente para el acceso docente.

El sistema abandonó el modelo inicial de PIN almacenado en JavaScript porque un dato incluido en el frontend puede ser inspeccionado por cualquier usuario desde las herramientas del navegador.

El modelo actual utiliza:

- correo electrónico;
- contraseña;
- sesión administrada por Supabase;
- JWT;
- rol PostgreSQL `authenticated`.

El registro público de usuarios se encuentra deshabilitado. Las cuentas docentes son creadas manualmente desde el Dashboard.

De esta manera, un visitante no puede registrarse libremente para obtener acceso al panel.

Documentación oficial:

https://supabase.com/docs/guides/auth

---

## 4.8. Row Level Security (RLS)

RLS es una característica de PostgreSQL utilizada para aplicar autorización directamente en la base de datos.

La tabla `tickets` tiene RLS habilitado.

### Usuario público

El rol `anon` no recibe acceso directo a la tabla.

Esto impide que desde las herramientas del navegador un visitante pueda ejecutar libremente:

```text
SELECT *
INSERT
UPDATE
DELETE
```

sobre `tickets`.

### Usuario autenticado

Los usuarios autenticados utilizados por el proyecto son docentes creados manualmente.

Se habilitaron:

- `SELECT` sobre los tickets;
- `UPDATE` únicamente sobre las columnas `status` y `admin_reply`.

Esto aplica el principio de **mínimo privilegio**: el panel puede modificar solamente los datos que necesita modificar.

### Importancia de esta decisión

La autorización no depende solamente de ocultar botones en la interfaz.

Aunque alguien intentara modificar manualmente el JavaScript del navegador, PostgreSQL continúa aplicando los permisos definidos en el servidor.

Documentación oficial:

https://supabase.com/docs/guides/database/postgres/row-level-security

---

## 4.9. Supabase Edge Functions

Las operaciones públicas se implementan mediante dos funciones de servidor:

```text
create-ticket
track-ticket
```

Las Edge Functions permiten ejecutar TypeScript del lado servidor sin administrar infraestructura propia.

### ¿Por qué no realizar INSERT y SELECT públicos directamente desde JavaScript?

Porque otorgar permisos directos a usuarios no autenticados sobre toda la tabla habría aumentado la superficie de exposición de la base de datos.

Con las Edge Functions el flujo queda controlado:

```text
Navegador
   │
   ▼
Edge Function
   │
   ├── valida datos
   ├── decide qué operación realizar
   ├── decide qué campos devolver
   ▼
PostgreSQL
```

Además, el código de servidor puede utilizar privilegios que nunca se exponen en el navegador.

Documentación oficial:

https://supabase.com/docs/guides/functions

---

## 4.10. Publishable key y claves secretas

El frontend utiliza una clave:

```text
sb_publishable_...
```

Una publishable key está diseñada para componentes públicos, incluido código JavaScript enviado al navegador.

La seguridad del sistema no depende de ocultar esa clave, sino de configurar correctamente permisos, RLS y autenticación.

Una clave `sb_secret_...` no debe incluirse nunca en:

- `config.js`;
- HTML;
- JavaScript del frontend;
- repositorios públicos;
- documentación pública.

Las claves secretas pertenecen exclusivamente a componentes controlados del backend.

Documentación oficial:

https://supabase.com/docs/guides/getting-started/api-keys

---

# 5. Estructura de la base de datos

La tabla principal es:

```text
public.tickets
```

## 5.1. Campos

| Campo | Tipo conceptual | Descripción |
|---|---|---|
| `id` | UUID | Identificador interno único |
| `ticket_code` | text / unique | Código entregado al usuario |
| `title` | text | Título de la incidencia |
| `device` | text | Dispositivo utilizado |
| `os` | text | Sistema/versión de Android |
| `description` | text | Descripción detallada |
| `reporter_name` | text | Nombre del remitente |
| `reporter_email` | text | Correo del remitente |
| `status` | text | Estado de la incidencia |
| `admin_reply` | text | Respuesta oficial docente |
| `created_at` | timestamptz | Fecha y hora de creación |

## 5.2. Estados permitidos

Los estados se restringen a:

```text
pendiente
en_revision
resuelto
descartado
```

La restricción se aplica en PostgreSQL mediante `CHECK`.

Esto evita guardar estados arbitrarios o inconsistentes.

## 5.3. Identificador interno y código público

Se utilizan dos identificadores diferentes.

### UUID interno

`id` identifica técnicamente una fila de la base.

No necesita ser mostrado al usuario.

### Código de seguimiento

El usuario recibe un código de la forma:

```text
ARIAE-ENG-48217
```

El prefijo identifica el sistema y los últimos cinco caracteres son dígitos.

El código se guarda con restricción `UNIQUE`.

---

# 6. Generación del código de ticket

La función `create-ticket` utiliza `crypto.getRandomValues()` para obtener un valor aleatorio y convertirlo a un número entre:

```text
10000 y 99999
```

Por lo tanto, el formato generado es:

```text
ARIAE-ENG-XXXXX
```

Ejemplo:

```text
ARIAE-ENG-48217
```

Existen 90.000 combinaciones posibles.

## 6.1. Manejo de colisiones

Al utilizar solamente cinco dígitos, es posible que con el tiempo se intente generar un código ya existente.

Para evitar duplicados:

1. `ticket_code` tiene restricción `UNIQUE`.
2. La función intenta insertar el ticket.
3. Si PostgreSQL devuelve el error de clave duplicada (`23505`), se genera otro número.
4. Se permiten varios reintentos antes de devolver un error.

De esta manera, la base de datos continúa siendo la autoridad final respecto de la unicidad.

## 6.2. Implicancia de seguridad

Un código de cinco dígitos fue elegido para mejorar la usabilidad y permitir que el usuario pueda copiarlo, anotarlo o comunicarlo fácilmente.

Sin embargo, un espacio de 90.000 combinaciones es menor que el utilizado originalmente con códigos aleatorios largos.

Por ello, para una versión expuesta a alto volumen de tráfico se recomienda incorporar:

- rate limiting en `track-ticket`;
- monitoreo de intentos repetidos;
- límites por IP o ventana temporal;
- CAPTCHA cuando resulte necesario.

Esta consideración debe mencionarse como una decisión de compromiso entre **usabilidad y resistencia a enumeración**.

---

# 7. Flujo de creación de una incidencia

## 7.1. Datos ingresados

El formulario solicita:

- título;
- dispositivo;
- versión de Android;
- descripción;
- nombre del remitente;
- correo electrónico.

Los campos principales utilizan validación HTML mediante `required`.

## 7.2. Validación en múltiples capas

Se adoptó una validación por capas:

```text
HTML
  ↓
JavaScript
  ↓
Edge Function
  ↓
PostgreSQL
```

### Capa HTML

Controla campos requeridos y formato básico de email.

### Capa JavaScript

Evita el envío cuando el formulario no supera `reportValidity()`.

### Edge Function

Vuelve a validar:

- longitudes;
- campos obligatorios;
- formato de correo;
- método HTTP.

### Base de datos

Aplica restricciones como:

- `NOT NULL`;
- `CHECK`;
- `UNIQUE`.

La razón de duplicar ciertas validaciones es que las verificaciones del navegador pueden ser evitadas manualmente. La validación importante debe existir también del lado servidor y de la base.

---

## 7.3. Invocación de `create-ticket`

El frontend construye el objeto del reporte y llama:

```javascript
db.functions.invoke('create-ticket', {
  body: newTicket
});
```

La función:

1. recibe el JSON;
2. valida los datos;
3. genera el código `ARIAE-ENG-XXXXX`;
4. inserta el ticket;
5. asigna `status = 'pendiente'`;
6. devuelve el código con HTTP `201 Created`.

---

# 8. Confirmación visual del ticket

Originalmente el código era mostrado mediante un `alert()` del navegador.

Se reemplazó por una confirmación integrada en la interfaz.

Después de crear el ticket:

1. se limpia el formulario;
2. el formulario se oculta;
3. aparece una tarjeta de éxito;
4. se presenta el código en tamaño destacado;
5. se habilita un botón **Copiar código**;
6. se ofrece **Registrar otro reporte**.

Ejemplo:

```text
✓ Reporte enviado correctamente

Guardá este código para consultar tu incidencia.

ARIAE-ENG-48217

[ COPIAR CÓDIGO ] [ REGISTRAR OTRO REPORTE ]
```

### Justificación

Esta solución ofrece mejor experiencia de usuario que un `alert()` porque:

- permanece visible hasta que el usuario decida continuar;
- permite seleccionar y copiar el código;
- no depende de una ventana modal nativa del navegador;
- mantiene la identidad visual del sistema;
- reduce la posibilidad de cerrar accidentalmente el mensaje sin copiar el número.

Para copiar el valor se utiliza:

```javascript
navigator.clipboard.writeText(code)
```

Si el acceso al portapapeles falla, el sistema utiliza un mecanismo alternativo mostrando nuevamente el código.

---

# 9. Flujo de seguimiento de una incidencia

El usuario accede a la pestaña **Rastrear Ticket** e introduce el código.

Ejemplo:

```text
ARIAE-ENG-48217
```

El frontend:

1. elimina espacios;
2. transforma el valor a mayúsculas;
3. invoca la Edge Function `track-ticket`.

```javascript
db.functions.invoke('track-ticket', {
  body: {
    ticket_code: code
  }
});
```

## 9.1. Información pública devuelta

La función devuelve solamente datos necesarios para seguimiento:

- `ticket_code`;
- `title`;
- `device`;
- `os`;
- `description`;
- `status`;
- `admin_reply`;
- `created_at`.

No es necesario devolver al consultante datos internos como:

- `id`;
- datos administrativos adicionales.

Como criterio de minimización, los datos personales del remitente tampoco necesitan formar parte de la respuesta pública de seguimiento.

## 9.2. Posibles respuestas

### Ticket encontrado

HTTP:

```text
200 OK
```

El sistema muestra el estado y la respuesta docente.

### Ticket inexistente

HTTP:

```text
404 Not Found
```

### Formato o solicitud inválida

HTTP:

```text
400 Bad Request
```

### Error interno

HTTP:

```text
500 Internal Server Error
```

---

# 10. Estados del ticket

La interfaz muestra los estados mediante badges visuales.

| Estado de base | Presentación |
|---|---|
| `pendiente` | Pendiente |
| `en_revision` | En revisión |
| `resuelto` | Resuelto |
| `descartado` | Descartado |

El alumno puede observar el cambio de estado cuando vuelve a consultar su código.

---

# 11. Panel Docente

El panel administrativo está integrado dentro de la misma página, pero permanece protegido por autenticación.

## 11.1. Inicio de sesión

El docente introduce:

- correo;
- contraseña.

El frontend utiliza:

```javascript
db.auth.signInWithPassword({
  email,
  password
});
```

Si la autenticación es correcta:

- se oculta el formulario de acceso;
- se muestra el espacio de trabajo docente;
- se cargan los reportes.

## 11.2. Persistencia de sesión

Al recargar la página, el frontend consulta:

```javascript
db.auth.getSession()
```

Si existe una sesión válida, el panel se restaura sin solicitar nuevamente las credenciales.

También se escucha el cambio de autenticación con:

```javascript
db.auth.onAuthStateChange(...)
```

## 11.3. Cierre de sesión

El botón **Cerrar Sesión** ejecuta:

```javascript
db.auth.signOut()
```

y vuelve a ocultar el panel.

---

# 12. Carga de reportes para el docente

Una vez autenticado, el frontend consulta:

```javascript
db
  .from('tickets')
  .select('*')
  .order('created_at', { ascending: false });
```

Esta consulta sólo funciona cuando:

- existe una sesión válida;
- PostgreSQL reconoce el rol `authenticated`;
- RLS permite la operación.

Los tickets se ordenan desde el más reciente al más antiguo.

La tabla muestra:

- código;
- estado;
- problema;
- dispositivo;
- remitente;
- fecha;
- acción de gestión.

---

# 13. Gestión de un ticket

Al presionar **Gestionar**, se abre un modal.

El modal muestra:

- código;
- título;
- descripción;
- dispositivo;
- estado actual;
- respuesta oficial.

El docente puede modificar:

```text
status
admin_reply
```

El frontend ejecuta un `UPDATE` sobre el registro correspondiente.

La configuración de permisos de la base limita las columnas actualizables a esos campos.

Esto evita que desde el panel se puedan alterar accidentalmente:

- el código del ticket;
- la descripción original;
- la fecha;
- el identificador;
- otros datos enviados por el alumno.

---

# 14. Respuesta oficial

El campo `admin_reply` permite al equipo docente escribir una respuesta asociada al ticket.

Cuando el alumno vuelve a rastrear el código, observa:

- estado actualizado;
- respuesta oficial.

Si todavía no existe respuesta, la interfaz informa que el caso continúa en revisión.

---

# 15. Exportación CSV

El panel permite exportar los reportes cargados a un archivo CSV.

El archivo contiene:

- ticket;
- título;
- estado;
- dispositivo;
- remitente;
- email;
- fecha;
- respuesta.

El CSV se genera en el navegador mediante `Blob` y un enlace temporal.

No se requiere una librería externa.

Esta funcionalidad facilita:

- análisis posterior;
- elaboración de estadísticas;
- respaldo manual;
- tratamiento de datos en Excel, LibreOffice Calc u otras herramientas.

---

# 16. Protección frente a XSS

Parte de la interfaz genera contenido dinámico con `innerHTML`.

Los valores procedentes de usuarios son procesados por una función `escapeHtml()` antes de insertarse.

Se convierten caracteres como:

```text
&
<
>
"
'
```

a entidades HTML.

Esta medida reduce el riesgo de que una descripción o título introducido por un usuario sea interpretado como código HTML ejecutable.

Cuando es posible, se utiliza `textContent`, especialmente en el modal administrativo.

---

# 17. Configuración de autenticación

En Supabase Authentication se adoptó la siguiente configuración:

```text
Email                         ENABLED
Allow new users to sign up    OFF
Allow anonymous sign-ins      OFF
Confirm email                 ON
```

Los docentes se crean manualmente desde:

```text
Authentication → Users → Add user
```

### Justificación

Si el registro público permaneciera habilitado, cualquier visitante podría intentar crear una cuenta que obtuviera el rol `authenticated`.

Como la versión actual considera que los usuarios autenticados son docentes controlados, el registro público debe permanecer deshabilitado.

---

# 18. Seguridad actual y limitación conocida

En la versión implementada se utiliza la siguiente premisa:

```text
usuario authenticated = usuario docente autorizado
```

Esta premisa es válida mientras:

- el registro público continúe deshabilitado;
- las cuentas sean creadas manualmente por el administrador;
- no se utilicen cuentas autenticadas de alumnos.

## Mejora futura recomendada

En una evolución del proyecto conviene crear una tabla o claim de roles:

```text
profiles
├── user_id
└── role
```

con valores como:

```text
docente
administrador
alumno
```

Las políticas RLS podrían comprobar explícitamente el rol.

De esa manera, estar autenticado no implicaría automáticamente poseer permisos docentes.

---

# 19. Política de acceso a la tabla

El esquema implementado sigue este criterio:

```text
ANON
 └── sin acceso directo a tickets

AUTHENTICATED
 ├── SELECT tickets
 └── UPDATE únicamente status y admin_reply

EDGE FUNCTIONS
 ├── create-ticket
 └── track-ticket
```

Esta separación constituye una de las principales medidas de seguridad del sistema.

---

# 20. Funcionamiento de `create-ticket`

La función se ejecuta del lado servidor.

Responsabilidades:

1. aceptar únicamente el método esperado;
2. interpretar JSON;
3. normalizar campos;
4. validar longitudes;
5. validar correo;
6. generar código de cinco dígitos;
7. insertar en PostgreSQL;
8. resolver posibles colisiones;
9. responder con el código.

Respuesta exitosa conceptual:

```json
{
  "ticket_code": "ARIAE-ENG-48217"
}
```

HTTP:

```text
201 Created
```

---

# 21. Funcionamiento de `track-ticket`

Responsabilidades:

1. recibir el código;
2. normalizar el valor;
3. buscar coincidencia exacta en `ticket_code`;
4. devolver solamente los campos necesarios;
5. devolver 404 si no existe;
6. evitar que el navegador realice un `SELECT *` público directo.

Como endurecimiento adicional, es recomendable validar estrictamente:

```regex
^ARIAE-ENG-\d{5}$
```

y aplicar limitación de consultas.

---

# 22. Configuración de Edge Functions

Las funciones fueron creadas desde:

```text
Supabase Dashboard
→ Edge Functions
→ Deploy a new function
→ Via Editor
```

Este método fue elegido durante la implementación porque permite:

- escribir el código desde el navegador;
- desplegar sin instalar herramientas adicionales;
- probar requests desde el Dashboard;
- visualizar HTTP status, body y logs.

Para las funciones se dejó:

```text
Verify JWT with legacy secret = OFF
```

porque el proyecto utiliza el esquema moderno de publishable keys y la lógica de autenticación/autorización correspondiente dentro del flujo actual de las funciones.

Para trabajo profesional con control de versiones a largo plazo se recomienda conservar también el código fuente de las Edge Functions dentro del repositorio.

---

# 23. Estructura recomendada del repositorio

```text
Soporte/
│
├── index.html
├── app.js
├── config.js
├── README.md
├── SETUP.md
│
├── css/
│   ├── base.css
│   ├── header.css
│   ├── main.css
│   ├── footer.css
│   └── styles.css
│
├── img/
│   └── logo.png
│
└── supabase/
    ├── migrations/
    │   └── 20261022_ariae_tickets.sql
    │
    └── functions/
        ├── create-ticket/
        │   └── index.ts
        └── track-ticket/
            └── index.ts
```

---

# 24. `config.js`

El archivo contiene únicamente configuración pública:

```javascript
const CONFIG = {
  SUPABASE_URL: "https://<project-ref>.supabase.co",
  SUPABASE_PUBLISHABLE_KEY: "sb_publishable_..."
};
```

No debe contener:

- contraseñas docentes;
- PIN administrativos;
- secret keys;
- service role keys.

---

# 25. Flujo completo de punta a punta

## 25.1. Alumno crea incidencia

```text
Alumno
  ↓
Formulario HTML
  ↓
Validación navegador
  ↓
app.js
  ↓
create-ticket
  ↓
Validación servidor
  ↓
PostgreSQL
  ↓
ARIAE-ENG-48217
  ↓
Tarjeta de confirmación
```

## 25.2. Alumno consulta

```text
ARIAE-ENG-48217
  ↓
app.js
  ↓
track-ticket
  ↓
PostgreSQL
  ↓
Estado + respuesta
```

## 25.3. Docente administra

```text
Correo + contraseña
  ↓
Supabase Auth
  ↓
JWT / authenticated
  ↓
RLS
  ↓
SELECT tickets
  ↓
Panel Docente
  ↓
UPDATE status/admin_reply
```

---

# 26. Pruebas realizadas

Durante la implementación se verificó el sistema en etapas.

## Prueba de creación

La función `create-ticket` fue ejecutada desde el tester de Supabase.

Resultado esperado y obtenido:

```text
HTTP 201
ticket_code generado
```

## Prueba de rastreo

El código generado fue enviado a `track-ticket`.

Resultado:

```text
HTTP 200
ticket encontrado
```

La respuesta contenía los campos públicos esperados.

## Prueba funcional completa

El flujo a verificar en cada versión es:

1. crear ticket;
2. copiar código;
3. rastrear código;
4. iniciar sesión como docente;
5. abrir el reporte;
6. cambiar estado;
7. escribir respuesta;
8. guardar;
9. cerrar sesión;
10. volver a rastrear;
11. comprobar el nuevo estado y respuesta.

---

# 27. Matriz básica de pruebas

| Caso | Resultado esperado |
|---|---|
| Formulario válido | Ticket creado |
| Campo obligatorio vacío | No se envía |
| Email inválido | Rechazado |
| Ticket inexistente | Mensaje 404 |
| Ticket existente | Muestra estado |
| Usuario sin sesión intenta panel | No obtiene datos |
| Docente válido inicia sesión | Panel visible |
| Cambio a `en_revision` | Actualización persistida |
| Respuesta docente | Visible al rastrear |
| Cerrar sesión | Panel protegido nuevamente |
| Exportar con datos | Descarga CSV |
| Copiar código | Portapapeles recibe ticket |

---

# 28. Decisiones de diseño relevantes

## 28.1. Backend administrado en vez de servidor propio

No se creó un servidor Node.js, PHP o Java propio.

La elección de Supabase reduce tareas de infraestructura como:

- instalación del motor de base;
- mantenimiento del servidor;
- administración de sesiones manual;
- creación de una API REST desde cero;
- despliegue de procesos backend tradicionales.

Para un proyecto académico, esto permite concentrar el trabajo en la arquitectura, seguridad y experiencia de usuario.

---

## 28.2. PostgreSQL en lugar de almacenamiento local

No se utilizó `localStorage` como base de datos porque:

- sólo existe en un navegador;
- no permite compartir tickets entre alumnos y docentes;
- puede ser borrado por el usuario;
- no proporciona control de acceso centralizado;
- no ofrece integridad relacional.

PostgreSQL permite persistencia central y validaciones de servidor.

---

## 28.3. Edge Functions en lugar de acceso anónimo directo

Aunque Supabase permite acceder a la Data API desde el navegador, se evitó dar a `anon` permisos generales sobre `tickets`.

Las operaciones públicas atraviesan endpoints específicos.

Esto permite controlar:

- qué recibe el servidor;
- qué se inserta;
- qué se devuelve;
- qué errores se exponen.

---

## 28.4. Auth real en lugar de PIN en JavaScript

Un PIN escrito en un archivo JavaScript público puede inspeccionarse.

Supabase Auth desplaza la verificación de credenciales a un sistema especializado.

Esto constituye una mejora arquitectónica y no solamente visual.

---

## 28.5. RLS en lugar de confiar en la interfaz

Ocultar un botón no equivale a proteger un recurso.

La seguridad se colocó en la base de datos mediante permisos y RLS.

La interfaz constituye la experiencia del usuario; PostgreSQL constituye la barrera de autorización.

---

## 28.6. Código corto de seguimiento

Se adoptó `ARIAE-ENG-XXXXX` para mejorar:

- legibilidad;
- memorización;
- transcripción;
- comunicación oral;
- copia manual.

Se reconoce el menor espacio de búsqueda y se deja rate limiting como mejora de seguridad.

---

# 29. Ventajas de la arquitectura

- interfaz simple y de carga rápida;
- frontend desacoplado del backend;
- base relacional;
- autenticación administrada;
- autorización en base de datos;
- funciones serverless;
- posibilidad de despliegue sin servidor tradicional;
- mantenimiento modular;
- registro centralizado de incidencias;
- trazabilidad mediante código;
- exportación de información;
- posibilidad de evolución futura.

---

# 30. Limitaciones y trabajo futuro

## 30.1. Rate limiting

Debe incorporarse sobre `track-ticket`, especialmente después de adoptar códigos de cinco dígitos.

## 30.2. Roles explícitos

Separar:

```text
docente
administrador
alumno
```

en lugar de considerar a todo `authenticated` como docente.

## 30.3. Auditoría

Registrar:

- quién modificó un ticket;
- fecha del cambio;
- estado anterior;
- estado nuevo.

## 30.4. Historial

Crear una tabla:

```text
ticket_history
```

para conservar el historial completo de acciones.

## 30.5. Notificaciones

Podría agregarse envío de email cuando:

- se crea un ticket;
- cambia de estado;
- recibe una respuesta.

## 30.6. Métricas

Agregar un dashboard con:

- tickets por estado;
- tiempo promedio de resolución;
- dispositivos con más incidencias;
- versiones de Android con mayor frecuencia de errores.

## 30.7. Control de acceso más granular

Utilizar claims, tabla de perfiles o RBAC para distinguir roles.

## 30.8. Versionado de Edge Functions

Aunque las funciones se pueden editar desde el Dashboard, sus fuentes deben conservarse en Git para poder auditar cambios y volver a versiones anteriores.

---

# 31. Consideraciones de privacidad

Los reportes pueden contener:

- nombre;
- correo;
- descripción del problema;
- información técnica del dispositivo.

Por ello:

- el panel docente debe permanecer autenticado;
- el rastreo público debe devolver únicamente información necesaria;
- no deben exponerse listas completas de tickets;
- las claves secretas no deben publicarse;
- deben revisarse periódicamente las políticas RLS.

---

# 32. Conceptos aplicados durante el desarrollo

El proyecto permite aplicar de manera práctica conocimientos de:

### Desarrollo web

- HTML semántico;
- CSS modular;
- responsive design;
- DOM;
- eventos;
- asincronía con `async/await`.

### Bases de datos

- PostgreSQL;
- modelado de tablas;
- UUID;
- restricciones;
- índices;
- unicidad;
- seguridad a nivel de fila.

### Backend

- endpoints HTTP;
- JSON;
- códigos de estado;
- validación de entrada;
- funciones serverless.

### Seguridad

- autenticación;
- autorización;
- mínimo privilegio;
- separación público/privado;
- XSS;
- manejo de claves;
- defensa en profundidad.

### Ingeniería de software

- separación de responsabilidades;
- modularización;
- pruebas de integración;
- despliegue;
- control de versiones;
- documentación técnica.

---

# 33. Texto sugerido para incorporar a una tesis

> Para la implementación del módulo web de soporte de ARIAE se adoptó una arquitectura desacoplada compuesta por un frontend estático y un backend administrado. La capa de presentación fue desarrollada utilizando HTML5, CSS3 y JavaScript, y se publicó mediante GitHub Pages. Esta alternativa resultó adecuada debido a que la interfaz no requiere procesamiento propio del lado del servidor y puede ser distribuida directamente a partir de los archivos versionados en el repositorio.
>
> La persistencia y los servicios de backend fueron implementados mediante Supabase. La plataforma proporciona una base de datos PostgreSQL, autenticación, políticas Row Level Security y funciones de servidor, lo que permitió concentrar en una misma solución la gestión de datos, usuarios y autorización. La elección de PostgreSQL permitió aplicar restricciones de integridad directamente sobre los datos, evitando depender exclusivamente de validaciones realizadas por el navegador.
>
> Las operaciones públicas de creación y consulta de incidencias fueron encapsuladas en Edge Functions. Esta decisión permitió evitar la asignación de permisos generales de lectura o escritura al rol anónimo sobre la tabla principal. De esta manera, el cliente web realiza solicitudes a endpoints con responsabilidades específicas, mientras que las funciones controlan los datos aceptados y la información que puede ser devuelta al usuario.
>
> Para el acceso docente se utilizó Supabase Auth mediante correo electrónico y contraseña. Inicialmente se había considerado un PIN validado en el navegador; sin embargo, dicho enfoque fue descartado debido a que cualquier valor incluido en JavaScript público puede ser inspeccionado. La autenticación administrada, combinada con RLS, permitió trasladar la decisión de acceso a una capa de servidor.
>
> La tabla de incidencias posee Row Level Security habilitado y aplica el principio de mínimo privilegio. El acceso público directo fue revocado, mientras que las cuentas docentes autenticadas pueden consultar los reportes y modificar únicamente los campos correspondientes al estado y a la respuesta oficial. Como consecuencia, la protección de los datos no depende de los controles visuales de la interfaz.
>
> Cada incidencia recibe un código público con el formato `ARIAE-ENG-XXXXX`. El uso de cinco dígitos se adoptó para facilitar su lectura, copia y comunicación por parte de los alumnos. La unicidad se garantiza mediante una restricción de PostgreSQL y un mecanismo de reintento en la función de creación. Debido a que este formato genera un espacio finito de combinaciones, se identifica como mejora futura la incorporación de mecanismos de rate limiting para reducir intentos automatizados de enumeración.
>
> Finalmente, el sistema permite al docente visualizar las incidencias, actualizar su estado, proporcionar una respuesta oficial y exportar la información en formato CSV. El alumno puede volver a consultar su código y observar el estado actualizado. Este flujo permitió implementar un ciclo completo de reporte, gestión y retroalimentación, manteniendo separadas las responsabilidades del frontend, la lógica de servidor y la base de datos.

---

# 34. Referencias técnicas oficiales

## GitHub

GitHub Pages — documentación oficial  
https://docs.github.com/pages/getting-started-with-github-pages/what-is-github-pages

## Supabase

Documentación general  
https://supabase.com/docs

Edge Functions  
https://supabase.com/docs/guides/functions

Edge Functions desde Dashboard  
https://supabase.com/docs/guides/functions/quickstart-dashboard

Auth  
https://supabase.com/docs/guides/auth

Autenticación mediante contraseña  
https://supabase.com/docs/guides/auth/passwords

Row Level Security  
https://supabase.com/docs/guides/database/postgres/row-level-security

Seguridad de datos  
https://supabase.com/docs/guides/database/secure-data

API Keys  
https://supabase.com/docs/guides/getting-started/api-keys

---

# 35. Resumen

La arquitectura final de ARIAE combina:

```text
GitHub Pages
    +
HTML / CSS / JavaScript
    +
Supabase PostgreSQL
    +
Supabase Auth
    +
Row Level Security
    +
Edge Functions
```

El resultado es un sistema de soporte que:

- permite reportar incidencias;
- genera códigos de seguimiento;
- presenta una confirmación dentro de la interfaz con opción de copia;
- permite rastrear el estado;
- autentica al equipo docente;
- restringe operaciones mediante RLS;
- permite responder incidencias;
- permite exportar reportes;
- mantiene separadas las operaciones públicas de las administrativas.

La solución fue diseñada buscando un equilibrio entre simplicidad de despliegue, seguridad, mantenibilidad y facilidad de uso en un contexto académico.
