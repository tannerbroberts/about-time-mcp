# Facts

Facts are claims about resources, and each one carries its source. The system keeps every claim. When sources
disagree, it picks one by source preference (`user` > `label` > `database` > `recipe` > `derived` > `estimate`) and
keeps the others visible. **If a fact is missing, the item counts as unknown, never as zero.** Constraint checks
report it as unverifiable instead of passing it.

`add_facts` takes a list:

```json
{ "resource": "rolled oats", "property": "energy", "value": "379 kcal", "per": "100 g",
  "source": { "kind": "database", "ref": "FDC:173904", "title": "USDA FoodData Central: Oats, rolled", "url": "https://fdc.nal.usda.gov/…" } }
```

| Property | Value | Notes |
|---|---|---|
| `energy` | `"379 kcal"` | per `"100 g"`, `"1 cup"`, `"1 count"`… |
| `protein`, `fat`, `carbohydrate`, `fiber` | `"13 g"` | |
| `sodium`, `iron`, `calcium` | `"2 mg"` | |
| `folate` | `"56 ug"` | DFE |
| `price` | `"3.49 USD"` | per a package size, e.g. `"per": "28 oz"`. Add `"context": {"region": "US"}` or a store |
| `contains` | `["wheat", "gluten"]` | allergens and dietary flags. `[]` means verified to contain none of concern |
| `equivalence` | `"120 g"` per `"1 cup"` | lets cups meet per-100 g facts. `"50 g"` per `"1 count"` for an egg |
| `package` | `"18 fl_oz"`, `"1 bunch"`, `"5 lb"` | a size it is sold in (no `per`). The shopping list rounds up to whole packages |

**Allergen tag names.** Use lowercase common names: `milk`, `egg`, `fish`, `shellfish`, `tree-nut`, `peanut`, `wheat`,
`soy`, `sesame`, and also `gluten`. Add more for other concerns (`pork`, `alcohol`, `caffeine`), and reuse
names that already exist.

**Trivial ingredients still count.** If a constraint is about protein, water and salt need a fact too:
`{"resource": "water", "property": "protein", "value": "0 g", "per": "100 g", "source": {"kind": "database", "title": "USDA"}}`.
Otherwise they are unknown and the check cannot pass. The same applies to `contains` for oil, salt and spices.

**USDA nutrition in two calls.** `nutrition_search("lentils cooked")` lists candidate USDA foods. Then
`nutrition_apply({resource: "lentils", fdcId: 172421})` records every nutrient per 100 g, plus portion weights such as
"1 cup = 198 g", with the FDC id as the source. Match the form you weigh: raw or dry for ingredients going into a recipe.

**Where facts come from.** Nutrition comes from USDA FoodData Central or the package label. Price comes from a store
listing, with the date in `observedAt`. Allergens come from the product's label; for a generic whole food such as
broccoli, the `database` kind is fine. If you could not find something, leave it out. Do not estimate an allergen.
A label describes one package. For a hard allergen rule, it settles only what the person already has (in the profile's
`inventory`). Anything still to buy stays on their checklist, because brands and "may contain" warnings vary. Recording
a label for it is still useful, since it tells them what to look for, but expect the check to remain.

Facts attach to the resource *as bought*, such as "lentils" or "chicken breast", and flow through cooking automatically.
The nutrition of a portion is computed from what went into the batch.
