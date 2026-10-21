# How Santioni Spirits moves: observations, inferences and decisions

On 2026-10-20 we studied the public files and the runtime behaviour of santionispirits.com. Active Theory built the site on their Hydra engine. We used three tools. The first was a static analysis with the REA 6.7.0 CLI (parent Evidence `ev_924209725ec6a64d2aaecad86cc603bee964eace0681bedb53f3ce464eda1305`, shortened below to `ev_9242…1305`). The second was REA runtime commands (capture-browser-scenario, inspect-web-page, list-browser-targets). The third was our own CDP probes driving headless Chrome. Each claim below was checked by a second, adversarial verification pass. Only confirmed or corrected observations are kept, and corrected ones show the corrected value.

Legal note: we studied only techniques and behavioural parameters. We did not copy or ship any code, shaders, curve data, audio, fonts or textures. Everything in HayPaComer is written from scratch. Short identifiers and numeric constants appear here only so the findings can be checked.

How evidence is cited: `ev_9242…1305 @ app.js byte N` gives a byte offset in the public bundle. `compiled.vs section Name` points to the public shader file. `uil.json key K` points to the tuned-parameter file. `cdp-probe file@line` points to a number our own probe measured. Full REA IDs mark runtime captures.

## 1. Questions and status

| # | Question | Status |
|---|---|---|
| Q1 | Wheel input: normalization, scale constants, clamps, block flags | answered |
| Q2 | Smoothing and inertia, frame-rate dependence, touch release | partial (no touch or high-Hz runtime) |
| Q3 | Keyboard, exempt elements, resize, scroll-to-scene mapping, story length | answered |
| Q4 | Scene sequencing, preload, show/hide, disposal, render passes | partial (no draw-call runtime per scene change) |
| Q5 | Camera keyframes, scroll binding, lerps, parallax, FOV | partial (no runtime camera timing) |
| Q6 | Hatching: object-space lines, thresholds, dots, stepped time | answered |
| Q7 | Ink transition: TransitionLineShader and BorderShader uTransition | answered |
| Q8 | Wind, ground and portal curve sketches: data, draw-on, scroll link | partial (no runtime timings) |
| Q9 | Captions: reveal, timing source, DOM vs WebGL, ARIA | answered |
| Q10 | HOLD interaction: duration, visual, release, keyboard, audio, gating | partial (static only, no runtime hold) |
| Q11 | Audio engine, loading, mute, crossfades, one-shots, formats | answered |
| Q12 | Performance: DPR caps, tiering, fps, preload, frame budget | answered |
| Q13 | Mobile: same 3D, touch, orientation, reduced features | answered |
| Q14 | Accessibility: reduced motion, keyboard, screen reader, focus | answered |

Totals: 9 answered, 5 partial, 0 unknown.

## 2. Scroll

### Observations

- SCRO-01: The story uses a virtual scroller with no DOM scroll element (`Scroll.object` is null). Story creates it with drag on mobile only, with mouse wheel, limit and keyboard on, with scale 1, touchScale 1 and multiTouch on. It is published as `Global.SCROLL`. (ev_9242…1305 @ app.js byte 1093951, ~1754560, 1758305; cdp-probe work/scroll/s0-eval.mjs@1)
- SCRO-02 (corrected): The wheel listener sits on window (on document.body in capture mode for IE) and is not added on mobile. The story scroller never calls preventDefault on wheel, because that branch also needs a DOM scroll object. A second window wheel listener only fires `Global/wheel`. (ev_9242…1305 @ app.js byte ~1101200, 1763207)
- SCRO-03: Wheel normalization depends on OS and browser. macOS Chrome/Safari: x0.33 with inertia. macOS Firefox in line mode (deltaMode 1): x4 with inertia, otherwise raw. Windows Firefox in line mode: x10. Windows Chrome: x0.25. IE: x1 with inertia. Every other case: raw, no inertia. At runtime the first-frame target matched 0.33 x 100 plus one 0.9 inertia step. (ev_9242…1305 @ app.js byte ~1096600-1097700, 1096764, 1097191)
- SCRO-04: Each wheel event first stops inertia, so a stream of events keeps restarting inertia from the latest one. A single deltaY 100 travelled 330 px (329.98). deltaY 300 travelled 990 px. A burst of 10 x 100 spaced 16 ms apart travelled 1351 px. A trackpad-like curve whose deltas sum to 258 travelled 295 px. (cdp-probe work/scroll/s4-measure.mjs@17-20)
- SCRO-05: With limit on, the target is clamped every frame to [0, max.y/scale]. max.y is the story root height minus the stage height. At 1440x813 the End key reached 27469 = 28282 - 813. (ev_9242…1305 @ app.js byte ~1094900, ~1758340; cdp-probe s4-measure.mjs@24)
- SCRO-06 (corrected): `blockUntilNewScroll` acts as a momentum gate. While block is set, the loop zeroes target, delta and y. A wheel event clears it only once hardBlock (200 ms) has expired and its scaled impulse (not the raw deltaY) is larger than the previous one. The "larger or equal" rule applies only while a lifetime delta-change counter is at 3 or below, and that counter is never reset. At runtime, deltas 50 to 10 left position at 0, and a later deltaY 80 released the block and scrolled 264 px. The bundle defines this function but never calls it. (ev_9242…1305 @ app.js byte 1097805, 1102496; cdp-probe work/scroll/s5-throttle-block.mjs@20)
- SCRO-07: Input is ignored over elements with an ancestor class `prevent_interactionScroll`, when the scroller is disabled, outside optional bounds, or when lockMouseX is set. Wheel checks use the global mouse position, not the event's coordinates. (ev_9242…1305 @ app.js byte 1094047, ~1758100, ~1726700)
- SCRO-08: In the scroll loop, the inertia is multiplied by 0.9 each frame and added to the target. Then y moves by delta = 0.5 x (target - y). The reported delta is zeroed under 0.01 (after the move), and |y| < 0.001 snaps to 0. Nothing in this loop is scaled by delta time. (ev_9242…1305 @ app.js byte 1094782)
- SCRO-09 (corrected): Story adds a second smoothing stage. A lerped scroll moves toward -scroll.y with alpha 0.08 on desktop and 0.9 on mobile, normalized as alpha' = 1 - exp(ln(1 - alpha) x FRAME_HZ_MULTIPLIER), where FRAME_HZ_MULTIPLIER = 60 / (1000/DELTA) x refreshScale (default 1). The smoothed scroll delta also pitches the camera: target 0.002 rad per px, lerp 0.05, clamped to +/-0.2 rad. (ev_9242…1305 @ app.js byte 1753619, ~3260, 25460, ~1758700)
- SCRO-10 (corrected): Headless Chrome ran at a median 33.3 ms per frame. After one wheel deltaY 100, scroll.y reached 50/90/99% of its final value at 228/732/1464 ms, and the story lerp reached the same marks at 430/999/1764 ms. With deltaY 300 the scroll.y marks were 237/735/1470 ms. The isInertia flag never clears on its own; only the next stopInertia clears it. (cdp-probe s4-measure.mjs@17-18)
- SCRO-11: Touch (mobile only): a press stops inertia, and each drag event subtracts the mouse delta from the target. Release tweens the target by -delta x 25 (35 on Android) over 2500 ms with easeOutQuint, unless preventInertia is set. touchstart is preventDefault-ed except over exempt elements. (ev_9242…1305 @ app.js byte ~1098300-1099100, 1098786)
- SCRO-12: On mobile the story lerp is 0.9, so it adds almost no lag; the smoothness comes from the release tween and the 0.5 approach. (ev_9242…1305 @ app.js byte 1753619)
- SCRO-13: Keyboard (desktop only): ArrowUp/Down move +/-150 px, PageUp/PageDown move one stage height, Home/End go to 0/max, Space is PageDown and Shift+Space is PageUp. Each is a 400 ms easeOutCubic tween. Measured: the target settled within 0.5 px at 364-402 ms, and the story lerp reached 99% at about 1065 ms. The handler does not check for a focused input and does not call preventDefault. (ev_9242…1305 @ app.js byte ~1099300, 154166; cdp-probe s4-measure.mjs@21-24)
- SCRO-14 (corrected): On resize, the story's Scroll class only stops inertia; it returns early because it has no DOM object. The px position is therefore kept, not the fraction. Story's handleResize (debounced 200 ms) recomputes world sizes, unitsPerPixelY = tan(fov/2) x |z| x 2 / Stage.height (0.003878 at fov 35, z 5, height 813), scene layouts and max.y. It also sets gaze movement to 0 on mobile or below 768 px width. (ev_9242…1305 @ app.js byte ~1100000, ~1762247, 1765660)
- SCRO-15 (corrected): Each frame, the lerped scroll times unitsPerPixelY becomes `Story/scrollY`. That value drives the composite uniform uScrollY and the y of the main and fixed camera groups. 31 shader or component bindings read Story/scrollY. Some of them add +0.2 to the discard top in portrait. (ev_9242…1305 @ app.js byte ~1758450, 1358058, 1401218)
- SCRO-16: 18 scenes are stacked from a config list, with heights in screen units. The mobile height is interpolated by Stage.width from 1600 (desktop value) to 393 (mobile value), clamped. "auto" heights come from the DOM. (ev_9242…1305 @ app.js byte ~1755900)
- SCRO-17: Per-scene progress maps the lerped scroll from "scene bottom enters" to "scene top leaves" onto 0..1, unclamped. Visibility is an overlap test with 0.1 world-unit padding; the top and bottom padding names are swapped in the call. (ev_9242…1305 @ app.js byte 1391717)
- SCRO-18: At 1440x813 the story root is 28282 px (34.8 screens) and max scroll is 27469 px. Wander is 2.5 x 813. Approach is 1.9006 x 813 = 1545 px (interpolated). Cathedral is 3252 px with a -406.5 px margin. (cdp-probe s0-eval.mjs@1, reproduced by arithmetic)
- SCRO-19: Scroll progress also drives audio and voice-over gating: VO waits until |delta| <= 0.6 for 100 ms. Scrolling is enabled 1500 ms after `Global/loaderFinished`. (ev_9242…1305 @ app.js byte ~1759000, 2035373, ~1763300)
- SCRO-20: scrollToTop disables input, tweens over 800 to 3000 ms mapped from 1 to 10 screens (easeOutCubic), and re-enables input after 200 ms while the tween keeps running. (ev_9242…1305 @ app.js byte ~1755000)

