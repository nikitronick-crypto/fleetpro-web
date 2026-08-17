# FleetPro · sitio público

**Operador de la plataforma:** Nicolás Vidal Gómez · `fleetpro.soporte@gmail.com`
**Mecánica Curicó Spa** es una flota usuaria, como cualquier otra: sus datos son suyos
y su correo no aparece en el sitio ni en los correos del sistema.

Sitio estático en GitHub Pages. Cubre tres cosas: la confirmación de correo y el cambio
de contraseña con la marca puesta, y los dos documentos que Google Play exige —política
de privacidad y eliminación de cuenta— publicados en una URL.

```
index.html            Presentación de la app
confirmado.html       Aterrizaje tras confirmar el correo
nueva-clave.html      Formulario de contraseña nueva
privacidad.html       Política de privacidad
eliminar-cuenta.html  Cómo eliminar la cuenta
404.html              Página no encontrada
assets/               Escudo, hoja de estilos y credenciales públicas
correos/              Plantillas HTML para los correos de Supabase
```

---

## 1. Publicar el sitio

Crea un repositorio **público** llamado `fleetpro-web` y sube el contenido de esta
carpeta a la raíz.

```powershell
cd fleetpro-web
git init
git add .
git commit -m "Sitio publico de FleetPro"
git branch -M main
git remote add origin https://github.com/nikitronick-crypto/fleetpro-web.git
git push -u origin main
```

En el repositorio: **Settings → Pages → Source: Deploy from a branch → main / (root)**.
En un par de minutos queda en:

```
https://nikitronick-crypto.github.io/fleetpro-web/
```

El repositorio tiene que ser público para que Pages funcione en el plan gratuito. No hay
problema: aquí no vive ningún secreto.

## 2. Poner la anon key

Abre `assets/config.js` y reemplaza `PEGA_AQUI_TU_ANON_KEY` por la clave de tu proyecto.

Esa clave está hecha para vivir en el cliente —también va dentro del APK—; lo que
protege los datos son las políticas RLS del servidor, no esconderla.

## 3. Configurar Supabase

**Authentication → URL Configuration**

| Campo | Valor |
|---|---|
| Site URL | `https://nikitronick-crypto.github.io/fleetpro-web` |
| Redirect URLs | `https://nikitronick-crypto.github.io/fleetpro-web/**` |

Sin los dos asteriscos, Supabase rechaza las redirecciones a páginas internas.

**Authentication → Emails → Templates**

Para cada plantilla, pega el HTML correspondiente y ajusta el asunto:

| Plantilla | Archivo | Asunto sugerido |
|---|---|---|
| Confirm signup | `correos/confirmar.html` | Confirma tu correo · FleetPro |
| Reset password | `correos/recuperar.html` | Restablecer tu contraseña · FleetPro |
| Invite user | `correos/invitacion.html` | Te invitaron a una flota · FleetPro |

En **Reset password**, cambia además la URL de redirección a:

```
https://nikitronick-crypto.github.io/fleetpro-web/nueva-clave.html
```

y en **Confirm signup**:

```
https://nikitronick-crypto.github.io/fleetpro-web/confirmado.html
```

Si el editor de plantillas no ofrece ese campo, agrégalo al enlace:
`{{ .ConfirmationURL }}&redirect_to=https://.../nueva-clave.html`

## 4. Probar

1. Crea una cuenta de prueba con un correo real y confirma desde el mensaje.
   Debe aterrizar en `confirmado.html` con el escudo.
2. En la app, «Olvidé mi contraseña». El correo debe llevarte a `nueva-clave.html`,
   dejarte cambiarla, y la nueva debe funcionar en la app.
3. Abre `nueva-clave.html` directamente, sin enlace: debe decir que falta el código,
   no quedarse en blanco.

---

## Detalles que importan

**El token nunca sale del navegador.** El enlace de recuperación trae el código en el
fragmento de la URL, esa parte que va después del `#`. Los navegadores no la envían al
servidor, así que GitHub nunca lo ve. La página lo lee, lo manda directo a Supabase y lo
borra de la barra de direcciones al terminar.

**Los enlaces caducan.** Los de confirmación duran 24 horas; los de recuperación, una
hora y un solo uso. Las páginas detectan el caso y lo dicen con claridad en vez de
fallar en silencio.

**Correos desde el dominio de Supabase.** Por defecto salen de `noreply@mail.app.supabase.io`,
lo que puede caer en spam y tiene un límite de envíos bajo. Para producción conviene
conectar un servicio de correo propio en Authentication → SMTP Settings; con un dominio
propio verificado, los mensajes llegan a la bandeja principal.

**No hay administrador global.** Las políticas RLS te dejan ver solo las flotas de las
que eres miembro, y eso te incluye a ti como operador: entrando por la app no ves los
datos de otras flotas. El acceso para soporte es por el panel de Supabase, y está
declarado en la política de privacidad. Si alguna vez quieres un rol de soporte dentro
de la app, hay que diseñarlo aparte y decirlo en la política.

**La política de privacidad menciona cosas concretas** —Supabase sobre AWS, plazo de 30
días para borrar, sin publicidad ni rastreo—. Si algo de eso cambia, hay que actualizar
el texto: en el cuestionario de Play se declara lo mismo, y las dos versiones tienen que
coincidir.
