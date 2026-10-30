# Food models

Every food HayPaComer can hold is drawn as one 3D model in the engraving style (`app/engrave.js`, `foodModel`). The same model serves the landing story, the 3D fridge in the app and the drag-and-drop shelf chips. This list is the set of everyday unit foods of a household in Pasto that deserve their own model, so nothing in a real fridge falls back to the generic category shape.

A model shows what the food is at a glance from a metre away: its silhouette first, then one or two details (a stalk, a band of colour, the way it is sold). Packaged foods are drawn in their usual container, loose foods as one piece or a small pile sized by grams.

## Status

- **Done:** has its own model now.
- **To add:** needs a model and a row in the food catalog (migration with `name_key`, category, unit, grams per piece and shelf days).

## Vegetables and tubers

| Food | Status | What the model shows |
|---|---|---|
| Potato (papa pastusa) | Done | Smooth pale tuber with eyes; three varieties in the market |
| Creole potato (papa criolla) | Done | Small round yellow potatoes in a little pile |
| Tomato (chonto) | Done | Lobed red fruit, sepals and stalk |
| Onion (cabezona) | Done | Golden bulb drawn to a dry tip, root tuft |
| Green onion (cebolla larga) | Done | Three white stems going pale green into hollow leaves, tied with a red band |
| Carrot | Done | Root swelling from its point to a rounded shoulder, faint rings, a feathery top from the crown, bunched side by side |
| Arracacha | Done | Two pale yellow knobbly roots and the purple knob of their crown |
| Ulluco | Done | Small glossy yellow and pink tubers in a pile |
| Oca | Done | Short ridged tuber, rose, yellow or ivory |
| Yuca | Done | Long bark-brown root with a white cut end and the line of its core |
| Green peas (arveja) | Done | Two pods split open with the peas in a row, one still closed |
| Corn (mazorca) | Done | Ear with rows of kernels, husk peeled back and a tuft of silk |
| Cabbage (repollo) | Done | Tight pale head in darker outer leaves that flare at the top |
| Lettuce | Done | Ruffled leaves opening from the heart in two rings |
| Cilantro | Done | Bunch of thin stems with lobed leaves, tied at the roots |
| Garlic (ajo) | Done | White bulb pinched into cloves, a dry tip and one loose clove |
| Pepper (pimentón) | Done | Glossy red pepper with four lobes at its foot and a bent stalk |
| Squash (ahuyama) | Done | Wedge of ribbed squash, green rind, orange cut faces and seeds |

## Fruit

| Food | Status | What the model shows |
|---|---|---|
| Banana | Done | Curved ridged finger, dark tips |
| Plantain | Done | Longer, straighter, green going yellow |
| Avocado | Done | Pear-shaped, pebbled dark skin |
| Tree tomato (tomate de árbol) | Done | Pointed red egg, sepals, short stalk |
| Lulo (naranjilla) | Done | Round orange fruit with dry calyx |
| Orange | Done | Round, dimpled peel, small leaf at the stem |
| Lime (limón) | Done | Small green oval, one cut in half showing its segments |
| Mango | Done | Kidney-shaped, sun-red on top going to yellow-green |
| Papaya | Done | Half a papaya on its back, orange flesh round a hollow of black seeds |
| Granadilla | Done | Orange shell freckled pale, curled stalk and a dry bract |
| Blackberries (mora) | Done | Clear tub of dark berries, each a knot of drupelets |
| Apple | Done | Red apple with a dip at the stalk and one leaf |
| Pineapple | Done | Body of diamond eyes and a crown of long stiff leaves |

## Dairy and eggs

| Food | Status | What the model shows |
|---|---|---|
| Milk | Done | Carton with coloured bands and cap |
| Cheese (queso campesino) | Done | Round of fresh white cheese with a slice cut out |
| Double cream cheese (queso doble crema) | Done | Yellower block in clear plastic crimped at the ends, blue band |
| Yogurt | Done | Cup with foil lid |
| Butter | Done | Block half out of its paper |
| Kumis | Done | White bottle with a green label band and a red cap |
| Eggs | Done | Carton with the number of eggs held |

## Meat, poultry and fish

| Food | Status | What the model shows |
|---|---|---|
| Chicken breast | Done | Breast on a white tray |
| Chicken thighs (pernil) | Done | Skin-on drumsticks with the bone showing, on a tray |
| Ground beef | Done | Mound on a tray |
| Beef steak (carne de res) | Done | Flat marbled red cut with its edge of fat, on a tray |
| Pork chop | Done | Pale pink chop round the bone, rim of fat, on a tray |
| Cuy | Done | Roasted cuy on a platter with small yellow potatoes (a dish of Nariño) |
| Chorizo | Done | A curved chain of links tied at the ends |
| Tuna | Done | Tin with label and rims |
| Trout (trucha) | Done | Whole fish on its side, olive back, pink stripe and dark spots |

## Grains, legumes and bakery

| Food | Status | What the model shows |
|---|---|---|
| Rice | Done | Soft sack with folded top and red band |
| Pasta | Done | Box with a window onto spaghetti |
| Bread (pan de sal) | Done | Round rolls with a scored top |
| Arepa | Done | Disc with griddle marks |
| Black beans | Done | Glass jar |
| Lentils | Done | Glass jar |
| Red beans (fríjol) | Done | Glass jar of red-brown beans with a paper label |
| Mote (maíz pelado) | Done | Clear bag of large white kernels with a yellow band, tied at the neck |
| Oats (avena) | Done | Round cardboard tub with a printed band and a lid |
| Panela | Done | Two round cakes half wrapped in dry plantain leaf, tied with string |

## Drinks, condiments and prepared food

| Food | Status | What the model shows |
|---|---|---|
| Orange juice | Done | Carton with a slice drawn on |
| Soy sauce | Done | Dark bottle |
| Soup | Done | Pot with a lid |
| Ají | Done | Small jar of red sauce with green herbs, dark lid |
| Oil | Done | Tall clear bottle of golden oil, green label and cap |
| Coffee | Done | Dark stand-up pouch, top folded, pale label and valve |
| Leftovers (sobras) | Done | Glass container with a blue snap lid, rice and stew inside |

## Next step

Every food on this list now has its model and its catalog row: 24 from `V4__seed_food_catalog.sql`, 15 from `V53__more_foods.sql`, 12 from `V54__andean_foods.sql` and 13 from `V55__pantry_and_kitchen_foods.sql`, each with shelf life by zone, a reference price in COP per kg and its es-CO and en names. Every model can be reviewed together at `app/dev/foods.html` (`?only=Lime,Corn` to pick some). A new food follows the same steps: a row in a migration, a model in `FOODS` (and `DETAILED` when grams change how many pieces show) in `app/engrave.js`, and a flat glyph in `SHAPES` (`app/stage.js`) with its class in `app/app.css`.