### Inferences

- One wheel notch travels about 10 x factor x delta in total: 3.3x deltaY on Mac, 2.5x on Windows. Confidence high (SCRO-03, SCRO-04: 330 measured against 330 predicted).
- The scroll loop depends on frame rate. At 120 Hz, inertia and approach finish twice as fast in wall time as at 60 Hz, though the total distance is the same. Only the story lerp is normalized. Confidence high (static; could not be shown at runtime because headless was locked to 30 Hz).
- At 60 Hz the inertia half-life is about 6.6 frames (110 ms), and the story lerp half-life at 0.08 is about 139 ms. Confidence medium.
- Touch release uses only the last event's delta, with no velocity window. Confidence high.
- All story animation is a pure function of smoothed scroll, so it can be scrubbed and reversed. The only exceptions are one-shot animateIn flags. Confidence high.

### Unknowns

- The Firefox and Windows wheel branches were read only statically. CDP cannot dispatch deltaMode 1.
- Runtime touch drag and release were not measured on mobile.
- Behaviour at 60, 120 and 144 Hz was not measured.
- Disputed: the perf group found a code path that keeps the scroll fraction on mobile resize (byte ~1099700). The scroll verifier showed that this path returns early for the story scroller. We treat fraction preservation as absent.
- How native page scroll is suppressed (presumably CSS plus a Mobile preventNativeScroll) was not traced.

### Decision for HayPaComer

Normalize wheel input in one function: pixel-mode deltas x0.3, line-mode deltas at 16 to 40 px per line, and seed inertia with the same impulse. Do no browser sniffing beyond deltaMode. Use one time-based smoothing stage (pos += (target - pos) x (1 - exp(-dt/tau)), inertia v x= 0.9^(dt/16.67)) so that 60 and 120 Hz screens feel the same. For touch, average velocity over the last 50 to 100 ms and glide for 1.5 to 2.5 s. Clamp every frame. Keep a "block until a new gesture" flag for programmatic jumps, with a counter that resets. Keep the story position as a fraction across resizes. Keyboard: Arrow 150 px, PageUp/PageDown/Space one viewport, Home/End, 400 ms ease-out, ignored while a form field has focus. Keep a small pitch from scroll velocity (0.002 rad/px, clamped at 0.2 rad). To detect "user is scrolling", use delta, not an inertia flag.

## 3. Scenes and camera

### Observations

- SCEN-01: All 18 scenes are created at startup (order 0 Wander to 17 Footer), and Story waits for all of them. None are created lazily. (ev_9242…1305 @ app.js byte 1749105)
- SCEN-02: All scenes share one world, stacked vertically. Each scene group sits at y = 0.5 x screenH - totalHeight - 0.5 x heightWorld. Each scene's DOM block gets a CSS height of Stage.height x height, so the page length matches the 3D stack. (ev_9242…1305 @ app.js byte 1755929, 1390257)
- SCEN-03 (corrected): Preload uploads every geometry, texture and program of each scene to the GPU. The uploads start before `Global/scenesReady` but scenesReady does not wait for them; the global loader (which leads to loaderFinished) does. (ev_9242…1305 @ app.js byte 1395134)
- SCEN-04 (corrected): Visibility is toggled every frame by band overlap. A hidden scene is skipped by the renderer, and it also pauses all of its per-frame callbacks and offscreen passes, because startRender checks every ancestor's visibility. GPU resources stay allocated. (ev_9242…1305 @ app.js byte 1391711, 642671, 46847)
- SCEN-05 (corrected): Only about 9 scenes read the per-scene scrollProgress; about 40 consumers, mostly shaders, read the global Story/scrollY. Wander uses its progress for footstep volume (in over 0.33-0.5, out over 0.7-1). The drone mix uses the progress of Approach, Cathedral, AntiGravity and Taste. (ev_9242…1305 @ app.js byte 1391711, 2054480)
- SCEN-06 (corrected): No story scene is ever disposed. dispose has 2 hits, both in framework code. destroy() owners are UI and framework classes only. (static grep)
- SCEN-07 (corrected): Scenes do not crossfade. A scene change is spatial: neighbours overlap (marginTop -0.1 Transition, -0.5 Cathedral), and TransitionScene is a full-width woodcut-line quad with renderOrder 1e5 and depth off. Directional transition audio is flagged only for newly visible scenes listed in SCENE_TRANSITION_AUDIO. (ev_9242…1305 @ app.js byte 1828450, 1826690, 1763550)
- SCEN-08: Frame order: AFTER_LOOPS (the Story loop: scroll, camera, visibility, audio), then BEFORE_RENDER (uniform updates), then render, then POST_RENDER. The renderer is Hydra's own, not three.js. (ev_9242…1305 @ app.js byte 1047382)
- SCEN-09: The world renders into a post chain (MSAA where allowed), followed by one full-screen composite pass. That pass reads lines, cloud noise and blue noise textures, uAgeGate, uLoaderFinished, uMenuHover, uDPR and uScrollY, plus mouse fluid when it is enabled. FXAA is used only when MSAA is off. (ev_9242…1305 @ app.js byte 1753855, 1786997)
- SCEN-10 (corrected): Extra render targets, each active only while its scene is visible. Collection: text RT plus liquid RT (both screen size) plus a 0.8x blur pair, 8 blur passes per frame. Products: text RT plus 8 blur passes. Hand: an FX scene plus an info layer, and on desktop an inverse FX scene at 0.25 with its own pass. HandScene also pre-warms by forcing 10 frames at init. (ev_9242…1305 @ app.js byte 1584022, 1422141, 1426063, ~1588288)
- SCEN-11: Startup: two loop passes run with scroll disabled. On loaderFinished, uLoaderFinished tweens to 1 over 1200 ms (easeOutSine, 3000 ms delay), scroll is enabled after 1500 ms, and the frame loop starts. Resize is debounced 200 ms. (ev_9242…1305 @ app.js byte 1763396, 1763464, 1764499)
- SCEN-12: Production uses no Theatre.js or keyframe timelines: 0 calls to TweenUIL.create or TimelineUIL.create, the loader is never defined, and uil.json has no tween or timeline keys. (ev_9242…1305 @ app.js byte 1243240; uil.json scan)
- SCEN-13 (corrected): The main story camera is a GazeCamera at z 5, fov 35, moveXY 0.5. A second fixed camera follows scroll without parallax or tilt. Its fov assignment has no effect (it stays at 30), which is harmless because only its view matrix feeds uFixedCameraMatrix. (ev_9242…1305 @ app.js byte 1765994, 314284)
- SCEN-14: Scroll drives the camera directly with no keyframes. The lerped scroll in world units is written to the camera group y, so one screen of scroll equals one screen of world. (ev_9242…1305 @ app.js byte 1753619, 1765660)
- SCEN-15: Scroll-velocity pitch: rotation.x approaches 0.002 x the lerped delta, with alpha 0.05, clamped to +/-0.2 rad. (ev_9242…1305 @ app.js byte 1759031)
- SCEN-16: Mouse parallax maps the pointer to -1..1. The target position = position + m x moveXY x strength. That target is lerped with lerpSpeed2 = 1, and the camera position follows it with lerpSpeed 0.05, then lookAt. moveXY is 0.5 on desktop and 0 on mobile or below 768 px (class default 4). (ev_9242…1305 @ app.js byte 324203, 325372, 1761856)
- SCEN-17: Every lerp is frame-rate normalized, so the constants are per 60 Hz frame. (ev_9242…1305 @ app.js byte 3300, 25482)
- SCEN-18: Roll from mouse velocity and procedural wobble are both off in the story (deltaRotate 0, wobbleStrength 0 by class default). The 0.07 lerp belongs to the inner-group roll. (ev_9242…1305 @ app.js byte 327457, 327556)
- SCEN-19 (corrected): FOV stays fixed at 35. The 75 CAMERA_* uil keys belong to sub-cameras and playground cameras. The only zoom move is in AntiGravity: 1 to 1.1 over 6000 ms with cubic-bezier(0.52, 0.02, 0.02, 1), and back. The user's drawing interaction triggers it, not scroll. (ev_9242…1305 @ app.js byte 1346134, 1336776)
- SCEN-20: On desktop only, DOM elements with data-tilt follow the camera parallax: translate (-80, 20) x 0.5 and rotate (-20, 5) x 0.5. (ev_9242…1305 @ app.js BaseView.applyTilt ~1394500)
- AntiGravity shakes the scene root, not the camera, by 0.0035 x sin/cos(10t) times a multiplier that tweens 0 to 1 over 3000 ms on draw. Config.JUMP_TO_SCENE and ANCHOR deep-link by setting the lerped scroll directly, so the camera does not fly through. (verifier notes, scenes-camera)

