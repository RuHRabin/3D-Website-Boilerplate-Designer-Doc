# Assets and performance

3D sites tend to fail in one predictable way: they look great on the developer's desktop and then stutter, drain the battery or crash the tab on a mid-range phone. Work through this page after scaffolding and before launch.

## Bringing in a real model

The hero and configurator templates use procedural placeholder shapes so the project runs with no files. They are not meant to ship. Replace them with your own assets.

### Format

Use **glTF** (`.gltf` with separate files) or its single-file form **GLB** (`.glb`). Both Three.js and React Three Fiber load it natively. Convert other formats first:

- **Blender:** File → Export → glTF 2.0, format "glTF Binary (.glb)".
- **Other tools:** most (Cinema 4D, Maya, SketchUp) have a glTF exporter. Otherwise export FBX or OBJ and convert with Blender or `obj2gltf`.

### Compress before shipping

A GLB straight from a modeling tool is often five to ten times larger than it needs to be. Two kinds of compression matter, and they are independent:

| Compression | What it shrinks | Command (with `@gltf-transform/cli`) |
|---|---|---|
| Draco or Meshopt | Geometry | `gltf-transform draco input.glb output.glb` |
| KTX2 / Basis Universal | Textures | `gltf-transform etc1s input.glb output.glb`, or `uastc` for higher quality and larger size |

Install the tool with `npm install -g @gltf-transform/cli`. Blender's exporter also has a built-in Draco option.

Texture compression is usually the biggest single win for load time and GPU memory.

**Target:** under 2 to 3 MB for a hero or landing model after compression. A configurator model people zoom into can be larger. Going from a 15 MB to a 3 MB file is the difference between a 2-second and a 15-second load on an average mobile connection.

### Loading it

- **Vanilla Three.js:** see the commented-out block in the `product-configurator` template's `src/main.js`. It shows `GLTFLoader`, `DRACOLoader` and `LoadingManager` together, including the progress bar. The Draco loader needs a path to the decoder files. The template points at Google's CDN. Self-host them if the project cannot depend on an outside CDN.
- **React Three Fiber:** use `useGLTF` from `@react-three/drei`. It handles Draco without extra setup. Call `useGLTF.preload(path)` outside the component to start loading early.

### Standalone textures and environments

- Prefer WebP over PNG or JPG.
- Size a texture to its on-screen footprint. A 4096 px texture on an object drawn at 200 px wastes bandwidth and memory. Stick to power-of-two sizes (512, 1024, 2048) for clean mipmaps.
- For reflections, a compressed `.hdr` or `.exr` environment at 1K to 2K resolution is enough for most cases. A full 8K HDRI is only worth it for a photoreal hero.
- Put large binary files in `public/` and reference them with an absolute path such as `/models/product.glb`. Files in `src/` go through the JavaScript bundler.

## Performance budget

| Area | Guidance |
|---|---|
| Pixel ratio | Keep the cap of 2 that the templates set (`Math.min(window.devicePixelRatio, 2)` in vanilla, `dpr={[1, 2]}` in React Three Fiber). The particle pattern lowers it to 1.5 on low-power devices. Rendering at a phone's native 3x costs a lot for a difference few people see. |
| Device capability | A small screen can still have a weak GPU. `navigator.hardwareConcurrency` and `navigator.deviceMemory` are blunt but useful. Safari does not support `deviceMemory`, so keep a safe default. The templates default to 4. |
| Triangles and draw calls | As a starting target for a hero or landing scene on mobile, stay under about 100,000 triangles and well under 50 draw calls. Merge geometry or use instancing for repeated objects. Profile your own scene and treat this as a check, not a rule. |
| Bundle size | The React Three Fiber stack commonly produces a 250 to 300 KB gzipped bundle even for a simple scene. About 270 KB was measured for the `scroll-story` template. |
| Hidden canvases | The particle templates pause when the tab is hidden. Extend this with `IntersectionObserver` if a canvas can scroll out of view on a long page. |

Reducing the React Three Fiber bundle:

- Use `React.lazy()` and `Suspense` to load the 3D scene after the rest of the page.
- Import only the drei helpers you use, for example `import { OrbitControls } from "@react-three/drei"`.
- In vanilla Three.js, import only the specific `examples/jsm/*` modules you need.

## Reduced motion

Every template checks `window.matchMedia("(prefers-reduced-motion: reduce)")` and turns off autorotate, parallax and continuous animation, while still showing a still frame of the real content. This is an operating-system accessibility setting for people with vestibular disorders and motion sensitivity.

- Reduced motion means "stop moving", not "stop existing". Never hide content behind the check.
- If you add scroll-triggered camera moves or flythroughs, gate the animation on the flag and keep the content visible.

## Loading states

A blank or grey canvas during a model load looks broken. The hero and configurator templates include a loading screen driven by `THREE.LoadingManager` (vanilla) or `<Suspense>` (React Three Fiber).

- Connect `LoadingManager.onProgress` to the loading bar width so people see real progress. The commented-out loader block shows how.
- In React Three Fiber, anything inside `<Suspense fallback={...}>` shows the fallback while `useGLTF` or `useTexture` load. Pass a real fallback, not `null`, if loading may take more than a second or two.

## Accessibility beyond motion

- **Decorative canvas.** If the canvas is only decoration, as in `particle-background`, keep `aria-hidden="true"` so screen readers skip it.
- **Canvas as content.** In a hero or configurator, put the same information in text on the page. A screen reader user should still learn what product, brand or message the scene shows. The skill cannot write this for you because it depends on your content.
- **Controls.** Keep interface controls as real HTML elements layered over the scene, not drawn on the canvas. The configurator's swatches are `<button>` elements with `aria-label`s and a visible `:focus-visible` outline.
- **No JavaScript or no WebGL.** Every `index.html` has a `<noscript>` block with real content. WebGL can also be unavailable with JavaScript on, for example under a security policy, on an old device or in a scraper. If the 3D scene is the main content, wrap the `WebGLRenderer` constructor in `try/catch` and show a static image or text instead of a dead black canvas.

## Deploying

`npm run build` writes static files to `dist/`. Any static host works: Netlify, Vercel, GitHub Pages, S3 with CloudFront, or a plain Apache or Nginx server.

Two things trip people up:

- `vite.config.js` sets `base: "./"`, so the build uses relative paths and works in a subfolder such as a GitHub Pages project site at `username.github.io/repo-name/`. Remove it, or set `base: "/"`, if you deploy to a domain root and prefer absolute paths.
- Large models and textures belong in `public/`, as described above.

## Launch checklist

- [ ] The pixel-ratio cap is still in place.
- [ ] If you raised particle or object counts, device-capability scaling still applies.
- [ ] Turning on "reduce motion" in your operating system stops motion and keeps the scene visible.
- [ ] Every real asset has a loading state wired to real progress.
- [ ] Models are compressed, and the total is within your target.
- [ ] The scene's message also exists as text on the page.
- [ ] You tested on a real mid-range phone, not only your own desktop.
- [ ] Placeholder geometry has been replaced with your real assets.
