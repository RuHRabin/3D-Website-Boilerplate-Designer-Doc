# Getting started

This page takes you from an empty folder to a running 3D site, then explains how the skill works so you can steer it.

## Before you begin

- Node.js 18 or newer, and npm.
- An agent that supports skills, with this skill installed. For Claude Code, place the skill folder in `~/.claude/skills/` (all projects) or `.claude/skills/` (one project).

## Your first project

1. **Describe the site.** You do not need technical terms. Any of these work:
   - "Build a 3D landing page for my studio with a rotating logo."
   - "I want the camera to move through scenes as people scroll."
   - "Let customers rotate my product and switch colors."
   - "Add a subtle animated background to my site."
2. **Confirm the pattern.** If you describe an effect instead of naming a pattern, the skill matches it to the closest of the four and checks with you first, because each one produces quite different code.
3. **Scaffold.** The skill runs its bundled script and creates the project folder.
4. **Run it.**
   ```bash
   cd your-project
   npm install
   npm run dev
   ```
   The dev server opens in your browser. Edits to `src/` update the page as you save.
5. **Customize.** Ask the skill to change the geometry, colors, lighting or copy. It works from the checklist for your pattern (see [Patterns](patterns.md)).
6. **Build.**
   ```bash
   npm run build
   ```
   The `dist/` folder is a static site. Upload it to Netlify, Vercel, GitHub Pages, S3 with CloudFront, or any web server.

## How the skill works

| Step | What happens |
|---|---|
| 1. Stack and pattern | The skill asks only what is unclear from context: whether the site is React or not, and which pattern fits. |
| 2. Scaffold | It runs `scripts/scaffold.mjs` to generate the project from tested templates. |
| 3. Pattern walkthrough | It reads the matching pattern guide with you: what is included, what to customize, what people commonly get wrong. |
| 4. Real assets | If you have a logo or product model, it reads the asset guide first so compression and format are right. |
| 5. Performance and accessibility pass | Before calling the site done, it checks device scaling, reduced motion and loading states. |

Scaffolding from templates, instead of writing a scene from memory each time, matters because Three.js projects have many details that are easy to get subtly wrong: color space, tone mapping, pixel ratio, cleanup, and import paths that change between versions.

## Choosing a stack

Both stacks give you the same four patterns and the same visual results. Pick by what the rest of the site uses.

**Choose vanilla Three.js with Vite when:**

- The site is not a React app: plain HTML, another framework, a CMS, or a WordPress theme.
- The 3D scene is the entire page, with little interface state around it.
- The person maintaining the site prefers imperative code (`scene.add(mesh)` and a hand-written render loop) to React components and hooks.

**Choose React Three Fiber when:**

- The rest of the site is React or Next.js.
- The 3D content has interface around it that changes over time, such as a configurator with several linked controls.
- The team already knows React and would rather write `<mesh><boxGeometry /></mesh>`.

| | Vanilla Three.js | React Three Fiber |
|---|---|---|
| Building the scene | Imperative: `new THREE.Mesh(...)`, `scene.add(...)` | Declarative JSX |
| Render loop | Hand-written `requestAnimationFrame` loop | `useFrame` hook, managed by the canvas |
| State | Manual DOM listeners | React state and hooks |
| Helpers | Import from `three/examples/jsm/*` | `@react-three/drei` components |
| Bundle size | Smaller baseline | Larger baseline: `three`, `react`, `@react-three/fiber`, `@react-three/drei` |

Dependencies the templates pin:

| Stack | Dependencies |
|---|---|
| Vanilla | `three` ^0.169.0, `vite` ^5.4.8 |
| React Three Fiber | `three` ^0.169.0, `@react-three/fiber` ^8.17.10, `@react-three/drei` ^9.114.3, `react` ^18.3.1, `react-dom` ^18.3.1, `@vitejs/plugin-react` ^4.3.2, `vite` ^5.4.8 |

## Adding 3D to an existing Next.js app

Do not run the scaffold for this. It creates a standalone project. Instead:

1. Use the React Three Fiber pattern's `App.jsx` as reference code, not as a drop-in file.
2. Make any component that uses `<Canvas>`, `useFrame` or drei hooks a Client Component by adding `"use client"` at the top of the file in the Next.js App Router. These libraries need browser APIs that do not exist during server rendering.
3. Install the dependencies listed above into your existing project. Do not copy the template's `package.json` wholesale.

## When to use something else

This skill covers the 3D scene: setup, camera, lighting, assets, interaction and performance. For layout, typography and other non-3D interface work, use regular frontend tooling.

## Next

- [Patterns](patterns.md) for what each starter includes and how to customize it.
- [Scaffold command reference](cli-reference.md) if you want to run the script yourself.
- [Assets and performance](assets-and-performance.md) before you launch.