### Inferences

- Draw calls peak at scene boundaries, where two scenes and the transition quad are all visible. Confidence medium (not measured per boundary).
- The stacked world is cheap mainly because hidden scenes stop all their per-frame work. Its cost is memory: every scene stays resident. Confidence medium.
- At 60 fps, lerpSpeed 0.05 has a half-life of about 225 ms and reaches 95% in about 1 s; the scroll lerp 0.08 has a half-life of about 140 ms. Confidence high.
- Maximum desktop parallax is +/-0.5 world units at a screen height of about 3.15 units, about 16% of the screen. Confidence medium.
- Santioni has no JSON camera keyframes. HayPaComer's keyframed cameras are our own addition. Confidence high.

### Unknowns

- Draw calls, framebuffer binds and timing across a scene change (the runtime probe was blocked).
- The internal render-target count of the post chain, frustum culling coverage, the space lookAt works in, and whether the raw zoom tween refreshes the projection matrix.

### Decision for HayPaComer

Stack market, fridge, scale and plate as world-space bands in one three.js scene, described by a JSON list (height in screen units, mobile height, margin). Toggle `group.visible` per band with about 0.1 units of padding, and skip that band's per-frame work while it is hidden. Upload everything behind the press-and-hold gate (renderer.compile, initTexture) and pre-warm each band for a few frames so nothing hitches on first show. Make transitions spatial overlaps plus a woodcut-line quad, not crossfades. Use one PerspectiveCamera (fov 35, z 5) in a group whose y follows the lerped scroll. Add our JSON keyframes per band (position, lookAt, fov), sampled by band progress with our own easing, beneath a +/-0.5 parallax (lerp 0.05, off on phones). Keep FOV moves rare and slow. Unlike Santioni, dispose far bands on phones.

## 4. Hatching and ink

### Observations

- SHAD-01: The line coordinate comes from object space: the raw position is rotated by a per-material axis and angle, and its xy is the line UV, so strokes stick to the surface. Lighting is also in object space: the unrotated normal is dotted with a fixed light direction, so the light turns with the mesh. (compiled.vs section StaticObjectBaseShader.glsl)
- SHAD-02: Density = UV x uLinesTile. Optional distance compensation scales the UV by 0.5 x mix(2, 0.5, clamp(depth x 0.0325)^2): 1.0 near, 0.25 from a depth of 30.77 onward, blended by uDistanceCompensation. (compiled.vs section StaticObjectBaseShader.glsl)
- SHAD-03: Stepped time: floor(t x 8)/8 x 0.15, applied as an x shift of -2x that value. That gives 0.3 UV/s in 8 jumps per second, a hand-drawn boil. The line texture is sampled with swapped axes and remapped to -1..1. (compiled.vs section StaticObjectBaseShader.glsl)
- SHAD-04: Two thresholds make three tones. mid = aastep(th.x, N·L + 0.45 x lines - 0.15 x ao - vgrad). high = aastep(th.y, N·L + 0.1 x lines - 0.15 x ao - vgrad). The colour goes ink 18/255, then uColor, then uColorHighlight. (compiled.vs section StaticObjectBaseShader.glsl)
- SHAD-05: Highlights break into dots. The stroke mask = aastep(0.2, lines + max(0, L) + noise x max(0, L) + 0.3 x L), with noise sampled at 2x the line UV. Mask 0 gives ink. (compiled.vs section StaticObjectBaseShader.glsl; ev_9242…1305 @ app.js byte 1744570)
- SHAD-06: aastep half-width = 0.7071 x the screen-space gradient length, so edges stay about one pixel soft at any zoom. (compiled.vs sections StaticObjectBaseShader.glsl, BorderShader.glsl)
- SHAD-07 (corrected): alpha = 1 - red, computed before the 18/255 floor. That gives ink about 0.93, the default mid tone about 0.32 and white 0. Whether this blends depends on the material's transparent flag, which was not confirmed; the screenshots suggest the result is opaque. The NoDiscard variant has alpha 1, no distance compensation, ao weight 1.0 and no section discard. (compiled.vs sections StaticObjectBaseShader.glsl, StaticObjectNoDiscard.glsl)
- SHAD-08: Each mesh is clipped to its scene band: fragments outside the NDC-height range [uDiscardBottom, uDiscardTop] are discarded. The range is recomputed from Story/scrollY. (compiled.vs section StaticObjectBaseShader.glsl; ev_9242…1305 @ app.js byte 1744570)
- SHAD-09: The outline is a true inverted hull: a second back-side mesh at renderOrder +1, extruded along the clip-space normal by uLineWidth x sqrt(w) x 2, drawn flat in 18/255. uLineWidth runs from 0.0005 to 0.009 per layer. (compiled.vs section StaticObjectBaseShaderInverse.glsl; ev_9242…1305 @ app.js byte 1362449)
- SHAD-10: Defaults: uLinesTile 4.5, thresholds (0.4, 0.7), light normalize(-0.75, 0.75, 2), axis normalize(0, 0.5, 2), angle -0.5, distance compensation 0, vertical gradient effectively off, mid tone #ae9c74, highlight #ffffff. (ev_9242…1305 @ app.js byte 1744570)
- SHAD-11 (corrected): Per-material tuning varies widely: 23 threshold entries across 24 elements, x from -2.71 to 1.73, y from -0.43 to 1.57, uLinesTile from 0.025 to 12.08. 7 pairs are inverted (x > y), and in those the mid band never appears. uDistanceCompensation has 11 entries: eight at 1, one at 0, one at 0.252 and one at 10. (uil.json StaticObjectBaseShader keys)
- SHAD-12 (corrected): Across all shaders there are 49 threshold pairs (plus one scalar); the most common is (0.38, 0.61), used 9 times. uLinesTile has 78 values from -1.507 to 30, most often 1.05. (uil.json scan)
- SHAD-13: On the live page, strokes follow the surface, curved strokes on a sphere break into specks toward the light, and stairs carry hatching at per-object angles. (cdp-probe work/shaders/s3.mjs@1, shots/04-wheel.png; REA ev_c96cac0e5c18977560cca8444904e434383f22cf75b05a9c301a7f60b6a7601d)
- SHAD-14: The ink transition is a scene of its own, order 6: 2 screens tall (1.5 on mobile), marginTop -0.1. The next scene (Cathedral, height 4) starts with marginTop -0.5. (ev_9242…1305 @ app.js byte 1755915, 1750656)
- SHAD-15: The cover is one plane, 2x screen width by baseHeight screens, transparent, depth off, renderOrder 1e5 (JS overrides the UIL value of 0). (ev_9242…1305 @ app.js byte 1828459; uil.json INPUT_Config_0_TransitionScene_*)
- SHAD-16 (corrected): A paper value runs from 1 at the top of the plane to 0 at v = 0.2. The plane is solid ink below v of about 0.36. Above that it is hatched ink wherever noise + 0.65 x lines - paper > 0.4. The top 5% stays clean, and the bottom alpha ramps over v 0 to 0.12 with ragged noise. (compiled.vs section TransitionLineShader.glsl)
- SHAD-17: The edge offset is time x 0.1 plus scrollY x 0.0002, so the scroll term is negligible and time drives the boil. uRepeat maps Stage.width 320..1600 to 0.4..1 without a clamp (about 1.15 at 1920). (ev_9242…1305 @ app.js byte 1826699)
- SHAD-18 (corrected): Measured at 1440x813: first sparse ink at about y 9100, bottom third covered at 9400, mid-screen at 9700, about 90% black at 10000, Cathedral showing from 10300. The ink rises from the bottom. (cdp-probe work/shaders/p2.mjs@3, recomputed luminance)
- SHAD-19: The edge keeps boiling at rest: about 16% of pixels change by more than 64 levels per ~1.3 s. (cdp-probe work/shaders/p3.mjs@2)
- SHAD-20 (corrected): BorderShader uTransition opens a panel frame like an iris. Pads go from the full half-size at 0 to halfSize x uPad at 1 (default pad 0.3, tuned from -1 to 1.94). Inner rim vertices are pushed inward by 0.1 x inner. Width is capped at 2100 CSS px. (compiled.vs section BorderShader.glsl)
- SHAD-21: The border edge ink is clouds noise scrolled at 1 UV/s in 1/8 s steps, subtracted at weight 0.6, with a colour threshold of 0.4 and an alpha threshold of 0.01. Mouse-fluid erosion applies only on desktop with uTransition > 0.95 and a non-zero edge setting (the default is off). (compiled.vs section BorderShader.glsl; ev_9242…1305 @ app.js byte 1396962)
- SHAD-22: uTransition is set to 0 on animateSet and tweens to 1 over 800 ms easeOutCubic when the section comes into view (once, unless continuous). (ev_9242…1305 @ app.js byte 1366688, 1388150)
- SHAD-23: No swap event exists; the overlapping bands hand off while the cover is solid. (ev_9242…1305 @ app.js byte 1391500, 1759153)

