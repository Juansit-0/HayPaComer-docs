# App research: how real food apps solve navigation, lists and cooking

Read-only research (2026-10-26) for plan v5, the signed-in app redesign. Sources are public pages and design critiques; app store screenshots were not readable, so observations are what each source states. Rappi, Cookpad's tab bar and cook mode, Fridgely and KitchenPal could not be verified and are left out on purpose; designer concepts on Dribbble or Behance are not evidence of a real app.

## Observations

| Topic | App | What it does | Source |
|---|---|---|---|
| Navigation | Too Good To Go | Four bottom tabs (Discover, Explore/Browse, Favorites, More/Me); active tab is a filled green icon, inactive grey; screen title top left | [orientatech](https://orientatech.es/en/?p=8230), [Pratt iOS critique](https://ixd.prattsi.org/2023/02/design-critique-too-good-to-go-ios-app/) |
| Navigation | Too Good To Go | A lone filter icon on Browse was hard to discover | [Pratt Android critique](https://ixd.prattsi.org/2023/09/design-critique-too-good-to-go-android-app/) |
| Navigation | Mealime | No tab bar: the meal plan is home, the grocery list sits behind an icon that shows an alert when a list is ready | [Mealime guide](https://support.mealime.com/article/151-getting-started-guide) |
| Lists | Too Good To Go | Horizontal shelves with "View all"; cards carry text stamps ("New", "Last chance") and the pickup time, which is lost in body type | [orientatech](https://orientatech.es/en/?p=8230), [Pratt Android critique](https://ixd.prattsi.org/2023/09/design-critique-too-good-to-go-android-app/) |
| Lists | NoWaste | Inventory rows with expiry status, storage place and quantity; fridge, freezer and pantry lists; sort by expiry; swipe actions; batch edit; undo delete; an expiry calendar and an impact screen with a waste rate | [nowasteapp.com](https://www.nowasteapp.com), [App Store](https://apps.apple.com/us/app/nowaste-food-inventory-list/id926211004) |
| Adding food | NoWaste | Manual, barcode, receipt, one photo for several items, expiry scanning, 200+ common foods | [nowasteapp.com](https://www.nowasteapp.com) |
| Adding food | Paprika | Pantry items carry quantity, purchase and expiry dates and move to and from the grocery list | [App Store](https://apps.apple.com/id/app/id1303222868) |
| Shopping list | Mealime | Ingredients merged across recipes and sorted by department; one text field at the top adds and auto-categorises; tapping a row opens substitutes, recipes that use it and how it looks | [Mealime guide](https://support.mealime.com/article/151-getting-started-guide) |
| Shopping list | Paprika | Sorted by aisle, custom aisles reorderable | [App Store](https://apps.apple.com/id/app/id1303222868) |
| Shopping list | NoWaste | "Add to shopping list" straight from a consumed item | [App Store](https://apps.apple.com/us/app/nowaste-food-inventory-list/id926211004) |
| Motion | Too Good To Go | A countdown bar replaces the reserve button until pickup opens, then swipe to confirm | [Pratt iOS critique](https://ixd.prattsi.org/2023/02/design-critique-too-good-to-go-ios-app/) |
| Motion | Mealime | A cooked recipe fades on the plan; hands-free step changes fade through black | [Mealime cooking mode](https://support.mealime.com/article/88-how-to-use-cooking-mode) |
| Cook mode | Mealime | One large instruction at a time, arrows between steps, "Done" marks the recipe cooked | [Mealime cooking mode](https://support.mealime.com/article/88-how-to-use-cooking-mode) |
| Cook mode | Paprika | Screen stays on, ingredients cross off, current step highlighted, times inside steps start a timer when tapped | [App Store](https://apps.apple.com/id/app/id1303222868) |
| Cook mode | Kitchen Stories | Distraction-free mode, unit converter, a timer per step | [App Store](https://apps.apple.com/app/kitchen-stories/id771068291) |
| Community | Cookpad | A photo of the finished dish ("cooksnap") replaces most text reviews | [Econsultancy](https://econsultancy.com/how-ux-makes-cookpad-s-app-the-best-antidote-to-instagram-for-foodies/) |

No source documented empty or loading states, check-off animation or budget totals for these apps.

## What HayPaComer takes

- **Navigation:** a phone bottom bar with the five sections, the active one filled in its section colour with a label; the bell and account in the top bar; the screen title top left. No unlabelled icon controls.
- **Now:** the key fact (days left, minutes to cook) set larger than body text; stamps such as "Úsalo hoy" in the engraving language, not generic pills.
- **Fridge:** rows with expiry, shelf and grams; undo after use and discard; the fast catalogue path stays the default in the add sheet.
- **Market:** the list first, one add field that files the item under its aisle, a row detail with substitutes and recipes, "Pasar a la nevera" when an item is checked.
- **Chef:** one large step at a time, ingredients crossed off beside it, times in the text that start a timer, the screen kept awake, voice as the hands-free mode.
- **Inbox:** alerts grouped by urgency with one action each (use, cook, add to market).
- **Motion:** one stamp on check-off and confirmation, a fade for finished things; no confetti, which no app here uses.
- **Words:** one term per concept, everywhere.

## 21st.dev references (retrieved 2026-10-27)

Reimplemented in vanilla code, nothing copied:

- [Tabs by ddoemonn](https://21st.dev/@ddoemonn/components/tabs): indicator measured from the active tab and kept on resize; single tab stop with arrows, Home and End; content enters 12 px from the side of the change; instant under reduced motion. Used for the section navigation and the direction of route changes.
- [Notification Panel by uvain](https://21st.dev/@uvain/components/notification-panel): each row is a sentence with only the subject and object emphasised; actions resolve in place into a quiet line; repeats collapse with a count; "inbox" holds what still needs something, apart from "all". Used for the bell and the inbox.
