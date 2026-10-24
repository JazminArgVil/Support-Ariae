# ARIAE — Guía de instalación y configuración

Esta guía describe el procedimiento utilizado en el proyecto ARIAE utilizando principalmente el **Dashboard de Supabase**. La Supabase CLI es opcional y no es necesaria para reproducir la configuración básica.

---

# 1. Requisitos

- Cuenta de GitHub.
- Repositorio para el frontend.
- Proyecto en Supabase.
- Archivos:
  - `index.html`
  - `app.js`
  - `config.js`
  - carpeta `css/`
  - carpeta `img/`
- Código de las Edge Functions:
  - `create-ticket`
  - `track-ticket`

---

# 2. Base de datos

Abrir:

```text
Supabase
→ SQL Editor
→ New query
```

Ejecutar la migración del proyecto que crea `public.tickets`.

La tabla debe contener como mínimo:

```text
id
ticket_code
title
device
os
description
reporter_name
reporter_email
status
admin_reply
created_at
```

## Advertencia

Si el script contiene:

```sql
drop table if exists public.tickets cascade;
```

se eliminarán los tickets existentes.

No ejecutar un `DROP` sobre datos que deban conservarse. En producción debe utilizarse una migración no destructiva.

---

# 3. Seguridad de la tabla

RLS debe estar habilitado:

```sql
alter table public.tickets enable row level security;
```

El rol público no debe acceder directamente:

```sql
revoke all on table public.tickets from anon;
```

Para el modelo actual, los usuarios autenticados son docentes creados manualmente.

Se habilita:

```text
SELECT
UPDATE de status y admin_reply
```

No se debe entregar acceso administrativo general al navegador.

---

# 4. Configurar Supabase Auth

Abrir:

```text
Authentication
→ Sign In / Providers
```

Configuración utilizada:

```text
Allow new users to sign up     OFF
Allow manual linking           OFF
Allow anonymous sign-ins       OFF
Confirm email                  ON
Email                          ENABLED
```

Guardar cambios.

---

# 5. Crear usuario docente

Abrir:

```text
Authentication
→ Users
→ Add user
```

Crear manualmente:

```text
correo docente
contraseña segura
```

No guardar la contraseña en `config.js`, `app.js` ni en GitHub.

---

# 6. Crear Edge Function `create-ticket`

Abrir:

```text
Edge Functions
→ Deploy a new function
→ Via Editor
```

Nombre:

```text
create-ticket
```

La función debe:

- aceptar POST;
- recibir JSON;
- validar campos;
- generar `ARIAE-ENG-XXXXX`;
- insertar el ticket;
- devolver HTTP 201 y `ticket_code`;
- reintentar si existe una colisión UNIQUE.

Después:

```text
Deploy function
```

---

# 7. Configuración JWT de `create-ticket`

Abrir:

```text
create-ticket
→ Settings
```

Dejar:

```text
Verify JWT with legacy secret = OFF
```

Guardar.

---

# 8. Crear Edge Function `track-ticket`

Repetir:

```text
Edge Functions
→ Deploy a new function
→ Via Editor
```

Nombre:

```text
track-ticket
```

La función debe:

- aceptar POST;
- recibir `ticket_code`;
- buscar coincidencia exacta;
- devolver sólo campos de seguimiento;
- devolver 404 cuando el ticket no existe.

Para la versión de cinco dígitos se recomienda validar:

```regex
^ARIAE-ENG-\d{5}$
```

Desplegar y dejar también:

```text
Verify JWT with legacy secret = OFF
```

---

# 9. Probar `create-ticket`

Abrir:

```text
Edge Functions
→ create-ticket
→ Test
```

Método:

```text
POST
```

Headers:

```text
Content-Type: application/json
apikey: sb_publishable_...
```

Body:

```json
{
  "title": "Prueba de conexión Supabase",
  "device": "Samsung S23",
  "os": "Android 14",
  "description": "Ticket de prueba para verificar el sistema.",
  "reporter_name": "Usuario de prueba",
  "reporter_email": "prueba@example.com"
}
```

Resultado esperado:

```text
HTTP 201
```

Body aproximado:

```json
{
  "ticket_code": "ARIAE-ENG-48217"
}
```

---

# 10. Probar `track-ticket`

Abrir:

```text
Edge Functions
→ track-ticket
→ Test
```