### Inferences

- An engraving material needs no post-process: planar object-space line UV, two aastep terminators with different line weights, a dot-dissolving mask and an 8 fps drift. Confidence high.
- Thresholds were tuned by eye per material, so they belong in scene data. Confidence high.
- The cover is positional (scroll sets coverage) while time only animates the edge, so reverse scroll works naturally. It spans about 1.5 viewports of scroll. Confidence high for the first point, medium for the span.
- The Border iris is a separate time-based reveal, not part of the cover. Confidence high.

### Unknowns

- What lines.jpg, perlin.png and clouds_noise.png contain, how the ao attribute was baked, what negative uLinesTile values mean, and the blending flag of the base material.
- Edge position at steps finer than 300 px, cover behaviour on mobile, and drone sync with the cover.

### Decision for HayPaComer

Write one `engraved` ShaderMaterial for three.js r169: object-space planar line UV with a per-mesh axis and angle, uLinesTile, optional distance compensation, two thresholds (default 0.4/0.7) with line weights 0.45/0.1, a stroke mask that turns into dots with N·L plus noise, and an 8 fps stepped drift. Choose world-space lighting on purpose so the light does not spin with props. Add a back-side inverted-hull outline in our ink colour. Read thresholds, tile and angle per object from the scene JSON. Build the market to fridge to scale to plate hand-offs as ink-cover planes about 2 screens tall (1.5 on phones), drawn last with depth off. Use a gradient plus our own noise and stroke textures with a 0.4 threshold and a solid ink zone at the base, overlap the next band by half a screen, and boil the edge at about 0.1 UV/s. Panel frames get an 800 ms ease-out iris reveal on in-view. No mouse-fluid erosion. Under adaptive quality, phones drop the noise sample and distance compensation first. All textures are drawn by us.

## 5. Drawn lines

### Observations

- LINE-01: Every curve file has the shape data.curves[].position, a flat xyz array. There are no width, time or uv fields. The loader derives uv.y = i/(n-1), sets uv.x to 0 or 1 for the two ribbon sides, and mirrors the end neighbours. (work/lines/stats.py; ev_9242…1305 @ app.js byte ~1312000-1314500)
- LINE-02: Counts: portal-wind 8 curves, 4342 points, arc 50.8-61.6. common wind 19 curves, 1339 points. pillarcrumble 12 curves, 416 points. outward 11 segments, 22 points. wander ground 18 curves, 680 points, on y = 0. ground-vertical 125 curves, 917 points (some single-point curves become degenerate ribbons). wander wind 7 curves, 584 points, arc about 20. (work/lines/stats.py)
- LINE-03: Most files are resampled at a fixed step: portal 0.100, pillarcrumble 0.130, ground 0.199, wander wind 0.247, with coefficient of variation at or below 0.06. common wind and ground-vertical are uneven. (work/lines/spacing.py)
- LINE-04: Ribbons use currpos/prevpos/nextpos attributes. The tangent is the averaged projected neighbour direction, corrected for aspect, and vertices are offset by thickness k x clip.w, so the width is constant on screen (k = 0.003 for wind, 0.01 for sketch). That is 2 vertices per point (portal 8684), with a Uint16 index. (compiled.vs sections WindLineShader, WindLinesSketchShader)
- LINE-05 (corrected): Dashes come from noise sampled along the line: step(uThreshold + 0.4 x scrollFactor, noise). The noise offset is -floor(t x uFrameRate)/uFrameRate x mix(0.075, 0.2, rand) x uSpeed + rand x 2.23 x uTile. uSpeed defaults to 0, so there is no boil unless a scene sets it. Only the portal, outward and wander files use WindLineShader; common wind and pillarcrumble use WindDustShader. Alpha is a tent across the width, discarded below 0.26 and hardened with aastep at 0.25. (compiled.vs section WindLineShader; ev_9242…1305 @ app.js byte 1363678, 1437139, 1650233)
- LINE-06 (corrected): scrollFactor = 1 - min(1, 0.5 - 0.25 x uScroll). It is 0 for uScroll <= -2, 0.5 at 0, 1 at 2, and keeps rising above 2. uScroll is the raw story scroll in world units, so where the erosion happens depends on each scene's world position. (compiled.vs sections WindLineShader, DrinkLineShader; ev_9242…1305 @ app.js byte 1758732)
- LINE-07: Optional drift: with uAnimatePosition = 1, x moves from -5 to +15 and wraps every 20 units, at 0.725 units/s (27.6 s per loop) or 0.525 (38.1 s). Only ground-curves-vertical enables it (x span 40, scaled 1.25). (compiled.vs; ev_9242…1305 @ app.js byte 2051520)
- LINE-08: The sketch shader is two-tone, ink against white, split at uv.x 0.5 with a cnoise(pos x 1.1) x 0.3 jitter. Its clock is uTime x 6 stepped at uFrameRate x 0.2 and ignores uSpeed. Both shaders clip to the scene's NDC band. (compiled.vs section WindLinesSketchShader)
- LINE-09: DrinkLine has a real draw-on. uAnimate 0 to 1 grows the line along uv.y behind a smoothstep front as wide as the thickness; 1 to 2 retracts it from the start. Progress multiplies the width, so the line collapses rather than being discarded. The width tapers over 5% at each end, and the line wobbles with two sines in clip space. Selecting a bottle tweens it 0 to 1 over 600 ms easeOutCubic; bottles that are not selected tween to 2 over 1200 ms. (compiled.vs section DrinkLineShader; ev_9242…1305 @ app.js byte 1499099, 1505687, 1506984)
- LINE-10 (corrected): LineDrawnShader is the generic line shader plus a two-tone split at uv.y 0.5 and alpha x 0.2. Its width follows perspective and is not constant on screen. (compiled.vs sections LineDrawnShader, Line)
- LINE-11 (corrected): uil.json tunes DrinkLineShader plus one value, windLinesSpeed 0.5. It has no keys for the WindLine, Sketch or LineDrawn shaders or for the curve files. The JS defaults for DrinkLine differ from uil (thickness 0.0075 and others), and which one wins at runtime is unverified. A separate GPU-particle tube system (1000 particles, 18 x 3 tubes, 256 curves) also exists. (uil.json; ev_9242…1305 @ app.js byte 1494834, 1509237)
- Per-file bindings: portal at threshold 0.84, speed 1, tile 2, uFrameRate 18 (an 18 fps stop-motion boil). Wander wind 0.78/1. Ground-horizontal 0.81/0.4. Ground-vertical 0.6 with drift. Dust shaders 0.78/1 with thickness 0.25/0.35. (ev_9242…1305 @ app.js byte 1363678, 2050469-2051520, 1649231)
- The sketch lines come from one procedural curve sampled at 256 points. Their speed multiplier tweens to 4 over 3 s while the user draws and back to 1 afterwards. The per-line random comes from Math.random, so it changes on every visit. (ev_9242…1305 @ app.js byte 1340950, 1344118, 1345906)

### Inferences

- Reusable technique: a constant-width ribbon built from prev/next attributes, a noise-threshold dash mask tied to scroll, low-fps quantized time for the boil, and a 0 to 1 to 2 draw-in/draw-out uniform. In Santioni the threshold follows world scroll and the draw-on follows UI tweens. Confidence high.
- Ribbons cost 2 vertices per point, which is cheap on phones. Confidence high.

### Unknowns

- Measured timings at runtime, and the final uniform values (uil against JS defaults).

### Decision for HayPaComer

Store our own curves as flat xyz arrays resampled at a fixed step of 0.1 to 0.25 units. At load time, expand them into ribbon BufferGeometry with prev/next attributes and a seeded per-line random (deterministic, unlike theirs). One ShaderMaterial does constant screen width, a noise dash mask whose threshold follows normalized band progress (not raw world scroll), time quantized to 8 to 18 fps, and a uAnimate 0 to 1 to 2 for explicit stroke reveals between market, fridge, scale and plate. On phones, use fewer curves and a smaller noise texture.

## 6. Captions and hold

### Observations

