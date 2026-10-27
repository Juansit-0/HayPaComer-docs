# App audit and direction contract (plan v5, stage A0)

Critique of the signed-in app on 2026-10-27, before the redesign. Two independent assessments: a design review (Nielsen heuristics, cognitive load, personas) and mechanical evidence (the Impeccable detector in the code and in the page, tap target and text size measurements at 1440 and 390, axe 0 on six routes). Screens: Ahora, Nevera, Mercado, Números, Chef, Ajustes, Bandeja.

## Score

| # | Heuristic | Score | Main issue |
|---|---|---|---|
| 1 | System status | 3 | Live grams, door and temperature are clear; "Conectando" never resolves into something a person can act on; a failed alerts request shows as "no alerts" |
| 2 | Real world | 3 | Local voice ("Botar", "Lo abrí", "≈ estimada"); zone names fall back to English; "Salva Tomate" reads as a command |
| 3 | Control and freedom | 2 | Undo only after a drag on the 3D stage; "Usar", "Botar" and "Marcar como usado" have none |
| 4 | Consistency | 2 | Two design systems layered: tokens and DESIGN.md describe v2, the paper look is an override with raw hex; three chip styles |
| 5 | Error prevention | 3 | Hold to confirm on discard and AI approval; Ahora pre-fills the full weight, so one tap uses everything |
| 6 | Recognition | 3 | Chef is off screen in the phone tabs; the hold is learned only by failing |
| 7 | Efficiency | 2 | No quick portions, no filter in the fridge list, servings hidden below the dishes |
| 8 | Minimalist design | 2 | Card inside card, equal weight everywhere, Números shows eaten/rescued/wasted three times |
| 9 | Error recovery | 2 | List fetch errors become empty states |
| 10 | Help | 2 | Nothing says the fridge is interactive or that discarding needs a hold |
| | **Total** | **24/40** | Acceptable |

Cognitive load: 4 of 8 checks fail (single focus, chunking, hierarchy, one thing at a time). Decision points over four options: global navigation (8 targets), shelf selector (5), stage palette (6+), add-food chips (15).

## Priority issues

1. **[P1] Phone shell and navigation.**
   - The stage takes about 450 of 844 px on every route.
   - The tabs sit at the top and overflow with no hint, so "Números" is cut and "Chef" is hidden.
   - On desktop "Salir" is clipped past the viewport, and the header is 13 px text.
   - Fix: a bottom tab bar on phones; a top bar with bell and account menu; the stage folded on routes that don't use it.
2. **[P1] Two design systems.** `tokens.css`, `DESIGN.md` and `theme.md` describe v2 (white rice ground, no shadows, Anton), while the app paints paper and ink through an override in `app.css` (around line 1555) with raw hex. As a result:
   - dialogs, chips and the Chef composer fall back to flat grey and white;
   - the composer covers the prompt chips on phones.

   Fix: the surface tokens (sheet, card, line, edge) move into `tokens.css`, the override goes, and DESIGN.md becomes v3.
3. **[P1] Undo and portions on inventory actions.** Use, discard and "mark as used" have no undo even though `POST /inventory/undo` and `undoable()` exist. Ahora pre-fills 100%.
   - Fix: the same undo toast everywhere.
   - Fix: "La mitad / Todo" portions.
   - Fix: discarding shows the money lost.
4. **[P1/P2] Header and inbox.**
   - The alerts link goes empty at 0 unread, so history becomes unreachable and focus lands on an invisible target.
   - Fetch errors read as "no alerts".
   - Alerts carry no action.
   - Fix: a bell that is always there with a count, an explicit error with retry, and one deep link per alert type ("Ver nevera", "Cocinar con tomate").
5. **[P2] Flattened hierarchy.**
   - Food cards sit inside the shelf tile, and "Agregar a la nevera" overflows it.
   - "Lo abrí", "Usar" and "Botar" carry the same weight.
   - Expiry is said twice.
   - Mercado opens on its form instead of the list.
   - Fix: rows for lists of the same kind; the list before the form; one hero figure per screen.

## Mechanical findings

