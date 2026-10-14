# Scaffold command reference

The skill runs `scripts/scaffold.mjs` for you. You can also run it yourself from the skill folder.

```bash
node scripts/scaffold.mjs \
  --stack <vanilla-threejs|react-three-fiber> \
  --pattern <hero-scene|scroll-story|product-configurator|particle-background> \
  --out <output-directory> \
  --name "<Project Name>"
```

## Options

| Option | Required | Values | Notes |
|---|---|---|---|
| `--stack` | Yes | `vanilla-threejs`, `react-three-fiber` | Which framework the project uses. |
| `--pattern` | Yes | `hero-scene`, `scroll-story`, `product-configurator`, `particle-background` | Which scene to generate. |
| `--out` | Yes | A folder path | Created if it does not exist. |
| `--name` | No | Any text | Used as the page `<title>` and, in slug form, as the npm package name. Defaults to the output folder name. |
| `--force` | No | (flag) | Scaffold into a folder that already has files in it. |

## What the script does

1. Copies the common files for your stack: `package.json`, `vite.config.js`, base HTML and CSS.
2. Copies the pattern's files on top. A pattern file with the same path as a common file **replaces** it completely. It is not merged.
3. Replaces the `__PROJECT_NAME__` and `__PROJECT_SLUG__` placeholders in `.json`, `.html`, `.js`, `.jsx`, `.ts`, `.tsx`, `.css` and `.md` files.
4. Prints the next steps.

The slug is your name in lowercase, with anything other than letters, digits and hyphens turned into hyphens. An empty result falls back to `my-3d-site`, because an npm package name cannot be empty.

## Examples

A hero scene in plain JavaScript:

```bash
node scripts/scaffold.mjs --stack vanilla-threejs --pattern hero-scene --out ./orbit-studio --name "Orbit Studio"
```

A React product configurator:

```bash
node scripts/scaffold.mjs --stack react-three-fiber --pattern product-configurator --out ./chair-shop --name "Chair Shop"
```

After either one:

```bash
cd orbit-studio
npm install
npm run dev
```

## Project scripts

| Command | Does |
|---|---|
| `npm run dev` | Starts the Vite dev server and opens the browser |
| `npm run build` | Builds a static site into `dist/`, with source maps |
| `npm run preview` | Serves the built `dist/` folder locally |

## Errors

| Message | Cause and fix |
|---|---|
| `--stack is required and must be one of: ...` | Missing or misspelled stack. Use one of the two listed values. |
| `--pattern is required and must be one of: ...` | Missing or misspelled pattern. |
| `--out is required (output directory)` | Add `--out <folder>`. |
| `... already exists and is not empty. Pass --force ...` | The folder has files in it. Choose a new folder, or add `--force` if you mean it. |
| `Missing template directory: ...` | The skill folder is incomplete. Reinstall the skill. |
