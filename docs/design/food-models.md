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
| Papa criolla | To add | Small round yellow potatoes in a little pile |
| Tomato (chonto) | Done | Lobed red fruit, sepals and stalk |
| Onion (cabezona) | Done | Golden bulb drawn to a dry tip, root tuft |
| Cebolla larga | To add | Long white stem turning into green leaves, tied in a bunch |
| Carrot | Done | Tapering root with three fronds, bunched side by side |
| Arracacha | To add | Pale yellow knobbly root |
| Ulluco | To add | Small glossy yellow and pink tubers |
| Oca | Done | Short ridged tuber, rose, yellow or ivory |
| Yuca | To add | Long bark-brown root with a white cut end |
| Green peas (arveja) | To add | A few open pods with peas |
| Corn (mazorca) | To add | Ear with rows of kernels and husk peeled back |
| Cabbage (repollo) | To add | Round head with wrapped outer leaves |
| Lettuce | To add | Open head of ruffled leaves |
| Cilantro | To add | Bunch of thin stems with lacy leaves |
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
| Orange | To add | Round, dimpled peel, small leaf at the stem |
| Lime (limón) | To add | Small green oval, one cut in half |
| Mango | To add | Kidney-shaped, green to red blush |
| Papaya | To add | Long fruit, one half cut showing seeds |
| Granadilla | To add | Orange shell, curled stalk |
| Mora (blackberries) | To add | Little tub of dark berries |
| Apple | To add | Red apple with stalk and leaf |
| Pineapple | To add | Diamond-patterned body with a crown of leaves |

## Dairy and eggs

| Food | Status | What the model shows |
|---|---|---|
| Milk | Done | Carton with coloured bands and cap |
| Cheese (queso campesino) | Done | Round of fresh white cheese with a slice cut out |
| Queso doble crema | To add | Yellower block in its plastic |
| Yogurt | Done | Cup with foil lid |
| Butter | Done | Block half out of its paper |
| Kumis | To add | Bottle with a white body and red cap |
| Eggs | Done | Carton with the number of eggs held |

## Meat, poultry and fish

| Food | Status | What the model shows |
|---|---|---|
| Chicken breast | Done | Breast on a white tray |
| Whole chicken / thighs | To add | Thighs with skin on a tray |
| Ground beef | Done | Mound on a tray |
| Beef steak (carne de res) | To add | Flat red cut with a fat edge |
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

The 24 foods marked **Done** are the whole catalog today (`V4__seed_food_catalog.sql`). The others need a migration that adds them to `food_catalog` (with es-CO and en names) and a model each in `app/engrave.js`. Suggested order, by how often they end up in a Pasto fridge: papa criolla, cebolla larga, cilantro, lime, orange, queso doble crema, beef steak, whole chicken, mora, mango, arveja, mazorca, then the rest.
