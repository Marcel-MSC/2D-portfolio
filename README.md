# 2D Portfolio

An interactive, game-like personal portfolio built as a top-down 2D scene in the style of classic Pokémon / Zelda games. Walk around a pixel-art bedroom, click on objects to learn about me, then exit through the door to a city-selection map to explore some places I'm connected to.

**Live demo (works on mobile):** https://2dportfoliomarcelomsc.netlify.app/

## Goal of the project

The goal of this project is to present my portfolio in a way that's more fun, memorable and personal than a traditional CV/landing page. Instead of scrolling through static sections, the visitor:

- **Moves a character around** a pixel-art room representing my workspace.
- **Interacts with objects** (PC, TV, bed, books, resume, etc.), each one triggering a dialogue box with information about me, my background, the tools I use and links to my GitHub, resume and contact.
- **Exits to a city selection scene** with castle markers for places that matter to me (e.g. Ouro Fino - MG and São Paulo - SP), where each castle opens a dialogue with more context.

It is also a learning playground for building games in the browser with **Kaboom.js**, bundling with **Vite**, and shipping a small but production-ready static site with CI.

## Tech stack

| Category | Tool | Why it's used |
|---|---|---|
| Game engine | [**Kaboom.js**](https://kaboomjs.com/) `^3000.1.17` | Lightweight 2D game library for JavaScript. Handles the canvas, scenes, sprites, animations, collisions, camera and input with a simple component-based API — ideal for a small top-down game like this. |
| Build tool / dev server | [**Vite**](https://vitejs.dev/) `^7` | Fast dev server with hot module replacement and a tiny, optimized production build. The project ships as a fully static site. |
| Minifier | **Terser** `^5` | Used by Vite for JS minification in production builds. |
| HTML sanitization | [**DOMPurify**](https://github.com/cure53/DOMPurify) `^3` | Dialogue text is rendered as HTML (so it can contain links to GitHub, my resume, etc.) outside the canvas. DOMPurify sanitizes that HTML before injecting it into the DOM. |
| Languages | **JavaScript (ES modules)**, **HTML5**, **CSS3** | Pure vanilla JS — no framework. The whole UI overlay (textbox, close button, "tap to move" hint) is plain HTML/CSS layered on top of the `<canvas>`. |
| Assets / tiles | **Tiled-style JSON maps** + **spritesheet** | Map boundaries, spawn points and named collision objects are defined in `map.json` and `map-cities.json` and consumed at runtime. The character and props come from a single sprite atlas (`spritesheet.png`) sliced into a 39×31 grid. |
| Font | `monogram.ttf` | Pixel-style font bundled via CSS `@font-face` so dialogues match the retro aesthetic. |
| Hosting | **Netlify** | Static hosting for the production build. |
| CI | **GitHub Actions** | A smoke-test workflow (`.github/workflows/ci.yml`) runs `npm ci` + `npm run build` on every push / PR to `main` / `master` to make sure the project always builds. |

## How it works (architecture)

The codebase is intentionally small and split by responsibility:

```
src/
├── main.js          # Entry point. Loads sprites + map, sets background, starts the "main" scene.
├── kaboomCtx.js     # Initializes the Kaboom context (k) shared by every module.
├── constants.js     # Scale factor + dialogue texts for the bedroom and for the cities.
├── player.js        # createPlayer(): sprite + collision area + body + speed/direction state.
├── utils.js         # displayDialogue() (typewriter effect + DOMPurify) and setCamScale().
├── index.css        # Loads the pixel font and styles the HTML dialogue UI.
└── scenes/
    ├── main.js        # Bedroom scene: loads map.json, wires up boundaries → dialogues, "exit" → city select.
    └── citySelect.js  # City select scene: draws castle sprites + labels, wires up city boundaries → dialogues, "voltar" → main.
public/
├── map.png, map.json            # Bedroom tilemap + collision/spawn data.
├── map-cities.json              # City scene collision/spawn data (no background image; rendered on a black canvas).
├── spritesheet.png              # Player + props + castle sprites (39x31 grid).
└── console-controller.svg       # Favicon.
```

Key ideas:

- Each scene **fetches its map JSON at runtime**, iterates over layers, and creates Kaboom entities for `boundaries` (static collision rectangles, optionally named) and `spawnpoints`.
- Named boundaries (`pc`, `tv`, `bed`, `resume`, `exit`, `ouro_fino`, `sao_paulo`, `voltar`, …) drive the interaction logic — colliding with them either opens a dialogue (`displayDialogue` in `utils.js`) or transitions to another scene (`k.go("citySelect")` / `k.go("main")`).
- Movement is **click / tap to move**: the player walks toward the cursor / touch point. `touchToMouse: true` in `kaboomCtx.js` means touch events become mouse events automatically, so the same code works on desktop and mobile.
- Dialogue text appears with a **typewriter effect** and is sanitized through DOMPurify before being injected, since some dialogues contain `<a>` links.

## Run locally

Requirements: **Node.js 20+** and **npm**.

```bash
npm install
npm run dev
```

Then open the URL printed in the terminal (typically `http://localhost:5173`).

### Other scripts

| Command | What it does |
|---|---|
| `npm run dev` | Start the Vite dev server with hot reload. |
| `npm run build` | Produce a minified static build in `dist/`. |
| `npm run preview` | Locally serve the production build for a final sanity check. |
| `npm test` | Currently an alias for `npm run build` — used as a build smoke test in CI. |

## Assets

Static assets — favicon, bedroom tilemap and the shared spritesheet — live in `public/` so Vite serves them as-is.

- The bedroom uses `map.png` as the background plus `map.json` for boundaries and spawn points.
- The **city selection** scene has **no background image**: it renders on a black canvas with castle sprites pulled from the spritesheet and city names drawn as text in code. Only `map-cities.json` is used, and only for boundaries and the player spawn point.
- The font `monogram.ttf` lives in `src/` and is bundled with the CSS at build time.

## Deployment

The production build (`npm run build`) outputs a fully static `dist/` directory that can be deployed to any static host. This project is currently deployed to **Netlify** (see the live demo link at the top).

## Continuous Integration

A minimal GitHub Actions workflow (`.github/workflows/ci.yml`) installs dependencies with `npm ci` and runs `npm run build` on every push and pull request targeting `main`/`master`. If the build breaks, the check fails — that's the project's safety net for now.

## Credits & inspiration

- **Kaboom.js:** https://kaboomjs.com/
- **Original tutorial that started this project:** [Create your own 2D portfolio (YouTube)](https://www.youtube.com/watch?v=gwtfWORCN0U)

## About

This is my first attempt at a game-like portfolio. I've always had a passion for games and a dream of making them professionally one day — for now, this is a small step in that direction, and a fun way to introduce myself.

Got a job opportunity or just want to chat? **marcelomarcos.s.c@gmail.com**