- CAPT-01: Captions are WebGL MSDF text (upper case) drawn with TextBoxTextShader. The text mesh is skipped in the scene pass and drawn after all post-processing, so post effects never touch it. (ev_9242…1305 @ app.js byte 1790520, 1791543)
- CAPT-02: The frame is a separate plane: white fill, #111111 border, and an edge wobbled by noise stepped at 8 fps. (compiled.vs section BorderBGShader)
- CAPT-03: The box starts at scale.x 0.25, x -0.5, alpha 0. The container (frame plus text) tweens to x 0 and scale 1 over 800 ms with cubic-bezier(0.45, 0.30, 0.16, 1.00). The background alpha reaches 1 over 333 ms, linear. (ev_9242…1305 @ app.js byte 1789831, 1793308)
- CAPT-04: Per line: translate over 2000 ms easeOutQuart (glyphs shift 0.4 behind a mask edge at x -0.1), and opacity over 1200 ms easeOutQuart with a 300 ms delay. The stagger is 200 ms for translate and 100 ms for opacity, or 150 ms for both when there are more than 4 lines. (ev_9242…1305 @ app.js byte 1793092, 1793505)
- CAPT-05: Each box plays a round-robin "textbox" one-shot when it appears. (ev_9242…1305 @ app.js byte 1793878)
- CAPT-06: Scroll triggers the box once, when scroll passes worldY - 0.5 x textHeight + 0.6 x screenHeightWorld and the parent scene is visible. The box then queues its VO. Scrolling out of a window around the box, or hiding the parent, removes it from the queue. A VO reset re-arms every box. (ev_9242…1305 @ app.js byte 1792570, 1792893)
- CAPT-07: The karaoke clock is AudioContext.currentTime minus the VO start time. Each glyph carries its word's start and end in a vertex attribute. (ev_9242…1305 @ app.js byte 1794074, 844787)
- CAPT-08: highlight = smoothstep(start - 0.4, start, t) x (1 - smoothstep(end, end + 0.4, t)) x influence, mixing toward red #c82924. With a median word of 0.26 s, each word stays lit about 1.06 s. (compiled.vs section TextBoxTextShader; ev_9242…1305 @ app.js byte 1790865)
- CAPT-09: Highlight influence fades to 0 over 600 ms when the VO stops and over 400 ms on mute (back to 1 on unmute). Text stays visible with sound off. (ev_9242…1305 @ app.js byte 1794466, 1793986)
- CAPT-10: The VO queue starts the next clip only when nothing is playing and scroll has been idle for 100 ms (|delta| <= 0.6). The next clip is the first queued id in key order, not first-in-first-out. On exit a clip fades with tau 0.15 and the next may start after 400 ms. VO baseGain is 1.25. (ev_9242…1305 @ app.js byte 2035074, 2035355, 2037655)
- CAPT-11: all_timestamps.json holds 18 entries (16 lines plus 2 flavours), each with words carrying start, end and type. Lines run 3.38 to 20.52 s and 1.47 to 2.82 words/s. The caption text is built by joining the words, so the screen always matches the timestamps. (raw/assets/data/vo/all_timestamps.json)
- CAPT-12 (corrected): createAnnouncer and srSpeak (aria-live helpers) each appear twice, both times inside their own definitions, with no call sites. Captions get no ARIA exposure. (ev_9242…1305 @ app.js byte 1290012, 1290407)
- CAPT-13 (corrected): Captured highlights showed 2 to 3 words red at once ("IN AN IMMENSE", "STANDS BEFORE"). The frame-before-text order comes from the code timing; it was not observed on a single box. (cdp-probe work/captions-hold/probe3.mjs@17)
- HOLD-01: "Hold" in Santioni is a continuous hold-to-act control in three scenes (draw, pour, move hand). Nothing completes or unlocks the story; the only DrawingComplete listener just resets a helper. (ev_9242…1305 @ app.js byte 1334880, 1471427, 1582569, 1353212)
- HOLD-02 (corrected): Desktop cursor: a 120 px WebGL disc that follows the pointer with lerp 0.1 and stretches with velocity. It scales in over 600 ms inside the scene (excluding the bottom 100 px) and out over 300 ms. On press it shrinks to 0.5 over 400 ms, the label fades and the ring thickens (0.06 to 0.08). On release it returns over 600 ms. Leaving the area ends the hold. (ev_9242…1305 @ app.js byte 1553903, 1557367; compiled.vs section GLUICursorBGShader)
- HOLD-03: Mobile disc button: 120 px with a 1.3x hit area, and a press counts only when |scroll delta| < 1. The press disables story scroll, shrinks the disc to 0.1, and slows the follow lerp to 0.08 over 1 s. Release restores it (lerp back to 1 over 5 s) and re-enables scroll after 300 ms. Leaving the area cancels the hold at once. (ev_9242…1305 @ app.js byte 1560113, 1561869, 1563886, 1564165, 1564638)
- HOLD-04: Pour progress tweens 0 to 1 over 3000 ms on press and back to 0 over 1000 ms on release, both linear. It scrubs an arm animation (frames 0 to 60), the water line and a shadow line. (ev_9242…1305 @ app.js byte 1472536, 1472647)
- HOLD-05 (corrected): Release has no penalty and keeps partial progress, but the durations are fixed (3000 and 1000 ms from the current value), so the rewind is 3x the fill speed only for a full-range run. (ev_9242…1305 @ app.js byte 1472536, 1472647)
- HOLD-06: The pour loop volume ramps over progress 0.24 to 0.28. Start and stop one-shots latch at 0.24. A drinking one-shot plays at animation keyframe 132. (ev_9242…1305 @ app.js byte 1482441)
- HOLD-07: There is no keyboard path for hold. Only mouse and pointer events are bound, and the generic helper maps Space to click only for role=button hits. (ev_9242…1305 @ app.js byte ~1559300, 1563886, 131295)
- HOLD-08: On desktop, drawing is always allowed. On mobile it is allowed only while the disc is held. (ev_9242…1305 @ app.js byte 1334948)

### Inferences

- The karaoke follows the audio clock, so it stays in sync when frames drop. Confidence high.
- Captions can be read silently; sound only adds the highlight. Confidence high.
- Screen readers get no caption text. Confidence medium (no live AX check of captions).
- Hold never gates the story. On mobile it only freezes scroll so a drag is not read as a scroll. Confidence high.
- Hold interactions cannot be reached by keyboard or screen reader. Confidence high.

### Unknowns

- Wall-clock time from box trigger to VO start and to the first highlighted word (the runtime probe was blocked).
- Per-scene caption placement values, and HandScene motion details.

### Decision for HayPaComer

Keep caption boxes as WebGL planes drawn after post effects, with our own easing in the same ranges (box about 800 ms, lines staggered 100 to 200 ms, frame boil at 8 fps). Mirror each caption in a visually hidden DOM element. Trigger boxes by scroll. Start narration only after about 100 ms of scroll idle, in story order. Drive word highlights from our own JSON word timestamps against AudioContext.currentTime, with ramps of 0.3 to 0.4 s, and keep the text when muted. Since Santioni has no timed gate, we design our own press-and-hold gate. It is a real button with an aria-label and a progress value (aria-valuenow). It fills linearly over about 3 s and rewinds at a fixed rate of about 3x on release, keeping partial progress. It plays a sound and a haptic cue at 20 to 25% and a completion sound at the end. Space/Enter keydown starts the fill and keyup releases it. Completion is announced in a live region. A single-activation alternative serves reduced motion and assistive technology. Scroll freezes on press only when scroll velocity is near zero and resumes about 300 ms after release.

## 7. Audio

### Observations

