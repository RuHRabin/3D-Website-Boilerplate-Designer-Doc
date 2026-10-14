# Patterns

The skill ships four starters. Each one is available in both stacks. Pick by what the site needs to do, then use the checklist to make it yours.

| Pattern | One line |
|---|---|
| [`hero-scene`](#hero-scene) | A 3D object fills the first screen |
| [`scroll-story`](#scroll-story) | The camera moves as people scroll |
| [`product-configurator`](#product-configurator) | People rotate a product and change it live |
| [`particle-background`](#particle-background) | Ambient motion behind normal content |

---

## hero-scene

A 3D object, or a small group of objects, fills the viewport and rotates gently. This is the classic agency homepage opener.

**Good for:** landing pages, portfolios, product announcements, anywhere the first two seconds matter most.
**Weaker for:** content-heavy pages that need text and navigation immediately. The full-bleed canvas competes with them. Try a smaller contained canvas or `particle-background` instead.

**Included**
- A centerpiece icosahedron with standard PBR materials, lit by a warm key light and a cool fill light.
- Orbit controls set to auto-rotate with zoom and pan **disabled**, so the object can never be lost.
- A loading screen. It does nothing until you replace the placeholder with a real model.
- Reduced-motion support: rotation stops and the object stays visible.

**Customize**
1. **Replace the geometry.** Swap in another primitive (`TorusKnotGeometry`, `DodecahedronGeometry`) for a fast reskin, or load a real glTF model.
2. **Tune the material.** Roughness near 0 with metalness near 1 reads as chrome. Higher roughness with low metalness reads as matte plastic or ceramic. Switch to `MeshPhysicalMaterial` for glass or clearcoat.
3. **Adjust the lighting.** Lighting sets the mood more than geometry does. Lower the ambient light for a darker brand, raise it for a brighter one.
4. **Decide how pointer input works.** For a subtle lean toward the cursor instead of drag-to-rotate, replace orbit controls with a lerp toward the pointer position.
5. **Choose the background.** A solid color with fog performs best. An environment map adds realism to reflective materials at the cost of one more file to load.

**Common mistakes**
- Enabling zoom or pan. People scroll the page and accidentally zoom the scene.
- Using four or more lights. Two or three well-placed lights look better and render faster.
- Adding a real model without wiring up load progress, which brings back a blank canvas for a couple of seconds.

---

## scroll-story

Normal page content scrolls over or beside a fixed 3D scene. The camera follows a defined path, so each section of text lines up with a camera angle.

**Good for:** product stories with a handful of distinct beats, feature walkthroughs, "our process" pages.
**Weaker for:** long text-heavy pages, or beats that do not map to clear camera positions.

**Included**
- A `KEYFRAMES` array that maps scroll progress (0 to 1) to a camera position and look-at target, interpolated between the two nearest keyframes.
- Vanilla: a scroll listener throttled with `requestAnimationFrame`. React Three Fiber: drei's `<ScrollControls>` and `useScroll()`.
- Three placeholder objects, each visited by one keyframe.
- Text sections synced to the same scroll.

**Customize**
1. **Write the real beats first, then the camera path.** Keyframe `progress` values should match where each text section is in view.
2. **Add or remove keyframes freely.** The interpolation walks the array, so no other code changes. Four keyframes suit a three or four section story.
3. **Use section height for pacing.** In vanilla, taller HTML sections mean more scroll per beat. In React Three Fiber, set `pages` on `<ScrollControls>`. It is `4` as scaffolded.
4. **Consider easing.** Linear interpolation can feel mechanical. Apply an ease-in-out to `t` for a more cinematic reveal.
5. **Watch off-screen objects.** They still render. For many heavy objects, toggle visibility or lean on frustum culling.

**Common mistakes**
- Layering text over the canvas without `pointer-events: none`, so scroll and drag gestures are swallowed. Both templates already set it on `.story`.
- Packing too many keyframes into too little scroll. Give each transition about one viewport height.
- Removing the content sections in vanilla. With nothing below the canvas, there is nothing to scroll, and the camera never moves.

---

## product-configurator

A product model people can orbit and zoom, with controls that change it in real time. Color swatches are included.

**Good for:** furniture, footwear, custom hardware, jewelry, any product where seeing it from several angles or finishes helps a purchase decision.
**Weaker for:** products where photos say more than a render, or teams that cannot commit to producing and maintaining a real 3D model. A configurator on placeholder geometry looks unfinished.

**Included**
- A procedural bottle-like placeholder, with a commented-out `GLTFLoader` or `useGLTF` block ready for a real model.
- Orbit controls with `minDistance`, `maxDistance` and `maxPolarAngle` limits, so the camera cannot pass through the object or dip below the ground.
- A ground shadow: `ShadowMaterial` in vanilla, `<ContactShadows>` in React Three Fiber.
- Color swatches as real `<button>` elements with `aria-label`s that change the material color directly.

**Customize**
1. **Replace the placeholder with your model.** Compress it first. See [assets and performance](assets-and-performance.md).
2. **Decide what is configurable.** Color is the simple case. Material swaps need several prepared materials. Swapping whole parts needs a model built with those parts as separate meshes, which is a modeling decision made before the web work.
3. **Target the right materials.** On a real model, the loader block includes a `model.traverse(...)` step so that only the body responds to color and not, say, the hardware.
4. **Add a reset-view button** if there are many options. It is easy to leave a product at an awkward angle.
5. **Connect price and SKU.** The pattern covers only the 3D half. Update your price or cart state in the same handlers that change the color.

**Common mistakes**
- Removing the zoom and angle limits, so people zoom through the model or lose it off-screen.
- Leaving auto-rotate running while someone inspects a detail. Consider pausing it on first touch or drag.
- Changing only the 3D scene and never the surrounding page state.

---

## particle-background

A light animated backdrop, a drifting point field, sitting behind ordinary HTML. It supports the page and is not the content.

**Good for:** SaaS landing pages, event sites, any page that wants texture and movement behind marketing copy.
**Weaker for:** anything that must show a product or tell a visual story. A point field carries no specific meaning.

**Included**
- A `THREE.Points` field (vanilla) or `<points>` (React Three Fiber) with the particle count reduced on weaker devices, using `navigator.hardwareConcurrency` and `navigator.deviceMemory`.
- A gentle upward drift plus soft pointer parallax. Both stop under `prefers-reduced-motion`.
- `aria-hidden="true"` and `pointer-events: none`, so the layer never interferes with clicks or screen readers.
- Rendering that pauses when the browser tab is hidden.

**Customize**
1. **Match motion to the brand.** Slow, sparse, large particles read as calm and premium. Fast, dense, small ones read as energetic and technical. Tune speed, size and `PARTICLE_COUNT` together.
2. **Consider another shape.** Points are the cheapest option. `THREE.InstancedMesh` allows small meshes, letters or icons at a modest cost.
3. **Stay near-monochrome.** One accent color at low opacity with additive blending gives a cheap glow.
4. **Check contrast** against your real copy. Add a semi-transparent scrim behind text if the particles get too close to the text color or too dense.
5. **Test tall and wide screens.** The field is fixed behind everything. Confirm it still looks intentional at unusual aspect ratios.

**Common mistakes**
- Skipping the device-capability check because it looks fine on a laptop. This is the most common way 3D backgrounds ruin mobile performance, since the background runs on every page load.
- Forgetting `aria-hidden` and `pointer-events: none`.
- Leaving the render loop running in a hidden tab.
