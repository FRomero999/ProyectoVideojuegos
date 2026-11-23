# Plan de la sesión 1 · Grid + `<figure>` + `<picture>`

> Documento de trabajo para aplicar en este proyecto lo indicado en `PLANNING.SESIONES.md` (§ Sesión 1).
> Estado: planificado. La implementación se hará sobre el proyecto actual, no sobre el proyecto de ejemplo de la guía.

## 1. Contexto y decisiones

La sesión 1 persigue maquetar la home y el detalle con CSS Grid y mostrar las carátulas con semántica correcta e imágenes adaptables.

Decisiones acordadas para este proyecto:

| Tema | Decisión |
| --- | --- |
| Maquetación | **CSS Grid** para el layout general, las cards y el detalle. Es una excepción justificada a la regla "solo flex" de `AGENTS.md`; la excepción queda documentada aquí y no se modifica `AGENTS.md`. |
| Imágenes | **Picsum con `<picture>`**: WebP de 800×1000 desde 768 px, WebP de 400×500 por defecto y JPG de reserva. Sin archivos locales en `public/img`. |
| Datos | Se **conserva** el esquema actual de `data/games.json`: `id`, `ranking`, `categoria`, `image`, `alt`, `title`, `description`, `metacritic`. |
| Contenido | Textos de interfaz en inglés, contenido realista, según `AGENTS.md`. |
| Código | Comentarios y nombres de clases/variables en español, CSS anidado y variables. |
| Alcance | Home y detalle. Sin JavaScript nuevo ni cambios en rutas o datos. |

## 2. Estado actual y objetivo

| Archivo | Estado actual | Objetivo |
| --- | --- | --- |
| `views/index.ejs` | Sin `lang`, `charset`, `viewport`, `<title>` ni `<main>`. | Documento HTML5 completo con `<main class="principal">`. |
| `views/partials/ranking.ejs` | `div.ranking-inner` con `a > div > img + h5`; imagen remota. | `ul.ranking-lista` con `li > a > figure > picture + figcaption`. |
| `views/game.ejs` | Solo muestra el título. | Detalle con carátula grande y ficha en Grid `1fr 2fr`. |
| `public/stylesheets/base.css` | Sin layout de página. | `body` con `grid-template-areas`. |
| `public/stylesheets/ranking.css` | Flex con tarjetas de altura fija y `img` absoluta. | Grid de cards con `auto-fill` + `minmax`. |
| `public/stylesheets/footer.css` | Vacío. | Pie con estilo básico. |
| `public/stylesheets/game.css` | No existe. | Estilos del detalle; se importa desde `style.css`. |
| `data/games.json` | Sin cambios. | Sin cambios. |
| `routes/index.js` | Sin cambios. | Sin cambios. |
| `public/javascripts/app.js` | Sin cambios. | Sin cambios. |

Resultado esperado: home con layout de tres zonas (cabecera, principal, pie), cards que se recolocan solas al cambiar el ancho de la ventana y carátulas con `<figure>`, `<figcaption>` y `<picture>`.

## 3. Tareas

### A. Estructura semántica del documento

- Añadir `<!DOCTYPE html>`, `lang="en"`, `<meta charset="utf-8">`, `<meta name="viewport" content="width=device-width, initial-scale=1">` y `<title>`.
- Envolver el contenido entre cabecera y pie en `<main class="principal">`.
- Mantener los comentarios EJS en las plantillas.

`views/index.ejs` objetivo:

```ejs
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Arcade Ledger · Video Game Rankings</title>
  <link rel="stylesheet" href="/stylesheets/style.css">
</head>

<body>

  <!-- Cabecera: marca del ranking y navegación principal -->
  <%- include("partials/header") %>

  <!-- Contenido principal: ranking semanal -->
  <main class="principal">
    <%- include("partials/ranking") %>
  </main>

  <!-- Pie: créditos y nota de actualización -->
  <%- include("partials/footer") %>

  <script src="/javascripts/app.js"></script>
</body>

</html>
```

`views/game.ejs` seguirá la misma estructura, con el markup del detalle dentro de `<main>` (ver tarea E).

### B. Layout general con `grid-template-areas`

En `public/stylesheets/base.css`:

```css
body {
  margin: 0;
  min-height: 100vh;
  display: grid;
  grid-template-areas:
    "cabecera"
    "principal"
    "pie";
  grid-template-rows: auto 1fr auto;

  background-color: var(--color_fondo);
  color: var(--color_titulos);
  font-family: var(--tipo_fuente);
}

.header {
  grid-area: cabecera;
}

.principal {
  grid-area: principal;
}

.pie {
  grid-area: pie;
}
```

Puntos clave:

- `1fr` en la fila central hace que el pie quede abajo aunque haya poco contenido.
- Quitar `min-height: 100vh` de `section.ranking`; ahora lo gestiona el `body`.
- No tocar la maquetación interna de `header.css` (sigue con flex).

