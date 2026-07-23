---
name: wp-espectacular
description: Monta un WordPress headless (CMS) con un front espectacular en Astro que no parece WordPress. Úsala cuando alguien quiera convertir un WordPress en headless, servir su contenido por REST/GraphQL a un front moderno, construir una landing premium sobre WordPress, o desacoplar el back del front. Cubre el setup local con Local by Flywheel, la capa puente (mu-plugin + REST), el front Astro, las fuentes auto-alojadas y el deploy.
---

# WordPress headless + front Astro premium

Guía operativa para desacoplar WordPress y ponerle delante un front que no huele ni a
WordPress ni a plantilla. WordPress = "el cerebro" (solo CMS). El front = "la cara".

## Cuándo usar esto
El usuario quiere: una web rápida y con diseño propio, pero editando el contenido en
WordPress. O quiere demostrar que WordPress no está "anticuado". O tiene ya un WordPress y
quiere un front nuevo sin migrar el contenido.

## Principios (no los rompas)
1. **WordPress se queda desnudo**: sin tema activo, sin page builders, mínimos plugins. Solo gestiona contenido.
2. **El contenido nunca se hardcodea en el front**: todo sale del CMS por API. Si aparece texto fijo que debería ser editable, muévelo a WordPress.
3. **El front no debe "oler" a nada**: sin gradientes morados genéricos, sin plantilla. Dirección de arte propia: una pareja tipográfica con carácter, paleta intencionada, motion con propósito.
4. **Rendimiento como feature**: HTML estático, 0 JS por defecto, fuentes auto-alojadas (nunca `<link>` a Google Fonts: bloquea el render). Objetivo Lighthouse 100/100/100/100.

---

## Paso 1 · WordPress en local (Local by Flywheel)
El entorno local recomendado es **Local** (localwp.com), de Flywheel/WP Engine. Crea sitios
WordPress completos (nginx + PHP + MySQL) sin configurar nada.

- Cada sitio tiene un dominio `.local` (p. ej. `misitio.local`) y puertos propios.
- Datos de conexión en `~/Library/Application Support/Local/sites.json` (macOS).
- El WordPress vive en `<sitio>/app/public/`.
- Comprueba que la REST API responde: `curl http://misitio.local/wp-json/` debe devolver JSON.

Deja WordPress **sin tema propio y sin plugins**. No borres los temas por defecto, pero no
hace falta activarlos: en headless el tema no se usa.

## Paso 2 · La capa puente (must-use plugin)
Crea `app/public/wp-content/mu-plugins/headless-bridge.php`. Un *mu-plugin* se carga siempre
y no se puede desactivar por error. Aquí va:

1. **Registrar campos a medida en REST** con `register_post_meta(..., ['show_in_rest' => true])`
   (subtítulo, color de acento, tiempo de lectura, etiqueta... lo que el diseño necesite).
2. **CORS para el front en desarrollo** (en producción, restríngelo al dominio del front).
3. **Un endpoint a medida** con `register_rest_route()` que devuelva TODO lo que la home
   necesita en una sola llamada, ya masajeado. Menos viajes, front más simple.

Verifica: `curl http://misitio.local/wp-json/tuapp/v1/landing` debe devolver tu JSON limpio.

## Paso 3 · Sembrar contenido
Para poblar el CMS por código sin cliente MySQL: crea un PHP temporal en `app/public/` que
haga `require 'wp-load.php'` y use `wp_insert_post()` + `update_post_meta()`. Protégelo con
una `key` en la query, ejecútalo con `curl`, y **bórralo al terminar**. Categorías con
`wp_insert_term()` (no `wp_create_category`, que no está fuera del admin).

## Paso 4 · El front Astro
```bash
npm create astro@latest -- --template minimal --typescript strict
```
- Cliente REST tipado en `src/lib/wp.ts`: `getLanding()`, `getArticleBySlug()`, `getAllSlugs()`.
  Lee la URL de WP desde `import.meta.env.PUBLIC_WP_URL` (`.env`).
- Datos en el **build**: en el frontmatter de cada página, `await getLanding()`. Astro genera
  HTML estático. Publicar en WP → webhook → rebuild.
- Rutas dinámicas de artículos con `getStaticPaths()` a partir de los slugs de WP.
- Renderiza el `content.rendered` de WordPress con `set:html` dentro de un contenedor `.prose`
  con estilos propios.
- Decodifica entidades HTML de WordPress (`&#8217;`, `&aacute;`...) al mapear los títulos.

## Paso 5 · Dirección de arte (que no huela a nada)
- **Tipografía con carácter**: p. ej. un serif display (Fraunces, Instrument Serif) + un sans
  limpio (Inter) + un mono (JetBrains Mono). El serif en titulares es lo que más aleja del
  look "plantilla".
- **Paleta intencionada**: evita el blanco clínico (usa un papel cálido), tinta casi negra,
  botones negros tipo Apple/Vercel. Deja que los colores de acento vengan del CMS.