- AUDI-01: Audio is Web Audio only: one AudioContext, a master gain to the destination and a main gain to master. There is no HTMLAudio path. (ev_9242…1305 @ app.js byte 252265)
- AUDI-02 (corrected): Each sound is fetched and decoded with decodeAudioData, and every play creates a new AudioBufferSourceNode into a per-sound gain. Despite a nominal 500 ms fade-in, the start gain jumps to the target at once. Only stop and pause with a duration ramp to 0 linearly. (ev_9242…1305 @ app.js byte 259791, 262273)
- AUDI-03 (corrected): Format: Ogg when canPlayType('audio/ogg') is non-empty, otherwise MP3. VO is always MP3. Silence trimming (threshold 0.01, loop start +0.05 s) applies only to SFX and bed tracks loaded as MP3. (ev_9242…1305 @ app.js byte 1372046, 1355863)
- AUDI-04 (corrected): The 47 SFX and bed tracks load eagerly inside the loader, before the gate. The 18 VO MP3s are fetched only once the context is running, so in a real browser they load after the first gesture. Headless Chrome started the context as running. (ev_9242…1305 @ app.js byte 1355511, 1356114; REA ev_417a86d04df62f9ae3752135e6e5dc13222999efc34a713560f97e1f49d025f5)
- AUDI-05: Content-Length totals: 47 Ogg files = 6,889,053 B, 18 MP3 files = 2,018,144 B. In the warm run, Ogg loaded at 876-1026 ms and MP3 at 1119-1280 ms after navigation. (cdp-probe work/audio/sidecar1.json)
- AUDI-06: 47 tracks: 25 round-robin variants, 5 drones, 17 others. Plus 18 VO files. All stereo, at a 48 kHz context rate. (ev_9242…1305 @ app.js byte 1371635; cdp-probe work/audio/probe1.json)
- AUDI-07: The default is unmuted and the setting is never stored. Capture-phase gesture listeners resume a suspended context: gains go to 0, then rise toward 1 starting 0.4 s later with tau 0.5. (ev_9242…1305 @ app.js byte ~252100, 253026)
- AUDI-08: All 15 loops start together at gain 0 once the context runs; after that, scenes only automate gains. The gate's YES click tweens drone_1 to 1 over 1200 ms (measured about 1.27 s). (ev_9242…1305 @ app.js byte 1356114, 1764852; cdp-probe sidecar1.json@909)
- AUDI-09: The toggle is a 4-bar WebGL visualizer with a hidden link (role=button, aria-pressed). It becomes clickable 3000 ms plus 800 ms after the loader. Mute and unmute start after 50 and 200 ms, both with tau 0.25. Measured: 0.217 at about 430 ms after mute, 0.977 at about 1.15 s after unmute. (ev_9242…1305 @ app.js byte 1381501, 255703; cdp-probe work/audio/probe2.json@25, @232)
- AUDI-10: The visualizer uses an AnalyserNode with fftSize 256, 4 bands (20-600, 600-1000, 1000-2000, 1500-22050 Hz) and easeInSine bars. (ev_9242…1305 @ app.js byte 257109)
- AUDI-11: Drone crossfades are equal-power (cos/sin), driven by scene progress, with tau 0.02. Bounds per scene: approach 0..1, cathedral 0..1, antiGravity -0.5..0.5, taste -0.25..0.5. drone_5 settles at 0.1. (ev_9242…1305 @ app.js byte 1760038, 256046, 1374474; cdp-probe probe2.json@832)
- AUDI-12 (corrected): Wind and jazz gains are written directly every frame with no smoothing. Wind measured 0.064 at y 87, 1 by y 703, 0 by about 8011. Jazz reached 0.45 by about 26142. Footsteps ramp in over progress 0.5..0.65 and out over 0.85..1. (ev_9242…1305 @ app.js byte ~1760900, 1367099; cdp-probe probe2.json)
- AUDI-13 (corrected): Nothing ducks under VO. Portal contact lowers the drone chain to 1 - 0.66 x contact and sweeps a lowpass target of (200..12000 Hz by hand speed) x contact, always at tau 0.1 because of an argument bug. Menu jumps silence every chain except drone and vortexFocus until |delta| <= 10, then restore all of them with tau 0.1. (ev_9242…1305 @ app.js byte 1578600, 1603785)
- AUDI-14 (corrected): VO is ordered by text id and only one clip is audible at a time. A clip that fades out keeps running at gain 0 until its buffer ends. VO lengths run 3.43 to 20.78 s. (ev_9242…1305 @ app.js byte 2034700; cdp-probe probe1.json@2229, probe2.json)
- AUDI-15 (corrected): One-shots: ui_click (gate, cookie, menu, products, scroll-to-top), textbox, footsteps at walk keyframes (approach 16/32, cathedral 3/6, wander 13/34), bottle hover/interact, carousel, pour start/stop, drinking, antigravity release, and transition_crack when scene progress passes 0.1. (work/audio/callsites.txt)
- AUDI-16: Interaction loops are mapped continuously: speed_up and spawn follow the shake multiplier (tau 0.2), the 800 Hz lowpass wet mix follows the same value, a stereo panner follows cursor x (mouse only, no touch path), and the portal hum follows visibility. (ev_9242…1305 @ app.js byte 1344835, 1352790, 1586188)
- AUDI-17: The round-robin index advances as (v+1) % n + 1, so with n = 6 only variants 1, 3 and 5 ever play. This was observed at runtime. (ev_9242…1305 @ app.js byte 1384395; cdp-probe sidecar1.json)
- AUDI-18 (corrected): Background muting follows document visibilitychange. Desktop fades master with tau 0.05; mobile also suspends the context. (ev_9242…1305 @ app.js byte 252050, 30881)
- Decoded bed lengths: jazz 15.6 s, wind 5.95 s, pouring 5.67 s, portal 12.5 s, drones 36 to 79 s. One-shots stop the previous instance of the same id unless overlap is allowed (only textbox and bottle_hover allow it). Muting pauses nothing; it only zeroes the gains. (cdp-probe probe1.json; verifier notes)

### Inferences

- Sound design is gain automation over loops that are always running: there is no start latency, but 15 sources run from the start. Confidence high.
- MP3 trimming exists to fix encoder padding in loops. Confidence medium.
- About 8.9 MB of audio loads with no mobile budget. Confidence medium.

### Unknowns

- The real suspended-to-running resume path, real tab-hide behaviour, mobile suspend, and wire bytes per file.

### Decision for HayPaComer

Use one AudioContext with master and main gains, fetch plus decodeAudioData, and no HTMLAudio. The press-and-hold gate is the gesture that resumes the context, followed by a master ramp (0.4 s delay, tau 0.5). Start bed loops together at gain 0 and drive them from band progress with equal-power crossfades and tau of about 0.02. Smooth every gain (setTargetAtTime), including the per-frame ones. Give one-shots a real short fade-in. Fix the round-robin with (i % n) + 1 or a shuffled bag. Store the mute choice; mute with tau 0.25 and a toggle that sets aria-pressed. Duck beds under narration and stop faded VO sources. Choose Ogg/Opus or MP3/AAC with canPlayType. Load the first band's beds up front and lazy-load the rest. Load nothing until sound is enabled. Keep phone audio under about 1.5 MB, and suspend the context on visibilitychange.

## 8. Performance and mobile

### Observations

- PERF-01: Canvas pixel ratio: tier 4 and above on desktop render at max(1.5, dpr), so a DPR 1 screen is supersampled. Most other tiers use max(1.25, dpr). Caps: OVERSIZED 1, T0 min(1.3), T1 min(1.8), mobile tier 2 or lower min(2). Mobile tier 3 and above is uncapped. Measured: desktop canvas 2160x1350 for 1440x900 at DPR 1, mobile canvas 1170x2532 for 390x844 at DPR 3. (ev_9242…1305 @ app.js byte 1044474; cdp-probe desktop2.json, mobile2.json)
- PERF-02: Post and render-target ratio per tier. Desktop T0..T5: 1 / 1.2 / 1.35 / 1.5 / 2 / 2. Mobile MT0..MT5: 1 / 1.2 / 1.35 / 1.4 / 1.6 / 1.8. Each is min(dpr, value), and OVERSIZED gives 0.8. Measured: 1 on desktop, 1.8 on mobile. (ev_9242…1305 @ app.js byte 1786390, 1302130, 744985)
- PERF-03 (corrected): Tiers come from the unmasked renderer string. Mobile tiers by chip: MT0 = A7-A9, Adreno below 415, Mali-T below 628. MT1 = A10, Adreno 530-615, Mali-G73 and up, any other Android. MT2 = A11-A12, Adreno 630 and up, Mali-G74 and up. MT3 = A13-A14, Adreno 640, G76. MT4 = A15-A16, Adreno 650, G78/G710. MT5 = A17 to A25 or Apple M, Adreno 740. There is a blocklist and a ?gpu override, and iOS drops one tier when refresh is below 40 Hz. The navigator.platform fallback is dead code. (ev_9242…1305 @ app.js byte 913400-918200)
- PERF-04: OVERSIZED means tier 0 or lower with the largest side over 1400 px, or tier 1 or lower with DPR below 2 and the largest side over 1600 px. (ev_9242…1305 @ app.js byte 918236)
- PERF-05 (corrected): MSAA is 4 samples at tier 4 and up, 2 at tier 3, and off at tier 2 or lower, on WebGL1, in compat mode or in VR. FXAA runs only when MSAA is off and the tier is high enough. Fluid runs at tier 2 and up. DOM tilt is off on mobile. There is no fps cap. A GPU that matches no table gets full quality by default. (ev_9242…1305 @ app.js byte 1786390)
- PERF-06 (corrected): The loop has no global fps cap and no render-on-demand. Every 3 s it takes the median of 31 fps samples and snaps it to [30, 60, 72, 90, 100, 120, 144, 240]. After the first result the rate can only rise until the window changes screens (checked every 5 s). Frame delta is clamped at 200 ms. Individual callbacks can be throttled, for example 24 fps state polling. (ev_9242…1305 @ app.js byte 20800-23700, 74157)
- PERF-07: Draw calls continue while idle: 27 per frame idle; while scrolling p50 38, p95 71, max 83; 74 to 80 parked mid-story. (cdp-probe work/perf-mobile/perf.mjs@26, desktop2.json)
- PERF-11: The cold preload before the gate is 209 requests and 12.33 MB. Ogg 6.91 MB, PNG 2.02, geometry .bin 1.77, JS 0.47 (including 181 KB of third-party gtag), JPG 0.27, WASM 0.25, JSON 0.17, WOFF2 0.12, KTX2 0.09, plus extensionless requests (0.19 MB) and SVGs. After the gate: 19 requests, 2.03 MB (VO). During scroll: 0 requests. (cdp-probe perf.mjs@14-21, desktop2.json, mobile2.json; REA ev_9e515fe602a49878a03044a2fb1b52cd034d0190b214090b432f156053da5a79)
- PERF-12 (corrected): 8 hydra-thread workers fetch assets during load. That they also decode is inferred. The gate was ready 7.3 to 8.4 s after navigation in headless runs. (desktop2.json, mobile2.json)
- PERF-13 (corrected): Ping-pong targets use HALF_FLOAT on mobile and on every WebGL2 renderer. The performance analyzer runs only in local builds. A hidden tab only suspends audio. The JS heap was 56 to 76 MB after scrolling. (ev_9242…1305 @ app.js byte 1295250, 746656)
- PERF-14: Phones download the same 209 assets as desktop. The only mobile asset is a header logo. (mobile2.json vs desktop2.json; ev_9242…1305 @ app.js byte 1592781)
- PERF-15: Device.mobile requires iOS, Android or MagicLeap plus touch. A tablet is a device whose largest side is over 1000 px. (ev_9242…1305 @ app.js byte 36950)
- PERF-16: Mobile story scroll uses touch drag, 0.9 inertia and a "prevent resize reload" flag. The carousel uses a 1.5x drag with touch-action pan-x. (ev_9242…1305 @ app.js byte 1755264, 1354672, 1666450)
- PERF-17: 15 s of swipes reached y 6300 of 24039. The mobile story is shorter than desktop's 29924. (cdp-probe perf.mjs@70-75, mobile2.json)
- PERF-18: A rotate prompt shows on mobile in landscape (tablets too). (ev_9242…1305 @ app.js byte 1721300)
- PERF-19: Mobile drops the GL cursor, tilt, fluid on several layers and the hand inverse FX pass. Hover motion becomes idle sine motion, and hold moves to an on-screen disc. (ev_9242…1305 @ app.js byte 1558467, 1397046, 1584050, 1681581, 1575103)
- PERF-20: A tier-5 phone gets full quality: MSAA 4, fluid, post at 1.8, canvas at DPR 3, about 28 idle draws and p50 41 while scrolling. (mobile2.json)