### C. Grid de cards y semántica de lista

Reescribir `views/partials/ranking.ejs`:

```ejs
<!-- Ranking semanal: lanzamientos destacados de noviembre de 2026 -->
<section class="ranking">
  <h2>November 2026 Standouts</h2>
  <p>Three releases already out this month, ranked from critic reviews and player scores.</p>

  <!-- Lista de juegos: cada tarjeta es un elemento de lista -->
  <ul class="ranking-lista">
    <% games.forEach(g => { %>
      <li class="ranking-item <%= (g.metacritic > 95) ? 'top' : '' %>">
        <a class="ranking-enlace" href="/game/<%= g.id %>">

          <!-- Carátula autocontenida con pie de foto -->
          <figure class="caratula">
            <picture>
              <!-- Art direction: imagen grande en escritorio -->
              <source media="(min-width: 768px)"
                      srcset="https://picsum.photos/seed/<%= g.id %>/800/1000.webp"
                      type="image/webp">
              <!-- Formato preferente en móvil -->
              <source srcset="https://picsum.photos/seed/<%= g.id %>/400/500.webp"
                      type="image/webp">
              <!-- Reserva obligatoria: la imagen que se pinta y lleva el alt -->
              <img src="https://picsum.photos/seed/<%= g.id %>/400/500"
                   alt="<%= g.alt %>"
                   loading="lazy"
                   width="400"
                   height="500">
            </picture>
            <figcaption>
              <span class="caratula-titulo"><%= g.title %></span>
              <span class="caratula-puntuacion"><%= g.metacritic %></span>
            </figcaption>
          </figure>

        </a>
      </li>
    <% }) %>
  </ul>
</section>
```

Puntos clave:

- El orden de los `<source>` importa: primero el más específico.
- El `<img>` es obligatorio y es el que lleva `alt`, `width` y `height`.
- `loading="lazy"` evita cargar carátulas fuera de pantalla.
- El seed de picsum es el `id` del juego, así cada partida mantiene siempre la misma imagen.
- El badge "top" pasa de `:has()` a la clase `top` del `<li>`, más simple y compatible.

### D. Estilos del ranking con Grid

Reescribir `public/stylesheets/ranking.css` (CSS anidado):

```css
section.ranking {
  max-width: var(--ancho_pagina);
  margin: 0 auto;
  padding: 2rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 2rem;

  ul.ranking-lista {
    margin: 0;
    padding: 0;
    list-style: none;
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(var(--ancho_tarjeta_min), 1fr));
    gap: 1.5rem;

    li.ranking-item {
      position: relative;

      &.top::after {
        content: "top";
        position: absolute;
        top: 0;
        right: 0;
        z-index: 1;
        padding: 0.3rem;
        background-color: var(--color_resaltar);
        color: var(--color_fondo);
        font-weight: bold;
      }

      a.ranking-enlace {
        display: block;
        color: inherit;
        text-decoration: none;
      }
    }
  }

  .caratula {
    margin: 0;

    picture,
    img {
      display: block;
      width: 100%;
    }

    img {
      height: auto;
      aspect-ratio: 4 / 5;
      object-fit: cover;
      border-radius: var(--radio_tarjeta);
      filter: grayscale(1);
      transition: filter 0.3s, transform 0.3s;
    }

    figcaption {
      display: flex;
      justify-content: space-between;
      gap: 1rem;
      padding: 0.5rem 0;
      font-weight: 600;
    }

    &:hover img {
      filter: none;
      transform: scale(1.03);
    }
  }
}
```

- `repeat(auto-fill, minmax(220px, 1fr))` crea tantas columnas como quepan sin media queries.
- Se elimina la altura fija de 400 px y la imagen absoluta; la proporción la mantiene `aspect-ratio`.
- El hover se mantiene en la carátula.

### E. Detalle del juego

`views/game.ejs` objetivo:

```ejs
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title><%= game.title %> · Arcade Ledger</title>
  <link rel="stylesheet" href="/stylesheets/style.css">
</head>

<body>

  <!-- Cabecera común -->
  <%- include("partials/header") %>

  <!-- Detalle: carátula a la izquierda y ficha a la derecha -->
  <main class="principal">
    <article class="detalle">

      <figure class="caratula">
        <picture>
          <source media="(min-width: 768px)"
                  srcset="https://picsum.photos/seed/<%= game.id %>/800/1000.webp"
                  type="image/webp">
          <source srcset="https://picsum.photos/seed/<%= game.id %>/400/500.webp"
                  type="image/webp">
          <img src="https://picsum.photos/seed/<%= game.id %>/400/500"
               alt="<%= game.alt %>"
               width="400"
               height="500">
        </picture>
        <figcaption><%= game.title %> · <%= game.metacritic %></figcaption>
      </figure>

      <div class="ficha">
        <h1><%= game.title %></h1>
        <p class="ficha-meta"><%= game.categoria %> · Metacritic <%= game.metacritic %></p>
        <p><%- game.description %></p>
        <a class="volver" href="/">← Back to ranking</a>
      </div>

    </article>
  </main>

  <!-- Pie común -->
  <%- include("partials/footer") %>

  <script src="/javascripts/app.js"></script>
</body>

</html>
```

