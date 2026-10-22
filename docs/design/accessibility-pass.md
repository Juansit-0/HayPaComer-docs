# Usability and accessibility pass (plan v2, step B7)

Checked on 2026-10-09 in the browser against the local app with axe-core 4.10 (rules WCAG 2.0 A and AA, WCAG 2.1 AA, and best practices), keyboard only, and phone and desktop sizes, in light and dark themes.

## Screens checked

Sign in, Now, Fridge with the add food sheet, Market, Numbers, Chef, and Settings, each at 1280 px and 375 px wide, in light and dark.

## Found and fixed

| Problem | Where | Fix |
|---|---|---|
| Copper text 4.31:1 on the tile surface (needs 4.5:1) | Links, ghost buttons, "Change the budget" in light theme | New token `--color-primary-text` (copper-700 `#9a3412`, 6.28:1); primary buttons keep copper-600 with white text (5.02:1) |
| Attention text 4.45:1 on the tile surface | "Expiring" status in light theme | `--color-attention` light is paprika-700 `#b23a0b` (5.15:1; white on it 5.99:1) |
| Attention text 4.26:1 on the dark surface | "Expiring" status in dark theme | `--color-attention` dark is orange-500 `#f97316` |
| Buttons kept the old theme color after the system theme changed (3.48:1) | Every primary button when switching light and dark with the page open | Removed the background transition on buttons; hover is instant |
| A `header` inside the sheet counted as a second banner landmark | Add food sheet | The sheet heading row is a `div` |

## Confirmed

- No axe violations on any screen in light or dark after the fixes.
- Keyboard order on sign in: skip link, home, language, email, password, sign in; focus is always visible (2 px ring with offset).
- The add food sheet moves focus inside when it opens and returns it to "Add to fridge" when Escape closes it.
- No horizontal scroll at 375 px, and no button, input, or chip under 44 px.
- Every visible text comes from the translations (`WebTextsTest`), so screen readers read the chosen language; `html lang` follows it.

# Engraving restyle pass (2026-10-20)

Checked against the engraving restyle of the landing and the app screens with axe-core 4.10.2: Now, Fridge, Market, Numbers, Chef, Settings, and the inbox sheet at 1440 px, Now and Fridge at 390 px, and the signed-out landing at both widths.

## Found and fixed

| Problem | Where | Fix |
|---|---|---|
| Inactive step names 3.77:1 on paper | Landing, "How it works" slider | Solid ink brown `#5c554c` (6.07:1) instead of ink at 55% |
| Text 3.56:1 on tomato red | Landing calculator label and explanation | The chapter red deepens to `#a8361f` and its text is solid paper (4.97:1) |
| Dish titles were `h3` right after the page `h1` | Now, Chef dish cards | Dish titles are `h2` |

## Confirmed

- No axe violations on any checked screen after the fixes.
- Sheets (inbox, add food) share the card language: paper, a 2 px ink edge, and a hard offset shadow; unread alerts add a tomato edge as well as the word, so the state is not told by colour alone.
- Secondary buttons and chips are outlined in ink and stay at 44 px or more.

# Motion pass (2026-10-21)

Checked against the motion system of plan v4 (virtual scroll story, route timeline, springs, sheets, hold, sounds) with axe-core 4.10.3 and headless Chrome on the mock: the landing story captured, released, and static (reduced motion), Now, Fridge, Market, Numbers, Chef, and Settings at 1440 px and 390 px, and the inbox sheet open.

## Found and fixed

| Problem | Where | Fix |
|---|---|---|
| "Expiring" tags on the 3D fridge pulsed forever (WCAG 2.2.2) | Every app screen on desktop | The pulse plays twice and stops |
| The shelf instructions were `display: none` with the 3D fridge, so screen readers lost them | Fridge | Visually hidden instead; the text in `main` is the same with and without reduced motion |
| The phone page stretched to 1218 px; sheets opened off to the left | Every app screen at 390 px | The ragged header edge is clipped by the shell |
| The phone fridge shrank while scrolling and shifted the page (layout shift about 3) | Every app screen at 390 px | It slides up under its tabs; layout shift 0 |
| "Skip to content" changed the route | Header skip link | It focuses `main` without touching the hash |
| A food picked on another shelf lost focus and scroll | Fridge from the 3D model | Focus and scroll reset before the next screen renders, then the card is centered and focused |
| The busy spinner kept turning under reduced motion | Buttons while a request runs | Static ring under reduced motion |

## Confirmed

- No axe violations in any checked state, including a sheet open and the story captured, released, and static.
- Nothing moves 5 s after the last interaction: 0 running animations after 6.5 s idle on every screen, and the 3D scenes stop rendering (8 Hz paper boil, then frozen).
- Reduced motion: the story is not mounted (engravings, stills, and all nine captions on native scroll), routes swap without ink, and press and hold keeps its 800 ms fill with a colour change instead of the stamp.
- Story: captions stay in the DOM in order with the full sentence for screen readers and the word layer hidden from them; no live region. Space and Page Down go to the next caption, Home to the start, End skips, and any native scroll (focus, find in page, a screen reader) releases the story. Two-finger gestures are never captured, so pinch zoom works.
- "Skip the story" focuses the answer heading without changing the route. From the top of the page it is the fourth Tab stop, after "Skip to content", the home link, and the language.
- At 320 px (400 % zoom of a 1280 px window) the story has no horizontal scroll, captions stay inside the screen, and the chapter buttons wrap to two lines.
- Sound is off on the first visit, starts only from a click on "Sound", and the choice is remembered; it can also be changed in Settings.

## To check by hand

- VoiceOver on Mac and iOS: captions read in order, the hold button announced, skip lands on the answer.
- Keyboard only through a whole visit: story, sign up, Fridge with a sheet, hold to discard.
- A real iPhone and a mid-range Android: no rubber band or URL bar jump in the story, inertia, pinch zoom, the hold with a finger.