- **Real detector findings:**
  - accent side borders on UI containers (`app.css:302, 1578, 1585`);
  - thick `border-bottom` accents on rounded elements (`361, 931, 953, 1240`);
  - an overshooting easing token (`tokens.css:72`);
  - cramped padding in the shelf tile;
  - 8.5 px text on the stage note label (`span.part-name`).
- **False positives:** the carton drawing (`.f-jugo`), the fridge hatching, and the typing dots (static under reduced motion).
- **Tap targets under 44 px:**
  - the alerts link (98×22, 91×20 on phones);
  - the brand (32×32 on phones);
  - stage parts on phones (24–35 px high);
  - market checkboxes (28×28);
  - 25 settings chips (38–41 px high).
- **Text under 14 px:**
  - header links 13 px;
  - shelf grams 13 px;
  - captions 12 px;
  - ring tags 12 px.
- **Overflow:** `.session` and Sign out past the viewport at 1440; shelf links clipped inside `.column` on the phone Nevera.
- **axe:** 0 violations at 1440 and 390 on six routes, light theme only. There is no dark theme today: `tokens.css` is light only.

## Persona notes

- **One-handed phone user:** navigation at the top and overflowing; "Cocinar" below the fold; the composer covering the prompts; "Quitar" wrapping under each market row.
- **First-timer:** nothing says the fridge is draggable; the hold is learned by failing; unclear labels ("Salva Tomate", "Repisas: Arriba", "Conectando").
- **Keyboard and screen reader:** focus order runs skip link, the nine stage parts, the tabs, then the header; the alerts link is empty at 0 unread; the Chef "sees" pills look like buttons but aren't; a live refresh of the Nevera can drop focus.

## Direction contract (refinement: the identity stays)

### Type

| Role | Face and size |
|---|---|
| Band title | Fraunces 900 uppercase, `clamp(36px, 4vw, 56px)` |
| Section title | Fraunces 900, 28 px, one per section |
| Hero figure | Hanken 700 tabular, 56 px, exactly one per screen (kg on Nevera, money on Números and Mercado, dish count on Ahora) |
| Card figure / row figure | 32 px / 20 px |
| Body | 17/1.5 |
| Small | 14 |
| Label | 13/600 sentence case, never uppercase-tracked |

### Spacing

8 pt rhythm:

| Where | Space |
|---|---|
| Inside a control | 8 |
| Between elements in a card or row | 16 |
| Card padding | 24 |
| Between sections | 32 |
| Band to first section | 48 |

Rows are at least 56 px tall.

### Card or row

- **Card** (2 px ink border, 4 px offset shadow, 6 px radius): only for an object acted on as a whole. That means a dish, a "use today" item, a pending AI confirmation, an unread alert or a sheet. Never a card inside a card.
- **Row** (20% ink hairline, no shadow): for lists of the same kind. That covers shelf contents, the market list, what the Chef sees, breakdowns and settings options.
- **Section**: a title with no container.

### Icons

- Engraved glyphs at 20 and 24 px: a 1.5–2 px ink stroke, with fill drawn as hatching.
- Drawn from the same food shapes as the 3D stage, replacing the CSS blobs.
- Navigation uses the same pen: fridge, list, coin ring, spoon, gear.
- No icon library and no emoji.

### Colour

- **Section colour:** belongs to the band, the active tab, the leader line and dot, and at most one data mark.
- **Status colours:** semantic and always paired with text. Papa = at risk, tomate = expired or danger, cilantro = fresh or done.
- **Cobre (copper):** the single primary next step per view. Every other button is an ink outline, and a destructive button is an ink outline with a tomate hold fill.
- **Mercado:** its band uses a deeper tomate so it doesn't read as an alarm.

### Motion

- Spring tokens only, with no overshoot on UI elements.
- One stamp for confirmations.
- The section changes direction with the tab order.
- Reduced motion keeps every state, without the travel.

## Decisions

- **Números:** celebrates the household instead of ranking members. Each person sees only their own figures, in positive terms.
- **Nevera panel:** keeps listing the inventory as light rows (glyph, name, grams, expiry). Actions appear when a row is chosen, and on phones the panel is the main view.
- **Dark theme:** "night paper" is created in this round and follows the system setting.
- **Tokens v3:** the two design systems are unified first, before the new top bar.