### Inferences

- Strategy: load everything, render every frame, and pick quality by GPU tier, with canvas resolution and post resolution as separate knobs. Confidence high.
- On real phones the GPU fill rate at DPR 3 with MSAA 4 is the likely bottleneck, not JS. Confidence medium (the emulation used a desktop GPU).
- Audio is more than half of the preload; making sound opt-in would roughly halve it. Confidence high.

### Unknowns

- Real phone GPU, thermal and memory behaviour; per-frame shader time; weak-GPU paths; iOS Low Power Mode effects; tablet branches; real-finger touch calibration.
- Whether the iOS low-refresh penalty reads a measured rate or the default 60.

### Decision for HayPaComer

Use two DPR knobs. The canvas gets min(dpr, 2) on phones (1.5 on weak tiers) and never supersamples on desktop. Post runs at 1.0 to 1.6. Pick the tier with a short startup frame-time probe, not renderer-string tables, and fall back to a safe middle tier when unsure. Keep the 200 ms delta clamp and refresh-normalized lerps, and let the refresh estimate go down as well as up. Add render-on-demand: stop the loop when scroll velocity, tweens and boils are idle, and drop the boil to a few fps. Ship one story and one asset set, scaled by tier. Preload only the first band behind the gate, decode KTX2 and geometry in a worker, and stream later bands. On phones, drop pointer-only effects (tilt, cursor trail, fluid). Budget: 6 ms of main-thread time per frame at p95 under a 4x CPU throttle.

## 9. Accessibility

### Observations

- AY-01: The bundle never queries prefers-reduced-motion. Its 5 matchMedia calls test hover, standalone display, color gamut and orientation. (ev_9242…1305 @ app.js byte 33332, 37615, 941843, 1721287)
- AY-02 (corrected): With reduced motion emulated, matchMedia reports reduce, and nothing changes. The gate fade (about 0.8 to 1.0 s), its 6 animations, and the keyboard sweeps (End about 28,900 units, Home about 29,900, both within 1 unit in about 500 to 660 ms) are the same in both modes. (cdp-probe work/a11y/probe-reduced-1440.json vs probe-normal-1440.json; REA ev_9b7fadcb9c58097d08eaac9d5013066c7103c7c8681033cc45fd30a9ccd6fd61, ev_25300eb650427ced1c8c969c05d036efeef35b9348f80c76530ed9eaae03627f)
- AY-03: The age gate cannot be operated by keyboard. YES and NO are DIVs with tabIndex -1 and no role or label. After 8 Tab presses focus is still on BODY, and the AX tree shows them only as text. (cdp-probe probe-normal-1440.json; REA ev_581b78e3752981120605e690e88f500bfa909898bcdd1eb0ea83a544e715001f)
- AY-04: After the gate, focus stays on BODY. No aria-live region exists at any point. (cdp-probe probe-normal-1440.json)
- AY-05: Story keyboard scroll works once the scroll is enabled. Arrow keys move 150 units, PageDown and Space one stage height (900), Home and End go to the ends, each with a 400 ms easeOutCubic tween. (ev_9242…1305 @ app.js byte 1099250, 1755324; cdp-probe keyScroll)
- AY-06 (corrected): Tab reaches 10 distinct controls plus one BODY stop per cycle; the DOM holds 12 focusables. Footer and flavour controls sit at 29,292 to 30,748 px. Focusing them never scrolls the story, and 6 of 6 tested stops were off screen. (cdp-probe probe-reduced-1440.json focusVsScroll)
- AY-07: Focus shows only the browser default outline (auto 1px). The only custom focus styles are in debug-panel CSS. (cdp-probe storyTabs; ev_9242…1305 @ app.js byte 1195292)
- AY-08 (corrected): Screen readers get a static mirror: 16 sr-only spans created from per-element addSrOnly layout flags, the same at every scroll position. 4 of them hold rotate-notice copy, which the AX captures do not show as exposed. The GLSEO layer holds only the Toggle audio link. The canvas has no name, role or fallback. (cdp-probe probe-reduced-1440.json; ev_9242…1305 @ app.js byte 1782272, 1163752)
- AY-09: The VO has no text alternative in the DOM (0 of 18 lines found). The accessibility toolkit (announcer, srSpeak, focus traps) is dead code; its class name appears only at its definition. (ev_9242…1305 @ app.js byte 1287703; work/a11y/rea-*-story.txt)
- AY-10: axe-core 4.10.2 finds 2 violations at both the gate and the story: meta-viewport (critical: user-scalable=no, maximum-scale=1.0) and region (5 and 12 nodes). Color contrast is incomplete because the text sits over WebGL. (cdp-probe gateAxe, storyAxe)
- AY-11: lang en, NAV and HEADER landmarks, no main, one heading exposed in the story. The AX tree has 34 nodes at the gate and 169 in the story. (REA ev_7cd559b4c758b2251104ba0422bcad9f2ed438fade50849b0832e8607e91fcf8)
- AY-12: Hydra buttons use a transparent hit element with role=button, an aria-label from its text, and focus mapped to hover. The custom key handler covers Space only; Enter works only because the hits are anchors. (ev_9242…1305 @ app.js byte 130850, 834550)

### Inferences

- No reduced-motion support. Confidence high.
- A keyboard-only or switch user cannot enter the experience at all. Confidence high.
- Screen-reader users get static marketing lines, without the narration and without any link to what is on screen. Confidence high.
- The framework's accessibility toolkit was available but unused; accessibility was an afterthought. Confidence medium.

### Unknowns

- Real VoiceOver and NVDA output, phone touch accessibility, full contrast results, and how the gate blocks Tab without inert or a trap.

### Decision for HayPaComer

Build accessibility into the motion system from the start: see section 12.

## 10. Parameters for Motion v2

