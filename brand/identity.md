# Visual Identity Brief, v3 "Pa' comer"

Version 3 replaces "Copper Counter" (v1 and v2). The name, the H-and-tray mark, the voice, and the principle "numbers first" stay. What changes is the scale and the colour: the identity moves from a quiet kitchen tool to a poster you hang in the kitchen.

## 01 Identity strategy statement

HayPaComer answers the question every Colombian household asks in front of the fridge: "¿qué hay pa' comer?". Version 3 makes that question the brand. The identity speaks at poster scale, in condensed capitals, over flat fields of colour taken from the food of Nariño: papa amarilla, tomate de árbol, cilantro and mora. Inside the app the posters become chapters: every tab owns one of those colours, and the data on it (grams, days, degrees) is the largest thing on the screen. The references are Nike for colour and type energy and Apple for restraint and motion; the result must still look like a kitchen in Pasto, not a sneaker shop.

## 02 Logo direction

**Primary mark (kept):** the H whose crossbar is a weighing tray, with the small round weight above it. It is a scale and a letter at once.

**Wordmark (new):** `HAYPACOMER` set in Fraunces 900 (soft, wonky cut), uppercase, one word. The apostrophe version "HAY PA' COMER" is reserved for the question headline, never for the logo.

**On colour fields:**

| Field | Mark and wordmark | Tray and weight |
| --- | --- | --- |
| Arroz (white) | Tinta | Copper |
| Papa amarilla | Tinta | Tinta |
| Tomate de árbol | Arroz | Arroz |
| Cilantro | Arroz | Arroz |
| Mora | Arroz | Papa amarilla |

Copper appears in the mark only on arroz. The mark is never outlined, never placed on a photograph without a solid field behind it, and never recoloured outside this table.

**Wordmark as loader:** the wordmark is the loader, at full viewport width, as on the /zeroz brand site. The letters fill from the bottom like a scale reading rising; when loading ends, the destination chapter's colour opens from the bottom edge (shutter) and the app appears. The fill is real progress (locale texts, session, first data), not a timer.

**Avoid:** fork-and-knife, fridge outlines, leaves, circuit boards, gradients, glows, sparkles, script type.

**References:** Nike (condensed capitals and the brand at poster scale on flat colour); the 60fps "FS 60P" watch site (the product name huge behind the object, the object exploding into its parts on scroll); /zeroz by Otsuka (the wordmark is the first and only thing on the loading screen).

## 03 Colour palette

Every field colour comes with its assigned text colour. Contrast is AA for body text on every pair.

| Role | Name | Hex | Text on it |
| --- | --- | --- | --- |
| Ground | Arroz | `#fbfbf8` | Tinta |
| Surface | Blanco | `#ffffff` | Tinta |
| Ink | Tinta | `#121212` | Arroz |
| Chapter | Papa amarilla | `#f3c443` | Tinta |
| Chapter | Tomate de árbol | `#c8432a` | Arroz |
| Chapter | Cilantro | `#2f7a4a` | Arroz |
| Chapter | Mora | `#4b1f2e` | Arroz |
| Action | Cobre | `#a8481f` | Blanco |
| Soft ink | Piedra | `#55534e` | Secondary text on arroz and blanco only |
| Step | Mesón | `#efeee9` | Tinta; a tone step between surfaces, never a border |

**States** keep their meaning and are told apart by words and shape, not only colour: expired and door open use tomate; at risk uses papa amarilla with tinta text; fresh, enough and done use cilantro; under review uses mora.

**Rules:** one chapter colour per screen; neighbouring chapters differ in lightness as well as hue; no tints, no gradients, no shadows at rest; copper only on the primary action of a screen.

**Palette type:** four flat produce fields on a white rice ground, anchored by tinta, with copper as the single action colour.

## 04 Typography

**Display: Fraunces.** A heavy serif at its largest optical size with the soft, slightly wonky cut, so headlines read like an engraved poster or a market label. Replaces Anton (2026-10-18) to match the engraved 3D fridge. Uppercase for screen titles, leading 0.92, at most two lines.

**Text and data: Hanken Grotesk.** 400 for reading, 600 for emphasis, `font-variant-numeric: tabular-nums` for every gram, date, degree and price. IBM Plex Mono is retired: numbers are now the text face at large size, not a monospace label.

**Hierarchy (five steps):**

| Step | Use | Size |
| --- | --- | --- |
| Poster | Landing headlines, loader | clamp(64px, 12vw, 200px), Fraunces 900 |
| Head | Screen and chapter titles | clamp(40px, 6vw, 88px), Fraunces 900 |
| Figure | The number of a tile (842 g, 4,2 °C) | clamp(40px, 5vw, 72px), Hanken 700 |
| Body | Reading, controls | 17px to 19px, Hanken 400 |
| Small | Secondary notes | 14px, Hanken 500 |

**Avoid:** monospace labels, uppercase letter-spaced captions, italics, a third family.

## 05 Imagery and photography

**Aesthetic:** real food and real kitchens, documentary, shot flat or from above, on plain grounds. The landing uses openly licensed photographs of Nariño produce and markets (credited in the footer) until the project has its own photographs of the fridge and the scale.

**Product:** the hardware (door sensor, temperature probe, ESP32, scale) is drawn in code as flat shapes in the palette, never faked as a photograph.

**Avoid:** stock lifestyle photos of people smiling at fridges, renders with glows, AI-generated food.

## 06 Iconography and illustration

**Engraving.** The fridge, its foods and every scene are drawn like an engraving: flat colour fills crossed by ink hatching where the light falls away, on cream paper, with the food as the only saturated colour. Scenes the 3D cannot show (markets, kitchens, people) come from public-domain engravings and plates, treated with the same ink, and are always credited.

Icons: filled, flat, 2px corners, drawn on a 24px grid in tinta or arroz. Icons appear only next to a word. Foods in the app are shown as solid circles and pills in their real colour (a tomato is tomate, an egg is arroz with a papa yolk), the same language as the drawn hardware.

## 07 Design principles

- **Numbers first.** On every screen the largest element is a measurement. Copy explains the number; it never competes with it.
- **One chapter per screen.** Each tab owns one produce colour; the colour tells you where you are before you read.
- **Motion explains change.** Things move when something changed or loaded: a number rises like a scale, a shelf slides when food leaves, a chapter opens like a shutter. Nothing moves to decorate, and everything respects reduced motion.
- **Honest states.** Unknown, estimated and stale data say so in words. Skeletons show the shape of what is coming, never fake content.

## 08 Brand expressions

- **App tabs and chapter colours:** Ahora is papa amarilla, Nevera is cilantro, Mercado is tomate de árbol, Números is mora, Chef is cobre, Ajustes and the inbox are arroz. The colour fills the screen header band and the active tab; data tiles stay on blanco for legibility.
- **Landing:** colour chapters at full bleed, the question headline at poster scale, the kit exploding into its parts on scroll, one live calculator for the food a household saves in a month.
- **Loader:** the wordmark filling like a scale reading, then the chapter shutter.
- **Social:** a single produce field per post with one number in Fraunces ("842 G").
- **Presentations:** the same chapters, one idea per slide, numbers in Fraunces.
