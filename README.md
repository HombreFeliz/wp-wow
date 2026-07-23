# wp-wow

Una **skill para agentes de IA** (Claude Code, Cursor, etc.) que enseña a montar un
**WordPress headless** con un front **espectacular** en Astro — uno que no huele ni a
WordPress ni a plantilla. Pásasela a tu agente y sabe guiarte (o construirlo) paso a paso:
desde el WordPress en local hasta el deploy, con rendimiento 100/100 en Lighthouse.

> Es la "guía viva" detrás del proyecto **Espectacular**: el sitio de demostración se
> construye siguiendo exactamente esta skill.

## ¿Qué es una skill?

Un archivo de instrucciones (`SKILL.md`) que tu agente lee y aplica. No es código que se
ejecute ni un plugin de WordPress: es conocimiento empaquetado que viaja con tu proyecto.

## Instalación

### Claude Code (u otros agentes con soporte de skills)

```bash
npx skills add HombreFeliz/wp-wow
```

Tu Claude Code detecta la skill automáticamente. Invócala por su nombre:

```
/wp-wow
```

¿Prefieres hacerlo a mano? Clona el repo y copia la carpeta:

```bash
git clone https://github.com/HombreFeliz/wp-wow
cp -r wp-wow/.claude/skills/wp-wow .claude/skills/
```

### Cualquier otro agente (Cursor, Windsurf, ChatGPT…)

Copia el contenido de `SKILL.md` en las reglas/instrucciones de tu proyecto, o simplemente
pásaselo al agente y pídele que lo siga.

## Úsala

Una vez instalada, arranca con este prompt:

```text
Monta un WordPress headless con un front Astro premium siguiendo la
skill wp-wow. Empieza por levantar el CMS en local y verificar
que /wp-json responde, y ve explicándome cada paso.
```

## Qué cubre

- WordPress como CMS "desnudo" (sin tema, mínimos plugins).
- La capa puente: `mu-plugin` + REST API + endpoints a medida.
- El front Astro: datos en el build, 0 JS por defecto, fuentes auto-alojadas.
- Dirección de arte premium (que no parezca WordPress).
- Rendimiento: objetivo Lighthouse 100/100/100/100.
- Deploy: núcleo vs. estado persistente, Hostinger / VPS+Docker, actualizar sin miedo.
- Evolucionar el esquema (nuevos tipos y campos) ya en producción.

## Licencia

MIT — úsala, cópiala y adáptala libremente.
