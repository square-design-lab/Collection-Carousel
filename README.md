# SDL Collection Carousel

A Squarespace plugin that fetches any Squarespace collection (Blog, Portfolio, Events, or Shop) via the Squarespace JSON API and renders it as a fully-featured touch slider powered by a bundled copy of Swiper 11.2.1. No external dependencies — Swiper is bundled directly inside `carousel.js`.

> Plugin identifier: `data-sdl-plugin="collection-carousel"`

## Files

| File | Purpose |
|---|---|
| `carousel.js` | Plugin logic + bundled Swiper 11.2.1 (no classes/constructors — a functional, easy-to-debug implementation) |
| `carousel.css` | Scoped Swiper base styles + plugin styles (all visual styling via `--sdl-cc-*` CSS variables) |
| `config-generator.html` | Visual configuration dashboard with a live preview and copy-paste code generation |
| `collection-carousel-build-guide.md` | Full settings, data-attribute, and CSS-variable reference |

## Installation

1. Upload `carousel.css` and `carousel.js` to Squarespace and add to **Settings → Advanced → Code Injection → Footer**:

   ```html
   <link rel="stylesheet" href="/s/carousel.css">
   <script src="/s/carousel.js"></script>
   ```

2. Add a **Code Block** where you want the carousel:

   ```html
   <div data-sdl-plugin="collection-carousel" data-source="/blog"></div>
   ```

Use [`config-generator.html`](config-generator.html) to configure every option visually and copy the generated markup.

## Configuration

Everything is configured through `data-*` attributes on the host element, with optional global defaults via `window.sdlCollectionCarouselSettings`. See the [build guide](collection-carousel-build-guide.md) for the complete reference: layout & responsive breakpoints, navigation layouts, pagination, autoplay, slide effects, content/metadata display, collection filtering, caching, and the full list of `--sdl-cc-*` styling variables.

## License

Sold via LemonSqueezy. Swiper is included under the MIT License.
