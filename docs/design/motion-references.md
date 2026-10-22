# Motion references from 21st.dev

21st.dev was used as a catalogue of interaction patterns for Motion v2. Its components are React; HayPaComer is vanilla JavaScript, so nothing is installed. The rule is to re-implement the technique in our own code. Code may only be copied when the component's licence allows it, with credit; neither retrieved component declared a licence, so none was copied.

The account is on the free tier (two code retrievals per day). Both retrievals were used on 2026-10-20; the other patterns are re-implemented from their public description and demo video.

| Pattern | Pick | How we got it | Licence | Where HayPaComer uses it | What we take |
|---|---|---|---|---|---|
| Hold to confirm | "Hold to confirm" by motiondotdev (24668) | Code retrieved | Not declared | App press and hold; story door gate | One progress value drives the fill, the ring and the button scale. We add Space and Enter, a single completion promise and a stamp |
| Ink blot reveal | "Ink Blot Reveal Image Carousel" by kedhareswer (36224) | Code retrieved | Not declared | How it works scene change | An SVG mask of a main circle plus seeded satellite drops, roughened by turbulence and displacement, growing to reach the farthest corner |
| Scroll word reveal | "Scroll word reveal" by motiondotdev (24525) | Description and video | Not checked | Story captions | Words revealed by scroll progress, with the full sentence kept in the DOM |
| Masked text reveal | "Text Reveal (Mask)" by soralabs (19257) | Description and video | Not checked | Landing hero lines | Lines slide up out of a mask when they come into view |
| Digits that roll | "Animate Digits" by unlumen (20071), "Number Ticker" by danielpetho (20459) | Description and video | Not checked | Calculator money, KPIs | Only the digits that change slide, with tabular figures so nothing shifts |
| Ink dissolve | "Ink Reveal" by ryanonline1234 (39360) | Description and video | Not checked | Story ink cover (ideas only) | Ink that develops like it is soaking into paper |
| Page transition | "Page Transition" by su2491251 (29262) | Skipped | — | — | Our ink mask between routes already covers it |

## Notes from the retrieved code

### Hold to confirm (24668)
- Pointer down sets the progress to 0 and animates it to 1 over 2 s with ease-out; pointer up or leave animates it back to 0 over 0.3 s.
- The same progress maps to the fill position, the button scale (1 to 0.85), the ring stroke width, colour, opacity and path length.
- The example has no completion callback and no keyboard support.

### Ink blot (36224)
- Seven satellite drops are placed with a seeded random generator from the tap point, 40 to 180 px away and 0.25 to 0.65 of the main size, so the same tap always inks the same way.
- Filter: fractal noise at a base frequency of 0.012 with 3 octaves, a displacement of about 90 px, then a 1.5 px blur.
- The radius grows to 1.3 times the distance to the farthest corner with a cubic ease-in-out over 2.4 s; reduced motion shortens it to 300 ms.
- It pauses when off screen or when the tab is hidden, and announces the new slide politely.

## Ink Reveal (2026-10-22)

| Component | Author | License | Used for |
|---|---|---|---|
| [Ink Reveal](https://21st.dev/ryanonline1234/ink-reveal) (21st id 39360) | ryanonline1234, [ease-atlas](https://github.com/ryanonline1234/ease-atlas) | MIT | `app/story/ink-reveal.js`: the fragment shader is theirs (fbm front, fibres that wick ahead of it, pooled rim, paper grain); the plain-module port, one shared context per page created on first use, and the landing wiring (fridge vignette, kit medallions, How it works swaps) are ours |
