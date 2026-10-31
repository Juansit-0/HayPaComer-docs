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

# Redesign close (2026-10-31)

Checked against plan v5 (header, tab bar, contextual fridge, transitions, swipes, microinteractions, kitchen stage, empty states) and the later landing stages, with axe-core 4.10.3 and headless Chrome on the mock: Now, Fridge, Market, Numbers, Chef and Settings, the add food sheet, the notifications and the account menu open, and the signed-out landing, at 1440 px with the 3D kitchen forced on and at 390 px, in light and dark.

## Found and fixed

| Problem | Where | Fix |
|---|---|---|
| Palette chips kept a white background with light text in dark mode (axe marks text over the 3D canvas as incomplete, so only a direct contrast check found it) | Fridge, chips to drag food onto a shelf | Chips use the theme surface and text colours |
| The page shifted on load (layout shift 0.38): the header first drew as the signed-out page, then as the app, and the fridge column slid to its place | Every signed-in screen, worst on Numbers | App mode is set before the first paint when a session exists; the first layout of a visit lands without the column animation |
| The credit of a CC BY-SA photo was hidden in the desktop dish rows | Now, dish rows next to the 3D kitchen | The credit shows under the picture in rows too |
| A request could wait forever when the server did not answer | Every screen | Every attempt ends after 75 s, a sleeping server is retried for up to 150 s, and a status note tells what is happening, then offers "Try again" |
| Food titles wider than their column pushed the kit pictures out of line | Landing, "Four parts" at about 1000 px | Fixed column width and a title that wraps and hyphenates |

## Confirmed

- No axe violations in any of the 40 checks (10 screens and states, at two widths, in light and dark), after the fixes.
- Frames: p95 16.7 to 16.8 ms on all six screens at 1440 px with the 3D kitchen, and at 390 px with the CPU slowed four times; no long tasks once a screen has settled. Loading a screen with the 3D kitchen costs one task of 120 to 230 ms (building the kitchen and compiling its shaders), once per visit.
- Route changes keep p95 16.7 ms; their layout shifts all happen within 500 ms of the tap, so they count as a response to input (layout shift 0 for the visitor).
- Keyboard: skip link, home, the five sections, theme, notifications, settings, account, then the 3D fridge; the account menu opens with Enter, its language, sound and sign out are reachable with Tab, and Escape closes it and returns focus to its button; the notifications open with focus inside and Escape returns it to the bell; arrow keys never change the section.
- Reduced motion: the same text on every screen as with motion, and no running animations 6.5 s after load.

## To check by hand

- iPhone and Safari: the bottom tab bar over the safe area, swipes between sections, the account popover, dragging the fridge door with a finger, the film grain.
- VoiceOver: the server status note is read when it appears.