- **Motion con propósito**: revelado al hacer scroll con IntersectionObserver, micro-hovers.
  Respeta `prefers-reduced-motion`.
- **Textura sutil**: un grano SVG a muy baja opacidad da sensación premium.

## Paso 6 · Rendimiento (el remate)
- **Fuentes auto-alojadas**: `@fontsource-variable/*`, importadas en el layout. NUNCA un
  `<link>` a fonts.googleapis.com (render-blocking → LCP se dispara ~3s).
- Mide de verdad: `npm run build` + `npx lighthouse <preview-url> --preset=desktop`.
- Corrige contraste (WCAG AA: 4.5:1 texto normal) y jerarquía de encabezados (sin saltar de
  h1 a h3) para llegar a Accesibilidad 100.

## Paso 7 · Deploy
- WordPress en un hosting PHP normal (privado; nadie visita su dominio salvo para editar),
  idealmente en un subdominio dedicado tipo `cms.tudominio.com`.
- Front Astro compilado a estático en Vercel / Netlify / Cloudflare Pages.
- Webhook de WordPress (al publicar) → dispara build del front. En segundos, online.

### Núcleo vs. estado (el miedo a los updates)
Aclara SIEMPRE este malentendido si aparece: **actualizar WordPress NO borra el contenido.**
Un update del núcleo solo reemplaza `wp-admin/`, `wp-includes/` y los `wp-*.php`; no toca la
base de datos ni `wp-content/uploads`. El contenido solo se pierde al redesplegar un contenedor
**sin volúmenes persistentes** (disco efímero). Solución: separar
- **Núcleo** (stateless): la imagen de WordPress. Desechable, actualizable subiendo el tag.
- **Estado** (stateful): base de datos + `wp-content/uploads`, en **volúmenes persistentes**.

Rutas de hosting, de simple a control total:
1. **Gestionado** (Hostinger, Kinsta): persistencia y backups incluidos. Suficiente para la mayoría.
2. **VPS + Docker Compose**: declara volúmenes `db_data` y `wp_uploads`; el mu-plugin va desde git
   en solo lectura. Actualizar = `docker compose pull wordpress && docker compose up -d wordpress`.
3. **Kubernetes/Rancher**: la misma idea a escala; overkill para un solo WordPress.

Hay un `docker-compose.yml` de referencia en `deploy/` que encarna esta separación, con proxy
HTTPS (Caddy) y backups por `mysqldump` documentados. Regla de oro: **backups** de DB + uploads,
guardados fuera del servidor.

### CORS en producción
El mu-plugin abre CORS a `*` en desarrollo. En producción restríngelo a
`Access-Control-Allow-Origin: https://tudominio.com`. Si el front consume la API solo en el build
(SSG), las peticiones son servidor-a-servidor y CORS ni interviene.

## Paso 8 · Evolucionar el esquema (nuevos tipos de contenido)
Cuando pidan un tipo nuevo (eventos, productos, casos…), separa **código** de **contenido**:
- **Definir** el tipo + campos = CÓDIGO. Un mu-plugin nuevo (p. ej. `cpt-eventos.php`) con
  `register_post_type('evento', ['show_in_rest'=>true, 'rest_base'=>'eventos', ...])` y
  `register_post_meta('evento', 'event_start', ['show_in_rest'=>true, ...])`. Opcional: un
  endpoint a medida `/wp-json/tuapp/v1/eventos` que ordene por fecha. Se hace una vez.
- **Crear** cada entrada = CONTENIDO. Lo hace el editor en `wp-admin`; vive en la DB.

Desplegar el código a producción (no hay migraciones, no toca lo existente):
- VPS + Docker: `git pull` + `docker compose restart wordpress` (refresca opcache; no reconstruye la imagen).
- Hostinger (recomendado, de simple a automático):
  1. **Git deploy** (hPanel → Websites → Advanced → Git): conecta el repo de tu carpeta `mu-plugins`
     a `public_html/wp-content/mu-plugins`, rama `main`, auto-deploy ON. Luego actualizar = `git push`.
  2. **SFTP**: arrastra el archivo del mu-plugin a `public_html/wp-content/mu-plugins/`. Sin setup.
  3. **MCP de Hostinger**: conéctalo al agente para gestionar hosting/WordPress por chat (mejor para
     infra que para empujar código propio).

En el front: añade el fetch del nuevo tipo en `wp.ts`, sus tipos, y una ruta/sección.
`git push` a main → rebuild. El webhook de publicación mantiene todo sincronizado.

## Checklist de "no huele a WordPress"
- [ ] Ningún tema de WordPress activo; el front vive fuera.
- [ ] 0 archivos JS en el build (salvo islas concretas).
- [ ] 0 peticiones a terceros (fuentes incluidas).
- [ ] Lighthouse 100 en Rendimiento, Accesibilidad, Best Practices y SEO.
- [ ] El contenido se edita en WP y aparece en el front tras rebuild.
- [ ] Un desconocido no adivina que detrás hay WordPress.