| Name | Santioni value | Evidence | HayPaComer will use |
|---|---|---|---|
| Wheel scale (pixel mode) | 0.33 Mac Chrome/Safari, 0.25 Win Chrome, 1 raw elsewhere | ev_9242…1305 @ app.js byte ~1096600-1097700 | 0.3 for every pixel-mode device |
| Wheel scale (line mode) | x4 Mac Firefox, x10 Win Firefox | app.js byte 1096764, 1097191 | 16 to 40 px per line |
| Travel per wheel notch | 330 px for deltaY 100 (10 x factor x delta) | cdp-probe s4-measure.mjs@17 | about 3x deltaY |
| Inertia decay | 0.9 per frame, not dt-scaled | app.js byte 1094782 | 0.9^(dt/16.67) |
| Approach smoothing | 0.5 per frame, not dt-scaled | app.js byte 1094782 | merged into one time-based stage |
| Story lerp | 0.08 desktop / 0.9 mobile per 60 Hz frame, normalized | app.js byte 1753619, ~3260 | tau about 140 ms desktop, about 60 ms phones |
| Settle after one notch | scroll 99% at 1464 ms, story 99% at 1764 ms (30 Hz) | cdp-probe s4-measure.mjs@17 | 99% within about 1.2 s |
| Momentum block | hardBlock 200 ms, needs a larger impulse | app.js byte 1102496, 1097805 | 200 ms, counter reset per jump |
| Touch release | last delta x 25 (35 Android), 2500 ms easeOutQuint | app.js byte ~1099000 | velocity averaged over 50-100 ms, 1.5-2.5 s glide |
| Keyboard | Arrow 150 px, Page/Space one viewport, 400 ms easeOutCubic | app.js byte ~1099300 | the same; instant under reduced motion |
| Camera | fov 35, z 5, parallax 0.5 (0 on phones), lerp 0.05 | app.js byte 1765994, 324203 | fov 35, z 5, parallax 0.5 desktop only, lerp 0.05 |
| Scroll pitch | 0.002 rad/px, lerp 0.05, clamp 0.2 rad | app.js byte 1759031 | the same |
| Scene visibility padding | 0.1 world units | app.js byte 1759153 | 0.1 |
| Resize debounce | 200 ms | app.js byte 1764499 | 200 ms |
| Mobile height interpolation | Stage.width 1600 to 393, clamped | app.js byte ~1756500 | 1440 to 390, clamped |
| Boil fps (hatching) | 8 fps steps, 0.3 UV/s drift | compiled.vs StaticObjectBaseShader.glsl | 8 fps |
| Boil fps (portal lines) | 18 fps | app.js byte 1363678 | 8 to 12 fps |
| Hatching thresholds | (0.4, 0.7) default, line weights 0.45/0.1, mask 0.2 | app.js byte 1744570; compiled.vs | the same defaults, per-object JSON |
| Ink cover | 2 screens (1.5 mobile), threshold 0.4, edge 0.1 UV/s, next scene overlap -0.5 | app.js byte 1755915; compiled.vs TransitionLineShader.glsl | 2 / 1.5 screens, 0.4, 0.1 UV/s, half-screen overlap |
| Panel iris | 800 ms easeOutCubic on in-view | app.js byte 1366688 | 800 ms ease-out |
| Hold duration | none (no timed gate); pour fills 3000 ms, rewinds 1000 ms | app.js byte 1472536, 1472647 | 3 s fill, fixed-rate 3x rewind, cue at 20-25% |
| Caption box reveal | 800 ms cubic-bezier(0.45,0.30,0.16,1), bg 333 ms | app.js byte 1789831, 1793308 | about 800 ms, own easing |
| Caption line speed | translate 2000 ms, opacity 1200 ms (+300 ms), stagger 200/100 ms (150 if >4 lines) | app.js byte 1793092, 1793505 | 1200-2000 ms, 100-200 ms stagger |
| Karaoke ramp | 0.4 s on each side of a word | compiled.vs TextBoxTextShader | 0.3-0.4 s |
| VO idle gate | 100 ms with abs(delta) <= 0.6 | app.js byte 2035355 | 100 ms |
| Mute | tau 0.25, 50/200 ms delay | app.js byte 255703 | tau 0.25, stored |
| Drone crossfade | equal power, tau 0.02 | app.js byte 256046 | equal power, tau 0.02 |
| Canvas DPR cap | desktop max(1.5,dpr) at tier 4+; phones uncapped at MT3+ (DPR 3 measured) | app.js byte 1044474; mobile2.json | min(dpr, 2) phones, min(dpr, 2) desktop, no supersampling |
| Post DPR cap | desktop 1-2, mobile 1-1.8 by tier, 0.8 OVERSIZED | app.js byte 1786390 | 1.0-1.6 by measured tier |
| MSAA | 4 at tier 4+, 2 at tier 3, off at tier 2 or lower | app.js byte 1786390 | 4 / 2 / off by measured tier |
| Frame delta clamp | 200 ms | app.js byte 20800-23700 | 200 ms |

## 11. Their frame times

Headless Chrome, Apple M3 through ANGLE Metal. The rAF interval mostly reflects the headless vsync clock. The rAF cost and the uncapped run are the real budget numbers.

| Metric | Desktop 1440x900, DPR 1, 60 Hz | Phone emulation 390x844, DPR 3, CPU 4x |
|---|---|---|
| rAF interval p50 / p95 / max | 16.7 / 16.7 / 16.8 ms | 16.7 / 16.7 / 16.8 ms |
| Trace frame-start interval p50 / p95 / max | n/a | 16.61 / 18.94 / 34.61 ms |
| Main-thread rAF cost p50 / p95 / max | 1.91 / 3.05 / 10.27 ms | 4.58 / 6.63 / 24.39 ms |
| Task duration p95 / max | 2.49 / 10.55 ms | 6.11 / 19.65 ms |
| Long tasks over 50 ms | 0 | 0 |
| GPU-process task p50 / p95 (proxy) | 0.27 / 0.53 ms | n/a |
| Uncapped frame interval p50 / p95 / p99 / max | 4.2 / 8.6 / 13 / 29 ms (about 238 fps) | n/a |
| Draw calls idle / scroll p50 / p95 / max | 27 / 38 / 71 / 83 | 28 / 41 / 50 / 53 |
| Earlier 30 Hz run, rAF cost p50 / p95 | n/a | 5.08 / 6.84 ms |
| Canvas backbuffer | 2160x1350 (1.5x supersampled) | 1170x2532 |
| JS heap after scroll | 56-76 MB | 56-76 MB |

Evidence: cdp-probe work/perf-mobile/perf.mjs@55 (rAF sampler), @87 (Tracing.start), @86 (CPU throttle), @26 (draw counter); work/perf-mobile/trace.py over desktop2.trace.json and mobile2.trace.json; desktop-uncapped.json. The measured rAF cost includes the probe's own small sampler callback.

## 12. Gaps HayPaComer will beat

Accessibility:

1. Reduced motion: read prefers-reduced-motion and also offer an in-page toggle. A single motion budget governs keyframed cameras, the gate, the ink covers and the boils. Under reduce, camera flights become cuts or crossfades of 200 ms or less, parallax and wobble turn off, boils freeze, and keyboard scroll is instant. Santioni ignores the setting (AY-01, AY-02).
2. Keyboard and switch access to the gate: a real button with an aria-label and an exposed progress value. Space/Enter hold works, and a single activation is offered. Santioni's gate cannot be reached by keyboard (AY-03, HOLD-07).
3. Focus and announcements: after the gate, move focus to the story region and announce it in a polite live region (AY-04).
4. A screen-reader story that follows the scroll: one chapter at a time (market, fridge, scale, plate) as DOM headings and text, with aria-hidden following the virtual scroll. Focusing an off-screen control scrolls the story to it (AY-06, AY-08).
5. Captions in the DOM for every narration line and sound cue, with a mute control reachable first and stored across visits (AY-09, AUDI-07).
6. Zoom is never blocked. Add a main landmark, mark the canvas aria-hidden, and give focus a visible style of 2 px or more (AY-07, AY-10, AY-11).
7. Keyboard scroll ignores keys while a form field has focus (SCRO-13).
8. CI runs axe on the gate and every chapter in both motion modes, with zero critical findings allowed.

Robustness:

1. Time-based smoothing and inertia, so 120/144 Hz screens feel the same as 60 Hz (SCRO-08).
2. Scroll position kept as a fraction across resizes (SCRO-14).
3. The momentum-block counter resets per jump, and "is scrolling" is derived from delta (SCRO-06, SCRO-10).
4. Phone canvas DPR capped at 2; no desktop supersampling; a safe middle tier for unknown GPUs (PERF-01, PERF-05).
5. Render-on-demand when idle (Santioni issues 27 to 80 draws per frame while idle, PERF-07).
6. Streaming per band and opt-in audio instead of a 12.3 MB eager preload (PERF-11).
7. Far bands disposed on phones (SCEN-06).
8. Audio fixes: a working round-robin, smoothed gains on every per-frame write, real fade-ins, narration ducking, faded VO sources stopped, a touch path for panning, and context suspension when hidden (AUDI-02, AUDI-12 to AUDI-17).
9. Seeded randomness so line styling is the same on every visit (lines group).

## 13. Limitations of this analysis

- The REA static semantic graph targets Node and Electron facts and says little about animation, shaders or audio. Most static citations are therefore byte offsets in the parent Evidence `ev_9242…1305`, found by plain string search; some offsets are approximate (~) but fall inside the cited function.
- REA scenario actions only support click, fill, goto, press, select and waits; they cannot do wheel, touch or press-and-hold. Our own CDP scripts drove that input and measured it, and REA observed passively where it was allowed. Some groups were blocked by the session permission classifier and produced no new runtime Evidence IDs: scroll runtime REA calls, scenes and camera runtime, lines runtime, the hold runtime and the caption timing.
- No real device was used. Every runtime number comes from headless Chrome on an Apple M3. Phone results come from emulation (iPhone user agent, DPR 3, touch, 4x CPU throttle) on a desktop GPU. Headless vsync varied between 30 and 60 Hz across launches, the AudioContext started as running without a gesture, and CDP cannot send deltaMode 1 wheel events.
- Not measured at runtime: touch drag and release curves, behaviour at high refresh rates, draw calls per scene change, camera response timing, caption-to-VO latency, the hold interaction, real screen-reader output, iOS Safari address-bar and Low Power Mode effects.
- The verification pass re-derived facts from the same saved probe outputs and the bundle. It did not run independent new runtime captures.
- The scroll and perf groups disagree on whether the mobile resize keeps the scroll fraction. We follow the scroll verifier: for the story scroller, that code path returns early.