Método:

```text
POST
```

Headers:

```text
Content-Type: application/json
apikey: sb_publishable_...
```

Body:

```json
{
  "ticket_code": "ARIAE-ENG-48217"
}
```

Resultado esperado:

```text
HTTP 200
```

La respuesta debe contener estado e información de seguimiento.

---

# 11. Configurar el frontend

`config.js` debe contener solamente datos públicos:

```javascript
const CONFIG = {
  SUPABASE_URL: "https://<project-ref>.supabase.co",
  SUPABASE_PUBLISHABLE_KEY: "sb_publishable_..."
};
```

## Nunca colocar

```text
sb_secret_...
contraseñas
PIN administrativos
service_role
```

en el frontend.

---

# 12. Publicar en GitHub Pages

Subir al repositorio:

```text
index.html
app.js
config.js
css/
img/
README.md
```

Configurar GitHub Pages según la estructura del repositorio.

GitHub Pages publica archivos HTML, CSS y JavaScript estáticos.

---

# 13. Prueba de punta a punta

Realizar en este orden:

1. Abrir la página.
2. Ir a `Reportar Bug`.
3. Completar el formulario.
4. Presionar `Enviar Reporte`.
5. Confirmar que aparezca la tarjeta de éxito.
6. Confirmar un código `ARIAE-ENG-XXXXX`.
7. Presionar `Copiar código`.
8. Ir a `Rastrear Ticket`.
9. Consultar el código.
10. Confirmar estado `Pendiente`.
11. Ir a `Panel Docente`.
12. Iniciar sesión.
13. Confirmar que el ticket aparezca.
14. Presionar `Gestionar`.
15. Cambiar a `En revisión`.
16. Escribir una respuesta.
17. Guardar.
18. Cerrar sesión.
19. Rastrear nuevamente el código.
20. Confirmar estado y respuesta actualizados.

---

# 14. Si falla `create-ticket`

Revisar:

```text
Edge Functions
→ create-ticket
→ Logs
```

Comprobar:

- función desplegada;
- JWT legacy OFF;
- publishable key correcta;
- tabla `tickets` existente;
- columnas correctas;
- restricciones de base.

---

# 15. Si falla `track-ticket`

Comprobar:

- función desplegada;
- código exacto;
- formato de cinco dígitos;
- registro existente;
- logs;
- que la función esté actualizada para el formato `ARIAE-ENG-XXXXX`.

---

# 16. Si falla el Panel Docente

Comprobar:

- usuario creado en Authentication;
- registro público deshabilitado;
- credenciales correctas;
- RLS habilitado;
- `authenticated` con permiso SELECT;
- políticas SELECT activas;
- sesión válida.

---

# 17. Si no permite guardar cambios

Comprobar:

- permiso UPDATE;
- columnas `status` y `admin_reply`;
- policy UPDATE;
- ticket existente;
- sesión autenticada.

---

# 18. Supabase CLI — uso opcional

La CLI **no es necesaria** para el procedimiento utilizado durante la implementación.

Puede utilizarse posteriormente para:

- versionar migraciones;
- trabajar localmente;
- descargar Edge Functions;
- desplegar desde terminal;
- integrar CI/CD.

Ejemplo opcional:

```bash
npm install supabase --save-dev
npx supabase login
npx supabase init
npx supabase link --project-ref <PROJECT_REF>
```

La implementación actual puede administrarse completamente mediante el Dashboard.

---

# 19. Mejoras recomendadas antes de una publicación de mayor escala

- rate limiting en `track-ticket`;
- roles explícitos de usuario;
- historial de cambios;
- logs de auditoría;
- notificaciones por email;
- CAPTCHA ante abuso;
- métricas de soporte;
- backups y políticas de retención;
- mantener Edge Functions versionadas en Git.

---

# 20. Documentación oficial

GitHub Pages  
https://docs.github.com/pages/getting-started-with-github-pages/what-is-github-pages

Supabase  
https://supabase.com/docs

Edge Functions Dashboard  
https://supabase.com/docs/guides/functions/quickstart-dashboard

Auth  
https://supabase.com/docs/guides/auth

RLS  
https://supabase.com/docs/guides/database/postgres/row-level-security

API Keys  
https://supabase.com/docs/guides/getting-started/api-keys