- Se usa `<%- game.description %>` porque las descripciones actuales contienen etiquetas como `<em>` y el contenido es local y de confianza.
- La maquetación `1fr 2fr` es la práctica que pide la guía; en la sesión 2 se convertirá a mobile-first.

### F. Estilos auxiliares

`public/stylesheets/game.css` (nuevo):

```css
article.detalle {
  max-width: var(--ancho_pagina);
  margin: 0 auto;
  padding: 2rem 1.5rem;
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 2rem;
  align-items: start;

  .caratula {
    margin: 0;

    img {
      width: 100%;
      height: auto;
      display: block;
      border-radius: var(--radio_tarjeta);
    }

    figcaption {
      padding: 0.5rem 0;
      font-weight: 600;
    }
  }

  .ficha {
    h1 {
      margin-top: 0;
    }

    .ficha-meta {
      color: var(--color_resaltar);
      font-weight: 600;
    }
  }

  .volver {
    color: var(--color_primer_plano);
  }
}
```

`public/stylesheets/footer.css`:

```css
footer.pie {
  padding: 1.5rem;
  text-align: center;
  border-top: 1px solid var(--color_primer_plano);
}
```

`public/stylesheets/vars.css`, añadir:

```css
--ancho_tarjeta_min: 220px;
--radio_tarjeta: 8px;
```

`public/stylesheets/style.css`, añadir la importación:

```css
@import url(game.css);
```

### G. Verificación

1. `npm start` y abrir `http://localhost:3000/`.
2. Redimensionar la ventana: las cards deben cambiar de número de columnas sin media queries.
3. Inspeccionar una card en DevTools: debe existir `figure > picture > source + img` y elegirse la imagen de 800 px a partir de 768 px.
4. Comprobar que cada `img` tiene `alt`, `loading="lazy"`, `width` y `height`.
5. Abrir `http://localhost:3000/game/elden-ring`: carátula a la izquierda y ficha a la derecha.
6. Comprobar que el pie queda abajo en una página con poco contenido.
7. `http://localhost:3000/game/no-existe` debe seguir mostrando la vista de error.

## 4. Archivos afectados

| Archivo | Acción |
| --- | --- |
| `views/index.ejs` | Modificar: documento HTML5, `main`. |
| `views/game.ejs` | Modificar: documento HTML5, detalle con `figure` y ficha. |
| `views/partials/ranking.ejs` | Reescribir: lista, `figure`, `picture`, `figcaption`. |
| `public/stylesheets/base.css` | Modificar: Grid del layout general. |
| `public/stylesheets/ranking.css` | Reescribir: Grid de cards y carátulas. |
| `public/stylesheets/game.css` | Crear: detalle `1fr 2fr`. |
| `public/stylesheets/footer.css` | Modificar: estilo del pie. |
| `public/stylesheets/vars.css` | Modificar: nuevas variables. |
| `public/stylesheets/style.css` | Modificar: importar `game.css`. |

Sin cambios en `routes/index.js`, `data/*`, `app.js` ni `public/javascripts/app.js`.

## 5. Criterios de aceptación

- La home usa `grid-template-areas` para cabecera, principal y pie.
- Las cards se distribuyen con `auto-fill` + `minmax` y no hay scroll horizontal.
- Cada card usa `figure`, `figcaption`, `picture`, `source` y un `img` con `alt`, `loading`, `width` y `height`.
- El detalle usa `grid-template-columns: 1fr 2fr`.
- No se han añadido eventos inline ni JavaScript nuevo.
- Los comentarios de plantillas están en español y el contenido visible en inglés.
- `npm start` arranca sin errores y las rutas existentes siguen funcionando.

## 6. Fuera de alcance

- Media queries de layout y enfoque mobile-first (sesión 2).
- Formularios y procesado en Express (sesión 2).
- Validación en servidor, Post/Redirect/Get y tablas (sesión 3).
- `<dialog>` (sesión 4) y sesiones (sesión 5).
- Cambios en `data/games.json` o en el modelo de datos.

## 7. Nota sobre `AGENTS.md`

`AGENTS.md` pide maquetar únicamente con flex. La sesión 1 exige Grid, por lo que se aplica Grid como excepción didáctica aprobada. Fuera de esta excepción se mantienen el resto de convenciones: vanilla JavaScript, `classList`, ausencia de atributos de eventos, comentarios en plantillas, CSS anidado y variables.
