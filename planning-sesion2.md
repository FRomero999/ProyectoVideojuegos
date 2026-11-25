# Plan de la sesión 2 · Media queries + formularios y procesado

> Documento de trabajo para aplicar en este proyecto lo indicado en `PLANNING.SESIONES.md` (§ Sesión 2).
> Estado: planificado. **Requiere que la sesión 1 esté aplicada** (`planning-sesion1.md`): el layout ya usa Grid, las cards son `figure > picture` y el detalle usa `1fr 2fr`.

## 1. Contexto y decisiones

La sesión 2 persigue dos objetivos: hacer la web responsive de forma controlada (mobile first) y crear el primer formulario que envía datos a Express para guardarlos en el JSON.

Decisiones acordadas para este proyecto:

| Tema | Decisión |
| --- | --- |
| Responsive | Media queries con `min-width` (mobile first). Se mantiene `auto-fill` en el ranking y se usan media queries para detalle, formulario y cabecera. |
| Maquetación | Se conserva Grid como excepción aprobada a la regla "solo flex" de `AGENTS.md`. |
| Datos | Se conserva el esquema de `data/games.json`: `id`, `ranking`, `categoria`, `image`, `alt`, `title`, `description`, `metacritic`. |
| Formulario | Los `name` de los campos coinciden con las claves del JSON (`id`, `title`, `categoria`, `metacritic`, `image`, `description`). El slug se escribe en el formulario. |
| Imágenes | Si no se indica URL, se genera una de picsum con el slug introducido. |
| Contenido | Textos de interfaz en inglés; comentarios y código en español. |

## 2. Estado de partida tras la sesión 1

| Archivo | Estado |
| --- | --- |
| `app.js` | Ya tiene `express.urlencoded({ extended: false })`; **no hay que añadirlo**. |
| `routes/index.js` | Solo rutas GET (`/` y `/game/:id`). |
| `data/data.js` | Solo lectura (`getGames`, `getGameById`). |
| `data/games.json` | Siete juegos; sin escritura desde el servidor. |
| `views/` | `index.ejs`, `game.ejs`, `error.ejs` y parciales. No hay formulario. |
| `public/stylesheets/` | Layout Grid y detalle `1fr 2fr`; sin estilos de formulario. |
| Navegación | El menú no tiene enlace para añadir un juego. |

## 3. Tareas

### A. Media queries mobile first

Regla general: el CSS base describe el móvil y los `@media (min-width: ...)` **añaden** mejoras.

**Detalle del juego** (`public/stylesheets/game.css`): pasar de `1fr 2fr` fijo a una columna en móvil y dos en escritorio.

```css
article.detalle {
  max-width: var(--ancho_pagina);
  margin: 0 auto;
  padding: 2rem 1.5rem;
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
  align-items: start;
}

@media (min-width: 768px) {
  article.detalle {
    grid-template-columns: 1fr 2fr;
  }
}
```

**Cabecera** (`public/stylesheets/header.css`): en móvil, altura automática y menú que puede envolver; en escritorio, el aspecto actual.

```css
header.header {
  height: auto;
  padding: 1.5rem 1rem;

  nav.menu ul {
    flex-wrap: wrap;
  }
}

@media (min-width: 768px) {
  header.header {
    height: 12rem;
    padding: 0;

    nav.menu ul {
      flex-wrap: nowrap;
    }
  }
}
```

**Ranking** (`public/stylesheets/ranking.css`): se mantiene `repeat(auto-fill, minmax(...))`, que ya es responsive. Sirve para explicar en clase la diferencia entre responsive automático (`auto-fill`) y control manual (media queries). No se sustituye por breakpoints fijos para no perder la mejora de la sesión 1.

**Formulario** (`public/stylesheets/formulario.css`): una columna en móvil, dos en escritorio (ver tarea E).

**Pie** (`public/stylesheets/footer.css`): sin cambios de estructura; opcionalmente ajustar el padding en móvil.

### B. Anatomía del formulario

Crear `views/nuevo.ejs`:

```ejs
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Add a game · Arcade Ledger</title>
  <link rel="stylesheet" href="/stylesheets/style.css">
</head>

<body>

  <!-- Cabecera común -->
  <%- include("partials/header") %>

  <!-- Alta de un juego: los name coinciden con las claves del JSON -->
  <main class="principal">
    <section class="alta">
      <h1>Add a game</h1>
      <p>Register a new release in the Arcade Ledger ranking.</p>

      <form action="/juegos" method="POST" class="formulario">

        <label for="id">Slug (URL id)</label>
        <input type="text" id="id" name="id"
               required pattern="[a-z0-9]+(-[a-z0-9]+)*"
               title="Only lowercase letters, numbers and hyphens"
               placeholder="hollow-knight">

        <label for="title">Title</label>
        <input type="text" id="title" name="title"
               required minlength="2" maxlength="80"
               placeholder="Hollow Knight">

        <label for="categoria">Category</label>
        <select id="categoria" name="categoria" required>
          <option value="Action-Adventure">Action-Adventure</option>
          <option value="RPG">RPG</option>
          <option value="MMORPG">MMORPG</option>
          <option value="Fighting">Fighting</option>
          <option value="Tactical Shooter">Tactical Shooter</option>
          <option value="Simulation">Simulation</option>
        </select>

        <label for="metacritic">Metacritic score (0-100)</label>
        <input type="number" id="metacritic" name="metacritic"
               min="0" max="100" step="1" required>

        <label for="image">Cover image URL (optional)</label>
        <input type="url" id="image" name="image"
               placeholder="https://picsum.photos/seed/my-game/400/500">

        <label for="description">Description (optional)</label>
        <textarea id="description" name="description" rows="4"></textarea>

        <button type="submit">Save game</button>
      </form>

      <a class="volver" href="/">← Back to ranking</a>
    </section>
  </main>

  <!-- Pie común -->
  <%- include("partials/footer") %>

  <script src="/javascripts/app.js"></script>
</body>

</html>
```

Puntos clave:

- Sin `name`, el campo no llega al servidor. El atributo `id` solo sirve para `label`, CSS y JavaScript.
- El campo con `name="id"` es el slug que se usará en la URL (`/game/hollow-knight`).
- `label for="x"` debe coincidir con `id="x"`.
- `required`, `pattern`, `min`, `max` y `minlength` son comodidad del navegador, no seguridad; se retoma en la sesión 3.
- Se usa `res.render('nuevo')`, no `sendFile`, porque el proyecto trabaja con EJS.

Añadir el enlace en `views/partials/header.ejs`:

```ejs
<li><a href="/juegos/nuevo">Add game</a></li>
```

### C. GET frente a POST

Documentar en el plan de aula y comprobar en la pestaña *Network*:

- **GET**: los datos viajan en la URL; se usa para consultar.
- **POST**: los datos viajan en el cuerpo; se usa para crear o modificar.

El formulario de alta usa `method="POST"` y `action="/juegos"`.

### D. Express recibe y guarda los datos

**1. Ampliar `data/data.js`** con escritura y alta:

```js
const RUTA = path.join(dataDir, 'games.json');

/**
 * Guarda el array completo de juegos en games.json
 * @param {Array} games Lista de juegos
 */
function guardarGames(games) {
  fs.writeFileSync(RUTA, JSON.stringify(games, null, 2) + '\n');
}

/**
 * Añade un juego nuevo al JSON y devuelve el juego creado
 * @param {Object} datos Datos del formulario
 * @returns {Object} Juego guardado
 */
function agregarJuego(datos) {
  const games = getGames();
  const ranking = games.length
    ? Math.max(...games.map(juego => juego.ranking)) + 1
    : 1;

  const juego = {
    id: datos.id,
    ranking,
    categoria: datos.categoria,
    image: datos.image || `https://picsum.photos/seed/${datos.id}/400/500`,
    alt: `Cover art for ${datos.title}`,
    title: datos.title,
    description: datos.description || '',
    metacritic: Number(datos.metacritic)
  };

  games.push(juego);
  guardarGames(games);
  return juego;
}

module.exports = { getGames, getGameById, agregarJuego };
```

**2. Añadir rutas en `routes/index.js`**:

```js
/* GET formulario de alta. */
router.get('/juegos/nuevo', function (req, res, next) {
  res.render('nuevo');
});

/* POST procesar el formulario. */
router.post('/juegos', function (req, res, next) {
  console.log(req.body); // ayuda visual en clase
  data.agregarJuego(req.body);
  res.redirect('/');
});
```

Puntos clave:

- `express.urlencoded` ya está configurado en `app.js`; sin él, `req.body` sería `undefined`.
- El `id` no se genera en el servidor: llega en `req.body.id` desde el campo slug del formulario.
- Todo llega como texto, también `metacritic`; de ahí el `Number(...)` en `data.js`.
- El `POST` termina en `res.redirect('/')` cuando todo va bien: así se evita reenviar el formulario al pulsar F5 (base del Post/Redirect/Get de la sesión 3).
- La ruta `/juegos/nuevo` debe declararse antes que cualquier futura `/juegos/:id` para que Express no interprete "nuevo" como un id.

### E. Estilos del formulario

Crear `public/stylesheets/formulario.css`:

```css
section.alta {
  max-width: var(--ancho_pagina);
  margin: 0 auto;
  padding: 2rem 1.5rem;

  .formulario {
    display: grid;
    gap: 0.75rem;
    max-width: 600px;

    label {
      font-weight: 600;
    }

    input,
    select,
    textarea {
      width: 100%;
      box-sizing: border-box;
      padding: 0.6rem;
      border: 1px solid var(--color_primer_plano);
      border-radius: var(--radio_tarjeta);
      font-family: inherit;
      font-size: 1rem;
    }

    textarea {
      resize: vertical;
    }

    button {
      justify-self: start;
      padding: 0.7rem 1.4rem;
      border: none;
      border-radius: var(--radio_tarjeta);
      background-color: var(--color_resaltar);
      color: var(--color_fondo);
      font-weight: 700;
      cursor: pointer;
    }

    button:hover {
      filter: brightness(0.9);
    }
  }

  .volver {
    display: inline-block;
    margin-top: 1rem;
    color: var(--color_primer_plano);
  }
}

