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
| Carrot | Done | Tapering root with three fronds, bunched side by side |
| Arracacha | To add | Pale yellow knobbly root |
| Ulluco | To add | Small glossy yellow and pink tubers |
| Oca | Done | Short ridged tuber, rose, yellow or ivory |
| Yuca | To add | Long bark-brown root with a white cut end |
| Green peas (arveja) | Done | Two pods split open with the peas in a row, one still closed |
| Corn (mazorca) | Done | Ear with rows of kernels, husk peeled back and a tuft of silk |
| Cabbage (repollo) | To add | Round head with wrapped outer leaves |
| Lettuce | To add | Open head of ruffled leaves |
| Cilantro | Done | Bunch of thin stems with lobed leaves, tied at the roots |
| Garlic | To add | White bulb with cloves and a dry tip |
| Pepper (pimentón) | To add | Glossy red pepper with four lobes and a stalk |
| Ahuyama | To add | Wedge of squash, green rind and orange flesh |

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
| Papaya | To add | Long fruit, one half cut showing seeds |
| Granadilla | To add | Orange shell, curled stalk |
| Blackberries (mora) | Done | Clear tub of dark berries, each a knot of drupelets |
| Apple | To add | Red apple with stalk and leaf |
| Pineapple | To add | Diamond-patterned body with a crown of leaves |

## Dairy and eggs

| Food | Status | What the model shows |
|---|---|---|
| Milk | Done | Carton with coloured bands and cap |
| Cheese (queso campesino) | Done | Round of fresh white cheese with a slice cut out |
| Double cream cheese (queso doble crema) | Done | Yellower block in clear plastic crimped at the ends, blue band |
| Yogurt | Done | Cup with foil lid |
| Butter | Done | Block half out of its paper |
| Kumis | To add | Bottle with a white body and red cap |
| Eggs | Done | Carton with the number of eggs held |

## Meat, poultry and fish

| Food | Status | What the model shows |
|---|---|---|
| Chicken breast | Done | Breast on a white tray |
| Chicken thighs (pernil) | Done | Skin-on drumsticks with the bone showing, on a tray |
| Ground beef | Done | Mound on a tray |
| Beef steak (carne de res) | Done | Flat marbled red cut with its edge of fat, on a tray |
| Pork chop | To add | Pink chop with the bone |
| Cuy | To add | Prepared cuy on a platter (a dish of Nariño) |
| Chorizo | To add | Linked sausages |
| Tuna | Done | Tin with label and rims |
| Trout (trucha) | To add | Whole fish, spotted skin |

## Grains, legumes and bakery

| Food | Status | What the model shows |
|---|---|---|
| Rice | Done | Soft sack with folded top and red band |
| Pasta | Done | Box with a window onto spaghetti |
| Bread (pan de sal) | Done | Round rolls with a scored top |
| Arepa | Done | Disc with griddle marks |
| Black beans | Done | Glass jar |
| Lentils | Done | Glass jar |
| Red beans (fríjol) | To add | Jar of red-brown beans |
| Mote (maíz pelado) | To add | Bag of large white corn kernels |
| Oats (avena) | To add | Round cardboard tub |
| Panela | To add | Brown block wrapped in leaf |

## Drinks, condiments and prepared food

| Food | Status | What the model shows |
|---|---|---|
| Orange juice | Done | Carton with a slice drawn on |
| Soy sauce | Done | Dark bottle |
| Soup | Done | Pot with a lid |
| Ají | To add | Small jar of red-green sauce |
| Oil | To add | Clear bottle of yellow oil |
| Coffee | To add | Bag of ground coffee |
| Leftovers (sobras) | To add | Glass container with a lid and food inside |

## Next step

The first 24 foods come from `V4__seed_food_catalog.sql`; the next 15 from `V53__more_foods.sql` (the 12 above plus tree tomato, oca and lulo, which had models but no catalog row), which adds them to `food_catalog` with shelf life by zone, a reference price in COP per kg and their es-CO and en names. Every model can be reviewed together at `app/dev/foods.html` (`?only=Lime,Corn` to pick some). The rest follow the same steps, a row in a migration and a model in `app/engrave.js`, in this order: arracacha, ulluco, yuca, garlic, pepper, ahuyama, papaya, granadilla, kumis, chorizo, trout, panela, then the others.
