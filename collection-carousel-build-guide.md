# Collection Carousel — Squarespace Plugin Build Guide

> **Plugin identifier:** `data-sdl-plugin="collection-carousel"`
> **Swiper version bundled:** 11.2.1
> **Supported collections:** Blog, Portfolio, Events, Shop (Products)

---

## Table of Contents

1. [Overview](#overview)
2. [How the Plugin Works](#how-the-plugin-works)
3. [File Structure](#file-structure)
4. [Installation & Initialization](#installation--initialization)
5. [HTML Markup](#html-markup)
6. [Complete Settings Reference](#complete-settings-reference)
   - [Core Settings](#core-settings)
   - [Layout & Display](#layout--display)
   - [Slides & Responsive Breakpoints](#slides--responsive-breakpoints)
   - [Navigation](#navigation)
   - [Pagination](#pagination)
   - [Autoplay](#autoplay)
   - [Scroll & Interaction](#scroll--interaction)
   - [Slide Effects](#slide-effects)
   - [Content Display](#content-display)
   - [Metadata](#metadata)
   - [Collection Filtering](#collection-filtering)
   - [Performance & Caching](#performance--caching)
   - [Integration](#integration)
7. [Data Attributes Reference](#data-attributes-reference)
8. [CSS Custom Properties (Variables)](#css-custom-properties-variables)
9. [Navigation Layouts](#navigation-layouts)
10. [Slide Effects Reference](#slide-effects-reference)
11. [Collection Types & Data Mapping](#collection-types--data-mapping)
12. [Content Order System](#content-order-system)
13. [Metadata System](#metadata-system)
14. [Loop Strategies](#loop-strategies)
15. [Full-Width Mode](#full-width-mode)
16. [Caching System](#caching-system)
17. [Global Configuration](#global-configuration)
18. [JavaScript Events API](#javascript-events-api)
19. [CSS Architecture](#css-architecture)
20. [Squarespace Editor Notifications](#squarespace-editor-notifications)
21. [Complete Implementation Examples](#complete-implementation-examples)

---

## Overview

The Collection Carousel is a Squarespace plugin that fetches data from any Squarespace collection (Blog, Portfolio, Events, or Shop) via the Squarespace JSON API and renders it as a fully-featured touch slider powered by Swiper.js. All Swiper logic is self-contained — Swiper is bundled directly inside `plugin.js` as a namespaced module (`sdlCollectionCarousel.Swiper`), so no external CDN is needed.

The plugin is configured entirely through `data-*` attributes on the host element, with optional global defaults via `window.sdlCollectionCarouselSettings`.

---

## How the Plugin Works

1. On page load the IIFE at the bottom of `carousel.js` scans the DOM for `[data-sdl-plugin="collection-carousel"]:not([data-loading])`.
2. For each matching element a `new sdlCollectionCarousel(el)` instance is created and the element receives `data-loading="loading"`.
3. The constructor deep-merges three layers of settings: **defaults → global user settings → per-element data attributes**.
4. `init()` is called asynchronously:
   - `getCollectionData()` fetches items from the Squarespace JSON API at the URL specified in `data-source`.
   - For events, both upcoming and past items are fetched and merged.
   - For products, category metadata is enriched from `nestedCategories`.
   - Results are optionally cached in `localStorage`.
5. `buildStructure()` constructs the full DOM: swiper container, slides, navigation buttons, and pagination wrapper.
6. A Swiper instance is created with the resolved config from `getSwiperConfig()`.
7. The custom events `sdlCollectionCarousel:beforeInit` and `sdlCollectionCarousel:ready` are dispatched on the element.
8. The instance exposes `el.sdlCollectionCarousel = { settings, swiper, items }` for external access.

---

## File Structure

```
plugin.js    — Main plugin class + bundled Swiper 11.2.1
plugin.css   — Swiper base styles (scoped) + plugin custom styles
```

Both files must be loaded on every page that uses the carousel. Add them via Squarespace **Settings → Advanced → Code Injection** or inject them into a specific page.

---

## Installation & Initialization

### 1. Inject the files

In Squarespace **Settings → Advanced → Code Injection → Footer**:

```html
<link rel="stylesheet" href="/s/carousel.css">
<script src="/s/carousel.js"></script>
```

### 2. Upload files to Squarespace File Storage

Upload `carousel.css` and `carousel.js` to **Assets** or use the file manager. Reference them using the Squarespace asset URL.

### 3. Add the HTML block

Place a **Code Block** on any page with the required markup (see [HTML Markup](#html-markup)).

### Re-initialization

The plugin attaches `window.sdlCollectionCarousel.init()` for manual re-initialization (useful after dynamic page loads or AJAX navigation):

```javascript
window.sdlCollectionCarousel.init();
```

The init function will only process elements that do **not** already have `data-loading` set.

---

## HTML Markup

### Minimal required markup

```html
<div
  data-sdl-plugin="collection-carousel"
  data-source="/blog"
>
</div>
```

### Full example with all common options

```html
<div
  id="my-carousel"
  data-sdl-plugin="collection-carousel"
  data-source="/portfolio"
  data-limit="12"
  data-slides-per-view-sm="1"
  data-slides-per-vies-dld="2"
  data-slides-per-view-lg="3"
  data-space-between-sm="17"
  data-space-between-md="24"
  data-space-between-lg="34"
  data-loop="true"
  data-autoplay="5000"
  data-navigation="true"
  data-navigation-layout="overlay"
  data-pagination="true"
  data-pagination-type="bullets"
  data-aspect-ratio="4/3"
  data-title="true"
  data-excerpt="true"
  data-thumbnail="true"
  data-clickthrough="true"
>
</div>
```

---

## Complete Settings Reference

### Core Settings

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `id` | — | `string` | auto-generated | Element ID, auto-assigned if absent |
| `source` | `data-source` | `string` | — | **Required.** Squarespace collection URL (e.g. `/blog`, `/shop`, `/portfolio`, `/events`) |
| `limit` | `data-limit` | `number` | `20` | Max items to display. Events default to `40` if not explicitly set |
| `speed` | `data-speed` | `number` | `300` | Transition speed in milliseconds |

### Layout & Display

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `layout` | `data-layout` | `string` | `"full-width"` | Layout mode: `"full-width"`, `"header-adapt"`, or `"folder"` |
| `fullWidth` | `data-full-width` | `boolean` | auto-detected | Forces full-width mode. Auto-detected based on element width vs. `window.innerWidth` |
| `disableFullWidthOffset` | `data-disable-full-width-offset` | `boolean` | `false` | Disable the automatic slide offset calculation in full-width mode |
| `aspectRatio` | `data-aspect-ratio` | `string` | `"auto"` | Thumbnail aspect ratio (CSS value, e.g. `"4/3"`, `"16/9"`, `"1/1"`) |
| `thumbnail` | `data-thumbnail` | `boolean` | `true` | Show/hide slide thumbnail image |
| `clickthrough` | `data-clickthrough` | `boolean` | `true` | Wrap images and titles in links to item URLs |
| `newWindow` | `data-new-window` | `boolean` | `false` | Open item links in a new tab |

### Slides & Responsive Breakpoints

The plugin uses three breakpoints: **sm** (0–767px), **md** (768–1023px), **lg** (1024px+).

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `slidesPerView` | `data-slides-per-view` | `number \| string` | `"auto"` | Global fallback slides per view |
| `slidesPerViewSm` | `data-slides-per-view-sm` | `number \| string` | `1` | Slides visible on mobile |
| `slidesPerViesdld` | `data-slides-per-vies-dld` | `number \| string` | `2` | Slides visible on tablet *(note the intentional key name)*|
| `slidesPerViewLg` | `data-slides-per-view-lg` | `number \| string` | `4` | Slides visible on desktop |
| `slidesPerGroup` | `data-slides-per-group` | `number` | `1` | Slides to advance per navigation click |
| `slidesPerGroupSm` | `data-slides-per-group-sm` | `number` | `1` | Per-group override for mobile |
| `slidesPerGroupMd` | `data-slides-per-group-md` | `number` | `1` | Per-group override for tablet |
| `slidesPerGroupLg` | `data-slides-per-group-lg` | `number` | `1` | Per-group override for desktop |
| `groupSlides` | `data-group-slides` | `boolean` | `false` | Auto-set group size to match the current `slidesPerView` |
| `spaceBetween` | `data-space-between` | `number` | `30` | Global gap between slides (px). Sets all breakpoints when applied alone |
| `spaceBetweenSm` | `data-space-between-sm` | `number` | `17` | Gap on mobile |
| `spaceBetweenMd` | `data-space-between-md` | `number` | `34` | Gap on tablet |
| `spaceBetweenLg` | `data-space-between-lg` | `number` | `54` | Gap on desktop |
| `centeredSlides` | `data-centered-slides` | `boolean` | `false` | Center the active slide. Auto-enabled when `loop=true` or `effect="coverflow"` |
| `loop` | `data-loop` | `boolean` | `false` | Infinite loop mode. Auto-enables `centeredSlides` |
| `loopFillItems` | `data-loop-fill-items` | `boolean` | `false` | Duplicate items if the total count is too low to fill the view in loop mode |

#### Slide width as CSS value

`slidesPerView` (and breakpoint variants) accepts CSS length strings:

```html
data-slides-per-view-lg="300px"
data-slides-per-view-lg="80%"
data-slides-per-view-lg="25vw"
```

When a CSS value is used, `slidesPerView` is set to `"auto"` and the width is applied as a `maxWidth` on each slide.

### Navigation

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `navigation` | `data-navigation` | `boolean` | `true` | Show prev/next arrow buttons |
| `navigationLayout` | `data-navigation-layout` | `string` | `"overlay"` | Arrow position. See [Navigation Layouts](#navigation-layouts) |
| `navigationArrowPrev` | — | `string` (HTML) | built-in SVG | Custom SVG HTML for the prev arrow |
| `navigationArrowNext` | — | `string` (HTML) | built-in SVG | Custom SVG HTML for the next arrow |

**Setting `data-navigation-layout="false"` also disables navigation** (equivalent to `data-navigation="false"`).

### Pagination

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `pagination` | `data-pagination` | `boolean` | `true` | Show pagination indicator |
| `paginationType` | `data-pagination-type` | `string` | `"bullets"` | `"bullets"`, `"fraction"`, or `"progressbar"` |
| `dynamicBullets` | `data-dynamic-bullets` | `boolean \| number` | `false` | Enable dynamic bullet sizing. When set to a number, becomes `dynamicMainBullets` |
| `dynamicMainBullets` | `data-dynamic-main-bullets` | `number` | `8` | Number of main bullets visible in dynamic mode |

### Autoplay

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `autoplay` | `data-autoplay` | `number \| false` | `false` | Autoplay delay in ms. Values 1–9 are auto-multiplied by 1000. Minimum enforced: 500ms |
| `autoplayDisableOnInteraction` | `data-autoplay-disable-on-interaction` | `boolean` | `false` | Stop autoplay after user interaction |

### Scroll & Interaction

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `freeMode` | `data-free-mode` | `boolean` | `false` | Enable free-scroll (momentum) mode |
| `mousewheel` | `data-mousewheel` | `boolean` | mirrors `freeMode` | Enable mousewheel scrolling |
| `keyboard` | — | `boolean` | `true` (always on) | Keyboard navigation (only when carousel is in viewport) |

> When `freeMode` is enabled, the prev button gets a custom click handler implementing snap-grid-based sliding (Swiper's default slidePrev does not work correctly in freeMode).

### Slide Effects

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `effect` | `data-effect` | `string` | `null` (slide) | Transition effect. See [Slide Effects Reference](#slide-effects-reference) |
| `coverflowRotate` | `data-coverflow-rotate` | `number` | `0` | Coverflow rotation angle |
| `coverflowScale` | `data-coverflow-scale` | `number` | `1` | Coverflow scale factor |
| `coverflowSlideShadows` | `data-coverflow-slide-shadows` | `boolean` | `false` | Show shadows in coverflow mode |
| `creative` | `data-creative` | `JSON` | see below | Full creative effect config as a JSON string |

### Content Display

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `title` | `data-title` | `boolean` | `true` | Show item title |
| `excerpt` | `data-excerpt` | `boolean` | `true` | Show item excerpt |
| `price` | `data-price` | `boolean` | `true` | Show product price (Shop only) |
| `eventDates` | `data-event-dates` | `boolean` | `true` | Show event start/end dates (Events only) |
| `categories` | `data-categories` | `boolean` | `true` | Show categories |
| `tags` | `data-tags` | `boolean` | `true` | Show tags |
| `contentOrder` | `data-content-order` | `JSON array` | see below | Order of content blocks within each slide |

**Default content order:**

```javascript
["metadataAboveTitle", "title", "metadataBelowTitle", "price", "eventDates", "excerpt", "metadataBelowExcerpt"]
```

### Metadata

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `metadataAboveTitle` | `data-metadata-above-title` | `string` (CSV) | `[]` | Comma-separated metadata fields to show above the title |
| `metadataBelowTitle` | `data-metadata-below-title` | `string` (CSV) | `[]` | Comma-separated metadata fields below the title |
| `metadataBelowExcerpt` | `data-metadata-below-excerpt` | `string` (CSV) | `[]` | Comma-separated metadata fields below the excerpt |
| `metadataDelimiter` | `data-metadata-delimiter` | `string` | `"\|"` | Separator rendered between adjacent metadata elements |
| `metadataCategoryLinks` | `data-metadata-category-links` | `boolean` | `false` | Wrap category names in links to their filtered collection URL |
| `metadataTagLinks` | `data-metadata-tag-links` | `boolean` | `false` | Wrap tag names in links to their filtered collection URL |
| `categoriesDelimiter` | `data-categories-delimiter` | `string` | `", "` | Delimiter between categories in the categories element |
| `tagsDelimiter` | `data-tags-delimiter` | `string` | `", "` | Delimiter between tags in the tags element |

**Available metadata field names:** `publisheddate` / `publishdate` / `date`, `author`, `categories`, `tags`

### Collection Filtering

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `featured` | `data-featured` | `boolean` | `false` | Only show items marked as featured/starred in Squarespace |
| `events` | `data-events` | `string` | `"all"` | Filter events: `"all"`, `"upcoming"`, or `"past"` |

### Performance & Caching

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `cacheDuration` | `data-cache-duration` | `number` | `0` | Minutes to cache collection data in `localStorage`. `0` = disabled |
| `cacheCollections` | `data-cache-collections` | `boolean` | `false` | Legacy cache flag (use `cacheDuration` instead) |

### Integration

| Setting | Data Attribute | Type | Default | Description |
|---|---|---|---|---|
| `sdlPopups` | `data-sdl-popups` | `boolean` | `false` | Use SDL Popups integration — rewrites item links to `#sdl-popup=<url>` format |

---

## Data Attributes Reference

Quick lookup for all supported `data-*` attributes:

```html
<!-- Required -->
data-source="/collection-url"

<!-- Core -->
data-limit="20"
data-speed="300"
data-full-width="true|false"
data-disable-full-width-offset="true|false"

<!-- Slides & Layout -->
data-slides-per-view-sm="1"
data-slides-per-vies-dld="2"          <!-- tablet breakpoint -->
data-slides-per-view-lg="4"
data-slides-per-group="1"
data-slides-per-group-sm="1"
data-slides-per-group-md="1"
data-slides-per-group-lg="1"
data-group-slides="true|false"
data-space-between="30"
data-space-between-sm="17"
data-space-between-md="34"
data-space-between-lg="54"
data-loop="true|false"
data-loop-fill-items="true|false"
data-group-slides-loop-strategy="keep-blanks|disable-loop-when-uneven"
data-centered-slides="true|false"
data-aspect-ratio="4/3"

<!-- Navigation -->
data-navigation="true|false"
data-navigation-layout="overlay|overlay image|top left|top right|top center|top spread|bottom left|bottom right|bottom center|bottom spread"

<!-- Pagination -->
data-pagination="true|false"
data-pagination-type="bullets|fraction|progressbar"
data-dynamic-bullets="true|8"

<!-- Autoplay -->
data-autoplay="3000"
data-autoplay-disable-on-interaction="true|false"

<!-- Interaction -->
data-free-mode="true|false"
data-mousewheel="true|false"

<!-- Effects -->
data-effect="fade|flip|cube|coverflow|cards|creative|creative-coverflow|creative-coverflow-2"
data-coverflow-rotate="50"
data-coverflow-scale="0.85"
data-coverflow-slide-shadows="true|false"
data-creative='{"limitProgress":3,"prev":{"opacity":0.75,"translate":["-110%",0,0],"scale":0.9},"next":{"opacity":0.75,"translate":["110%",0,0],"scale":0.9}}'

<!-- Content -->
data-thumbnail="true|false"
data-title="true|false"
data-excerpt="true|false"
data-price="true|false"
data-event-dates="true|false"
data-categories="true|false"
data-tags="true|false"
data-clickthrough="true|false"
data-new-window="true|false"
data-content-order='["metadataAboveTitle","title","metadataBelowTitle","excerpt"]'

<!-- Metadata -->
data-metadata-above-title="publisheddate, author"
data-metadata-below-title="categories"
data-metadata-below-excerpt="tags"
data-metadata-delimiter="|"
data-categories-delimiter=", "
data-tags-delimiter=", "
data-metadata-category-links="true|false"
data-metadata-tag-links="true|false"

<!-- Filtering -->
data-featured="true|false"
data-events="all|upcoming|past"

<!-- Caching -->
data-cache-duration="60"

<!-- Integration -->
data-sdl-popups="true|false"
```

---

## CSS Custom Properties (Variables)

All visual styling is controlled through CSS custom properties. Override them in Squarespace's **Custom CSS** editor or in a `<style>` block targeting the element's ID.

### Thumbnail

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-aspect-ratio` | `4/3` | Thumbnail aspect ratio |
| `--sdl-cc-border-radius` | `0` | Thumbnail corner radius |
| `--sdl-cc-border-width` | `0` | Thumbnail border width |
| `--sdl-cc-border-style` | `solid` | Thumbnail border style |
| `--sdl-cc-border-color` | `transparent` | Thumbnail border color |

### Typography — Title

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-title-font-size` | `var(--heading-3-font-size)` | Title font size |
| `--sdl-cc-title-font-family` | `var(--heading-font-font-family)` | Title font family |
| `--sdl-cc-title-color` | `var(--headingMediumColor)` | Title text color |
| `--sdl-cc-title-text-align` | `start` | Title alignment |

### Typography — Excerpt

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-excerpt-font-size` | `var(--paragraph-font-size)` | Excerpt font size |
| `--sdl-cc-excerpt-color` | `var(--paragraphMediumColor)` | Excerpt text color |
| `--sdl-cc-excerpt-text-align` | `start` | Excerpt alignment |

### Typography — Price (Shop)

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-price-font-size` | `var(--paragraph-font-size)` | Price font size |
| `--sdl-cc-price-color` | `var(--paragraphMediumColor)` | Price text color |

### Typography — Event Dates

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-event-dates-font-size` | `var(--paragraph-font-size)` | Event date font size |
| `--sdl-cc-event-dates-color` | `var(--paragraphMediumColor)` | Event date text color |

### Typography — Published Date

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-published-date-font-size` | `0.9rem` | Published date font size |
| `--sdl-cc-published-date-color` | `var(--paragraphMediumColor)` | Published date color |

### Typography — Author

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-author-font-size` | `0.9rem` | Author font size |
| `--sdl-cc-author-color` | `var(--paragraphMediumColor)` | Author text color |

### Categories

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-category-background-color` | `transparent` | Category pill background |
| `--sdl-cc-category-border-radius` | `0` | Category pill radius |
| `--sdl-cc-category-border-width` | `0` | Category pill border width |
| `--sdl-cc-category-border-style` | `solid` | Category pill border style |
| `--sdl-cc-category-border-color` | `transparent` | Category pill border color |
| `--sdl-cc-category-padding` | `0` | Category pill padding |
| `--sdl-cc-category-color` | `var(--paragraphMediumColor)` | Category text color |
| `--sdl-cc-category-font-size` | `0.9rem` | Category font size |

### Tags

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-tag-background-color` | `transparent` | Tag pill background |
| `--sdl-cc-tag-border-radius` | `0` | Tag pill radius |
| `--sdl-cc-tag-border-width` | `0` | Tag pill border width |
| `--sdl-cc-tag-border-style` | `solid` | Tag pill border style |
| `--sdl-cc-tag-border-color` | `transparent` | Tag pill border color |
| `--sdl-cc-tag-padding` | `0` | Tag pill padding |
| `--sdl-cc-tag-color` | `var(--paragraphMediumColor)` | Tag text color |
| `--sdl-cc-tag-font-size` | `0.9rem` | Tag font size |

### Navigation Arrows

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-arrow-color` | `var(--list-section-carousel-arrow-color, #000)` | Arrow SVG stroke color |
| `--sdl-cc-arrow-size` | `26px` (mobile: `20px`) | Arrow icon size |
| `--sdl-cc-arrow-background-color` | `var(--list-section-carousel-arrow-background-color, #fff)` | Arrow button background |
| `--sdl-cc-arrow-background-size` | `40px` (mobile: `40px`) | Arrow button diameter |
| `--sdl-cc-arrow-background-border-radius` | `50%` | Arrow button shape |
| `--sdl-cc-arrow-background-opacity` | `0.7` | Arrow button opacity (1 on hover) |
| `--sdl-cc-navigation-prev-offset` | `0px` | Left margin for prev button |
| `--sdl-cc-navigation-next-offset` | `0px` | Right margin for next button |
| `--sdl-cc-navigation-spacing` | `17px` | Gap between navigation elements (top/bottom layouts) |
| `--sdl-cc-navigation-top-padding` | `17px` | Top padding when navigation is placed top or bottom |
| `--sdl-cc-navigation-overlay-padding` | `2vw` | Horizontal padding for overlay navigation |

### Pagination Bullets

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-bullet-size` | `8px` | Bullet diameter |
| `--sdl-cc-active-bullet-color` | inherits from Squarespace | Active bullet color |
| `--sdl-cc-active-bullet-opacity` | `1` | Active bullet opacity |
| `--sdl-cc-inactive-bullet-color` | inherits from Squarespace | Inactive bullet color |
| `--sdl-cc-inactive-bullet-opacity` | `0.5` | Inactive bullet opacity |
| `--sdl-cc-pagination-top-padding` | `17px` | Space above pagination |

### Layout

| Variable | Default | Description |
|---|---|---|
| `--sdl-cc-max-page-width` | `100%` | Max width, synced from Squarespace `maxPageWidth` tweak |
| `--sdl-cc-page-padding` | `2vw` | Page horizontal padding, synced from Squarespace `pagePadding` tweak |

---

## Navigation Layouts

Set via `data-navigation-layout`:

| Value | Behavior |
|---|---|
| `overlay` *(default)* | Arrows float over the slides at vertical center |
| `overlay image` | Arrows float over the thumbnail image only (height tracks image height) |
| `overlay images` | Alias for `overlay image` |
| `top left` | Arrows above the carousel, aligned left |
| `top right` | Arrows above the carousel, aligned right |
| `top center` | Arrows above the carousel, centered |
| `top spread` | Arrows above the carousel, spread to edges |
| `top start` | Alias for `top right` |
| `top end` | Alias for `top left` |
| `bottom left` | Arrows below, left-aligned; pagination pushed right |
| `bottom right` | Arrows below, right-aligned; pagination pushed left |
| `bottom center` | Arrows below, centered alongside pagination |
| `bottom spread` | Arrows below, spread to edges |

When any `bottom` layout is used, the pagination is integrated into the navigation row rather than appearing separately below it.

---

## Slide Effects Reference

Set via `data-effect`:

| Value | Description | Notes |
|---|---|---|
| *(none)* | Standard slide | Default |
| `fade` | Cross-fade between slides | 1 slide visible at a time |
| `flip` | 3D flip on vertical axis | 1 slide visible at a time |
| `cube` | 3D cube rotation | 1 slide visible at a time |
| `coverflow` | 3D coverflow (iTunes style) | Auto-enables `centeredSlides` |
| `cards` | Stacked card effect | |
| `creative` | Fully custom transform | Configure via `data-creative` JSON |
| `creative-coverflow` | Preset creative effect: 3D rotated coverflow | Shorthand — auto-converts to `creative` |
| `creative-coverflow-2` | Preset creative effect: scaled peek | Shorthand — auto-converts to `creative` |

### Creative Effect JSON structure

```json
{
  "limitProgress": 3,
  "progressMultiplier": 1,
  "prev": {
    "opacity": 0.75,
    "translate": ["-110%", 0, 0],
    "scale": 0.9,
    "rotate": [0, -10, 0]
  },
  "next": {
    "opacity": 0.75,
    "translate": ["110%", 0, 0],
    "scale": 0.9,
    "rotate": [0, 10, 0]
  }
}
```

---

## Collection Types & Data Mapping

The plugin auto-detects the collection type from the Squarespace API response's `collection.typeLabel` field.

| Squarespace typeLabel contains | Internal type | Special behavior |
|---|---|---|
| `"portfolio"` | `portfolio` | Standard items |
| `"products"` | `products` | Price display, sale price, multi-variant support, 25+ currencies |
| `"blog"` | `blog` | Standard items |
| `"event"` | `event` | Upcoming/past merge, date formatting, 40-item default limit |
| `"lessons"` | `video` | Standard items |
| `"course"` | `course` | Standard items |

### Pagination / Load More

The plugin automatically follows Squarespace's `pagination.nextPageUrl` until `limit` items have been collected, so carousels with more than 20 items will fetch multiple pages transparently.

### Product Price Features

- Displays lowest variant price with "from" prefix when variants differ in price
- Sale price shown with strikethrough on original price
- 26 currencies supported with correct prefix/suffix and symbol
- Currency detected from `priceMoney.currency` on variant data

### Event Date Formatting

Configure via the `eventDateFormat` global setting (cannot be set per-element via data attributes; use `window.sdlCollectionCarouselSettings`):

```javascript
window.sdlCollectionCarouselSettings = {
  eventDateFormat: {
    showYear: true,
    timeFormat: "12",        // "12" or "24"
    locale: "default",       // e.g. "en-US", "fr-FR"
    options: {
      weekday: "short",
      month: "short",
      day: "numeric",
      hour: "numeric",
      minute: "2-digit"
    }
  }
};
```

Same-day events display as: `Mon, Jan 1, 9:00 AM - 5:00 PM`
Multi-day events display as: `Mon, Jan 1, 9:00 AM - Tue, Jan 2, 5:00 PM`

---

## Content Order System

Control the order of every content element per slide using `data-content-order` as a JSON array:

```html
data-content-order='["metadataAboveTitle","title","metadataBelowTitle","price","eventDates","excerpt","metadataBelowExcerpt"]'
```

Available tokens:

| Token | Renders |
|---|---|
| `title` | Item title (h3) |
| `excerpt` | Item excerpt |
| `price` | Product price (Shop only) |
| `eventDates` | Event date range (Events only) |
| `metadataAboveTitle` | Metadata group above title |
| `metadataBelowTitle` | Metadata group below title |
| `metadataBelowExcerpt` | Metadata group below excerpt |

Only tokens present in `contentOrder` will be rendered, regardless of whether the individual setting (e.g. `title: true`) is enabled.

---

## Metadata System

Metadata fields are placed into named groups: **above title**, **below title**, **below excerpt**. Within each group, items are rendered in the order specified in the comma-separated attribute value, separated by the `metadataDelimiter`.

### Available field names (case-insensitive)

| Field name | Renders | Applicable collections |
|---|---|---|
| `publisheddate` | Publication date | Blog, Portfolio, Events |
| `publishdate` | Alias for publisheddate | — |
| `date` | Alias for publisheddate | — |
| `author` | Author display name | Blog |
| `categories` | Category list | Blog, Shop |
| `tags` | Tag list | Blog, Portfolio |

### Example: author and date above title, categories below

```html
data-metadata-above-title="author, date"
data-metadata-below-title="categories"
data-metadata-delimiter=" · "
data-metadata-category-links="true"
```

---

## Loop Strategies

When `loop` is enabled and grouped slides (`groupSlides` or `slidesPerGroup > 1`) are used, the total item count may not divide evenly by the group size. Two strategies are available:

| Strategy value | Behavior |
|---|---|
| `keep-blanks` *(default)* | Loop runs with blank padding slides added by Swiper |
| `disable-loop-when-uneven` | Loop is silently disabled if item count is not divisible by group size |

```html
data-group-slides-loop-strategy="disable-loop-when-uneven"
```

### Loop Fill Items

When you have fewer items than needed to fill the view in loop mode, enable `loopFillItems` to automatically duplicate items until there are enough:

```html
data-loop="true"
data-loop-fill-items="true"
```

---

## Full-Width Mode

When `data-full-width="true"` (or auto-detected), the plugin calculates the offset between the Squarespace content column and the viewport edge and applies it as `slidesOffsetBefore` / `slidesOffsetAfter`, allowing slides to bleed to the viewport edge while the first slide aligns to the content column.

This offset is recalculated on window resize.

**Fluid Engine support:** When the element is inside a Fluid Engine grid, offset is measured from a temporary block placed at `gridArea: 1 / 2 / 2 / -2`.

Disable the offset while keeping full-width layout:

```html
data-full-width="true"
data-disable-full-width-offset="true"
```

---

## Caching System

Collection data can be cached in `localStorage` to reduce API calls:

```html
data-cache-duration="60"  <!-- cache for 60 minutes -->
```

- Cache key: `sdlPlugins.<sanitized_source_url>`
- Meta key: `sdlPlugins.<sanitized_source_url>_meta` (stores timestamp)
- Fresh data is always fetched and written to cache after expiry
- If `cacheDuration` is `0`, caching is fully disabled

---

## Global Configuration

Set defaults for all carousels on the page before the plugin initialises:

```html
<script>
window.sdlCollectionCarouselSettings = {
  // Any default setting key
  limit: 12,
  loop: true,
  autoplay: 5000,
  navigationLayout: "overlay",
  eventDateFormat: {
    locale: "en-GB",
    timeFormat: "24",
    showYear: false
  }
};
</script>
```

Settings priority (highest wins): **per-element data attributes** → **global user settings** → **plugin defaults**

---

## JavaScript Events API

The plugin dispatches two custom events on the carousel element:

| Event | When | `event.detail` |
|---|---|---|
| `sdlCollectionCarousel:beforeInit` | Before data fetch and DOM build | The class instance |
| `sdlCollectionCarousel:ready` | After Swiper is initialised | The class instance |

### Listening to events

```javascript
document.addEventListener("sdlCollectionCarousel:ready", (e) => {
  const carousel = e.detail;
  console.log("Carousel ready:", carousel.settings.id);
  console.log("Swiper instance:", carousel.swiper);
  console.log("Items:", carousel.items);
});
```

### Accessing instance programmatically

After initialisation, each element exposes its instance:

```javascript
const el = document.getElementById("my-carousel");
const { settings, swiper, items } = el.sdlCollectionCarousel;

// Control Swiper directly
swiper.slideNext();
swiper.autoplay.start();
```

---

## CSS Architecture

### Scoping

All plugin CSS is scoped to `[data-sdl-plugin="collection-carousel"]` to prevent bleed into other page elements.

### Generated DOM structure

```
[data-sdl-plugin="collection-carousel"]  ← host element
│
├── .navigation-wrapper                   ← (top layout: order -1)
│   ├── .navigation-button-prev
│   │   └── button
│   │       ├── .swiper-button-background
│   │       └── <svg> (prev arrow)
│   └── .navigation-button-next
│       └── button
│           ├── .swiper-button-background
│           └── <svg> (next arrow)
│
├── .swiper.collection-carousel           ← Swiper container
│   └── .swiper-wrapper
│       └── .swiper-slide.collection-carousel-slide  (× N)
│           ├── .slide-thumbnail
│           │   ├── img
│           │   └── a.slide-thumbnail-link           (if clickthrough)
│           └── .slide-content
│               ├── .content-metadata-above-title.metadata-group
│               ├── .content-title
│               │   └── h3 (> a if clickthrough)
│               ├── .content-metadata-below-title.metadata-group
│               ├── .content-price                   (Shop only)
│               ├── .content-event-dates             (Events only)
│               ├── .content-excerpt
│               └── .content-metadata-below-excerpt.metadata-group
│
└── .collection-carousel-pagination        ← (bottom layout: order 2)
    └── .pagination-wrapper
```

### Overriding styles for a specific instance

```css
#my-carousel {
  --sdl-cc-aspect-ratio: 16/9;
  --sdl-cc-arrow-background-color: #2C2319;
  --sdl-cc-arrow-color: #F5F0E8;
  --sdl-cc-title-color: #2C2319;
}
```

---

## Squarespace Editor Notifications

The plugin shows notification banners **only inside the Squarespace editor** (when `.sqs-edit-mode` is present on the body). These appear for:

- **Too few items for loop mode** — warns that `slidesPerViewLg` has been reduced, and auto-reduces the value.
- **Uneven groups in loop mode** — warns that blank slides will be added (when strategy is `keep-blanks`).

These messages are never shown to site visitors.

---

## Complete Implementation Examples

### Blog carousel — 3 columns, overlay arrows, with date and author

```html
<div
  data-sdl-plugin="collection-carousel"
  data-source="/blog"
  data-limit="9"
  data-slides-per-view-sm="1"
  data-slides-per-vies-dld="2"
  data-slides-per-view-lg="3"
  data-space-between-sm="20"
  data-space-between-lg="34"
  data-aspect-ratio="16/9"
  data-navigation-layout="overlay"
  data-metadata-above-title="publisheddate"
  data-metadata-below-title="author, categories"
  data-metadata-delimiter=" · "
>
</div>
```

### Portfolio carousel — full-width, coverflow effect

```html
<div
  data-sdl-plugin="collection-carousel"
  data-source="/portfolio"
  data-full-width="true"
  data-slides-per-view-sm="1.2"
  data-slides-per-vies-dld="2.2"
  data-slides-per-view-lg="3.2"
  data-effect="coverflow"
  data-coverflow-rotate="30"
  data-coverflow-scale="0.85"
  data-loop="true"
  data-autoplay="4000"
  data-navigation-layout="overlay image"
  data-pagination-type="bullets"
>
</div>
```

### Shop carousel — 4 columns, bottom navigation layout

```html
<div
  data-sdl-plugin="collection-carousel"
  data-source="/shop"
  data-limit="20"
  data-slides-per-view-sm="2"
  data-slides-per-vies-dld="3"
  data-slides-per-view-lg="4"
  data-group-slides="true"
  data-space-between-lg="24"
  data-aspect-ratio="3/4"
  data-navigation-layout="bottom spread"
  data-pagination-type="fraction"
  data-title="true"
  data-price="true"
  data-excerpt="false"
>
</div>
```

### Events carousel — upcoming only, with dates below title

```html
<div
  data-sdl-plugin="collection-carousel"
  data-source="/events"
  data-events="upcoming"
  data-slides-per-view-sm="1"
  data-slides-per-vies-dld="2"
  data-slides-per-view-lg="3"
  data-aspect-ratio="4/3"
  data-event-dates="true"
  data-navigation-layout="top right"
  data-pagination="false"
  data-content-order='["title","eventDates","excerpt"]'
>
</div>
```

### Blog carousel — minimal style, no arrows, auto-scrolling

```html
<div
  data-sdl-plugin="collection-carousel"
  data-source="/blog"
  data-limit="10"
  data-slides-per-view-sm="1.1"
  data-slides-per-vies-dld="2.1"
  data-slides-per-view-lg="3"
  data-free-mode="true"
  data-navigation="false"
  data-pagination="false"
  data-loop="true"
  data-autoplay="2500"
  data-autoplay-disable-on-interaction="false"
>
</div>
```

### Custom CSS styling example

```css
#portfolio-carousel {
  --sdl-cc-aspect-ratio: 4/5;
  --sdl-cc-border-radius: 4px;
  --sdl-cc-title-font-size: 1rem;
  --sdl-cc-title-color: #2C2319;
  --sdl-cc-excerpt-color: #6B5F52;
  --sdl-cc-arrow-background-color: #2C2319;
  --sdl-cc-arrow-color: #F5F0E8;
  --sdl-cc-arrow-background-opacity: 0.9;
  --sdl-cc-arrow-background-border-radius: 0px;
  --sdl-cc-active-bullet-color: #2C2319;
  --sdl-cc-inactive-bullet-color: #2C2319;
  --sdl-cc-inactive-bullet-opacity: 0.3;
}
```

---

*Generated from `plugin.js` (sdlCollectionCarousel class + Swiper 11.2.1) and `plugin.css`*
