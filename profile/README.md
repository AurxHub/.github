# Aurx

Mentalidad, dinero, salud, relaciones. Todo se puede aprender; lo difícil es
saber qué, en qué orden y de quién. Aurx da el camino, las herramientas y una
comunidad de gente que quiere llegar más lejos.

Esto **no es una agencia**. Es un producto propio con cuatro piezas y una sola
cuenta, y este directorio es el monorepo donde vive entero: las aplicaciones, la
web pública, el área privada y el diseño que comparten.

---

## Las cuatro piezas

Una sola idea debajo: aquí no se puede fingir. Lo que se enseña en la comunidad
no lo escribe nadie — lo registró la aplicación mientras esa persona la usaba.

| Pieza | Qué es | Dónde vive |
|---|---|---|
| **The Club** | La comunidad, y el centro. Un chat en directo, no un foro: gente construyendo con lo que hace delante. | `app.aurx.club/club` — servicio aparte, autohospedado |
| **The Kit** | Las seis herramientas y Aura. La base. | `app.aurx.club/<app>` |
| **The Campus** | Los cursos y el programa largo. Cada uno trae la app que lo ejecuta. | `app.aurx.club` (aún sin aula) |
| **The Podcast** | Conversaciones con quien ya construyó algo. La puerta de entrada. | `app.aurx.club`, y público |

Los cuatro nombres son **en inglés en los dos idiomas** y sus slugs son
idénticos (`/campus`, `/en/campus`): son nombres propios, y los nombres propios
no se traducen. *Aurx* es el paraguas y se queda fuera del día a día — se dice
«The Club», no «Aurx Club», igual que se dice CashFlow y no Aurx CashFlow.

El Club se entra gratis. Por encima hay cinco niveles —**Member, Pro, Builder,
Founder, Council**— y una regla que los ordena: **la membresía incluye producto,
y el producto nunca incluye membresía**. Las cifras viven en un único sitio,
`site/frontend/content/pricing.ts`, y en ningún otro.

---

## Dos dominios, y la frontera entre ellos

**`aurx.club` decide fuera del login. `app.aurx.club` decide dentro.**

`aurx.club` es la web pública (`site/`): responde a desconocidos, vive en
Google y cambia todos los días de un lanzamiento. Es oscura, fotográfica y no se
parece a las apps a propósito.

`app.aurx.club` es **el origen único** de todo lo privado (`suite/`): la casa en
la raíz y cada app bajo su ruta. No hay subdominios — `cashflow.cinkit.com` y
sus seis hermanos dejaron de existir el 2026-08-21.

```
app.aurx.club/           la casa: el Club, el Campus, el Podcast y el Kit
app.aurx.club/auth       Iris: entrar, registrarse, la cuenta
app.aurx.club/o/         el servidor OAuth de Iris (clientes MCP)
app.aurx.club/cashflow   ·  /pulso  ·  /yurnal  ·  /digest  ·  /rumbo
app.aurx.club/aura       redirige a la casa; su API sigue viva bajo /aura/api
```

Existe por una razón concreta: una PWA instalada vive dentro del `scope` de su
manifest, y saltar de un subdominio a otro abría una ventana de navegador con
barra de direcciones. En iOS no hay alternativa. Un solo origen convierte
«cambiar de app» en navegación normal.

---

## El mapa del directorio

**El Kit** — seis herramientas, cada una con su backend Django y su frontend
React. Los puertos son *frontend / API* en desarrollo.

| | Qué hace | Puertos |
|---|---|---|
| `cashflow/` | Tu dinero bajo control y con estrategia | 3000 / 4000 |
| `pulso/` | Entrenamientos y progreso real, sesión a sesión | 3001 / 4001 |
| `yurnal/` | Diario y hábitos, medidos en vez de prometidos | 3002 / 4002 |
| `digest/` | La información que quieres sin el ruido de las redes | 3005 / 4005 |
| `rumbo/` | Objetivos claros partidos en épicas, historias y tareas | 3006 / 4006 |
| `aura/` | Perfil, feed, directos y **el aura**: puntos, niveles y logros | 3007 / 4007 |

**Lo que no es una app del Kit**, y confundirlo es el error más caro:

| | Qué es | Puertos |
|---|---|---|
| `members/` | **La casa.** El área privada que *contiene* al Kit: el Kit es una de sus cuatro puertas | 3009 / 4009 |
| `global_auth/` | **Iris.** La identidad de todo el ecosistema: cuenta, planes, tokens, OAuth | 3003 / 4003 |
| `site/` | **aurx.club.** La web pública, prerenderizada en tiempo de build | 3008 / 4008 |
| `suite/` | El origen único que sirve todo lo anterior bajo un dominio | 3010 |
| `club/` | El Club: [Stoat](https://github.com/stoatchat/self-hosted) —un Discord autohospedado— clonado del repo upstream | 8880 |
| `shared/` | **El sistema de diseño y el auth compartido.** Ni app ni servicio: la única implementación | — |
| `mcp-servers/` | CashFlow desde asistentes de IA, vía MCP sobre OAuth | 4010 |
| `mobile-builds/` | Los envoltorios Capacitor para iOS y Android | — |
| `backups/` | Volcados horarios de cada base de datos | — |

Aura merece una nota: es del Kit, pero no se elige como las demás. Es donde las
otras cinco reportan, y es **gratis para siempre** — si exigiera plan, entrar
aterrizaría en un muro.

---

## Cómo se levanta

```bash
make help              # todos los comandos, por app
make status            # qué está en marcha

make dev-cashflow      # desarrollo con recarga en caliente
make rebuild-cashflow  # reconstruir y redesplegar producción
make logs-cashflow

make rebuild-suite     # OBLIGATORIO tras tocar shared/ o cualquier app que monte
make rebuild-site      # la web pública: build + prerender + despliegue
```

Tres cosas que se olvidan y cuestan una tarde:

- **Un cambio en una app no se ve en producción hasta `make rebuild-suite`.** La
  imagen suelta de su puerto sigue viva, pero es la de desarrollo.
- **`shared/` se commitea y se empuja antes que las apps**: sus imágenes lo
  copian del working tree al construir.
- Lo que se prerenderiza (`site/`) sólo existe después de `make build-site`.
  En `make dev-site` un crawler no vería nada.

---

## Antes de escribir código

Las reglas de `.context/rules/` son **obligatorias** y ganan a cualquier otra
convención:

- **`frontend-design-system.rules.md`** — hay **un** sistema de diseño, en
  `shared/frontend/`, y ocho frontends que lo importan. No hay Tailwind, no hay
  layouts por app, y las clases `ax-*` sólo se definen en `shared/`. Una app
  puede añadir clases de dominio con su prefijo (`cf-`, `sn-`, `yn-`, `dg-`,
  `ga-`, `rb-`, `mb-`) y nada más.
- **`site-design.rules.md`** y **`site-boundary.rules.md`** — la web pública
  consume `shared/` y nunca le escribe; sus piezas de marketing son `st-*`.
- **`commits-and-push.rules.md`** — se commitea **después** de que Manuel
  valide, con mensajes de una sola frase.

Doce repos independientes —`shared`, las seis apps, `members`, `global_auth`,
`site`, `mcp-servers` y `mobile-builds`— bajo la organización **AurxHub** de
GitHub, todos con rama `main`. `club/` es el clon del repo upstream de Stoat, no
código nuestro. La raíz de este workspace —este README, el Makefile,
`CLAUDE.md`, `.context/`— **no está versionada** a propósito.

---

## Qué falta

Nada de esto está a medias por descuido:

- **El Club.** Stoat está en pie en el 8880, pero sin desplegar. Tiene que
  colgar del **propio origen** (`app.aurx.club/club`) y no de un subdominio: su
  cliente guarda la sesión en IndexedDB, e Iris se la inyecta con JS, que en otro
  dominio no puede. No lo metas en `nginx_net` — sus servicios se llaman `api`,
  `redis` y `database`.
- **DNS.** `mcp.aurx.club` todavía no resuelve, así que el MCP de CashFlow sigue
  sin dirección desde la mudanza.
- **El aula.** Cursos, lecciones, vídeo y progreso están en el modelo; el alojamiento
  del vídeo no está decidido y el reproductor no existe.
- **Los pagos del Campus.** Iris ya cobra la suscripción de la suite; cómo
  convive con ella la del Campus es una decisión de negocio abierta.
- **El alta sin contraseña.** «Deja tu correo y estás dentro» aún no es literal:
  falta el enlace mágico en Iris.
- **El correo.** El doble opt-in está escrito y apagado, porque no hay dominio
  de envío.

Y tres huecos que se enseñan en amarillo en la web pública: los testimonios, el
profesorado y los títulos de los cursos. **No hay alumnos inventados, ni precios
inventados, ni mentores inventados en este código, y no puede haberlos**: una
página que finge prueba es peor que una que admite lo que aún espera.