@media (min-width: 768px) {
  section.alta .formulario {
    grid-template-columns: 180px 1fr;
    align-items: center;

    button {
      grid-column: 2;
    }
  }
}
```

Añadir la importación en `public/stylesheets/style.css`:

```css
@import url(formulario.css);
```

### F. Verificación

1. `npm start` y abrir `http://localhost:3000/juegos/nuevo`.
2. Comprobar en móvil (DevTools) que el formulario ocupa una columna y en escritorio dos.
3. Enviar un juego. Debe redirigir a `/` y aparecer la nueva card.
4. Abrir `data/games.json` y comprobar que el juego se ha guardado con el slug introducido en `id`, además de `ranking`, `alt`, `image` y `metacritic` numérico.
5. Si se dejó vacía la URL de imagen, la card debe usar `https://picsum.photos/seed/<id>/400/500`.
6. Ver en la pestaña *Network* la petición `POST /juegos` y el `302` de redirección.
7. Probar a pulsar F5 tras guardar: al ser un `GET /`, no se duplica el juego.
8. Desactivar `required` desde DevTools y enviar campos vacíos: el servidor guardará datos inválidos. Es el problema que se resuelve en la sesión 3.
9. Abrir `/game/<id-nuevo>` y comprobar que el detalle funciona con los datos guardados.
10. Repetir el alta con el mismo slug: se guardarán dos juegos con el mismo `id`. La unicidad se valida en la sesión 3.

## 4. Archivos afectados

| Archivo | Acción |
| --- | --- |
| `data/data.js` | Modificar: `guardarGames` y `agregarJuego`. |
| `routes/index.js` | Modificar: `GET /juegos/nuevo` y `POST /juegos`. |
| `views/nuevo.ejs` | Crear: formulario de alta. |
| `views/partials/header.ejs` | Modificar: enlace "Add game". |
| `public/stylesheets/game.css` | Modificar: mobile first (`1fr` → `1fr 2fr` desde 768 px). |
| `public/stylesheets/header.css` | Modificar: cabecera responsive. |
| `public/stylesheets/formulario.css` | Crear: estilos del formulario. |
| `public/stylesheets/style.css` | Modificar: importar `formulario.css`. |

Sin cambios en `app.js` (`urlencoded` ya existe), `data/games.json` (se modifica en tiempo de ejecución al guardar), `bin/www` ni `public/javascripts/app.js`.

## 5. Criterios de aceptación

- La web se ve correctamente en móvil y escritorio usando solo `min-width`.
- El detalle pasa de una a dos columnas a partir de 768 px.
- El formulario tiene `action="/juegos"`, `method="POST"` y todos los campos con `name` e `id` correctos.
- Los datos enviados se guardan en `data/games.json` con el esquema actual.
- El alta redirige a `/` y la nueva card aparece en el ranking.
- La imagen se toma del campo `image` o, si está vacía, de picsum con el slug introducido.
- El `id` se toma del campo slug del formulario y el `ranking` se asigna al final.
- No se han añadido eventos inline ni JavaScript de cliente nuevo.
- `npm start` arranca sin errores y las rutas de la sesión 1 siguen funcionando.

## 6. Fuera de alcance

- Validación en servidor y mensajes de error (sesión 3).
- Post/Redirect/Get con repintado de errores (sesión 3).
- Tabla de administración, edición y borrado (sesión 3).
- `<dialog>` (sesión 4) y sesiones (sesión 5).
- Persistencia con base de datos o escritura concurrente segura; para el aula basta con `fs.writeFileSync`.

## 7. Riesgos y notas

- `Number('')` devuelve `0`; no es una validación real. Se corregirá en la sesión 3.
- La escritura síncrona del JSON es suficiente para el aula, pero no para producción.
- Si se repite un slug, se guardarán dos juegos con el mismo `id`; la unicidad se valida en la sesión 3.
- El contenido de `description` puede contener HTML local; en el detalle se pinta con `<%- ... %>` porque es contenido propio.
- Se mantiene la excepción de Grid aprobada en la sesión 1; el resto de convenciones de `AGENTS.md` se respetan.
