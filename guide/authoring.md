# Authoring format

A plan is a JSON node. A node is one of these:
- a **step**, which has `time`
- a **group**, which has `steps`, run in order by default
- a **choice**, which has `options`
- a **repeat**, which has `repeat` plus `steps` or `use`
- a **reference**, which has `use: "<saved id>"`

## Step fields

| Field | Meaning | Example |
|---|---|---|
| `name` | Short, imperative | `"Whisk the batter"` |
| `do` | Full instructions for a first-timer | `"Whisk until no dry streaks remain; a few lumps are fine."` |
| `time` | Duration or range | `"5 min"`, `"5-7 min"`, `"~90 min (60-120)"`, `"2 days"` |
| `hands` | `"busy"` (default) or `"free"`: the person can do something else meanwhile | `"free"` |
| `doneWhen` | How to tell it's finished. Strings, or `{text, target:{what, op, value}}` | `["Bubbles on top start to pop"]` |
| `consumes` | What it uses up | `{"buttermilk": "2 cup", "egg": "2"}` |
| `produces` | What it makes. Add `keepsFor` for perishables | `{"batter": "1 batch"}`, `{"chili": {"qty": "8 servings", "keepsFor": "3-4 days"}}` |
| `uses` | Equipment held during the step and given back | `["griddle", "2 burner"]` |
| `requires` | A state that must hold for the whole step | `["griddle (hot)"]` |
| `settings` | Parameters | `{"griddle": "375 F"}` |
| `ifThen` | Mistake recovery | `[{"if": "Pancakes brown before bubbling", "then": "Lower heat to medium-low"}]` |
| `safety`, `tips` | Short strings | |
| `tags` | Free labels. Use `"meal"` for eating events | `["meal"]` |
| `source` | On a plan: credit for someone else's recipe or guide, shown with the plan | `{"url": "https://…", "title": "…", "author": "…"}` |

**States.** `"butter (melted)"` and `"butter@melted"` mean the resource *butter* in state *melted*. A step that
melts butter `consumes {"butter": "4 tbsp"}` and `produces {"butter (melted)": "4 tbsp"}`. Equipment works the same
way: preheating `produces {"oven (hot)": "1"}`, and baking `requires ["oven (hot)"]`.

**States that wear off.** When using something spends its state, `consume` the state and `produce` the spent one,
so the plan has to restore it before the next use. A cordless drill job `consumes {"drill battery (charged)": "1"}`
and `produces {"drill battery (flat)": "1"}`; "Charge the battery" turns it back. If a later job needs it charged
again and nothing recharges it, the analysis says so. The same goes for anything that must be clean, cold or
sharp before each use.

**Amounts.** Use `"2 cup"`, `"1/2 tsp"`, `"113-152 g"` (ranges are honest), `"? g"` (unknown amount), `"1 clove"`,
or `"1 can"`. A bare number counts items.

## Order

- Siblings run **in the order written**. Each one starts after the previous one finishes.
- `"after": ["key1", "key2"]` overrides that: start after these siblings. `"after": []` means no dependency.
- `"with": "key"` means start alongside that sibling.
- `"wait": "10 min"` sets a minimum gap after the `after` steps, and `"wait": "8-12 h"` gives both a minimum and a maximum.
- `"within": "15 min"` sets a maximum gap, for things that must not sit, such as searing right after the pan is hot.
- A group with `"parallel": true` has no default order. Its children are ordered only by `after` and by data flow:
  if one sibling produces what a later sibling consumes, the producer goes first automatically.
- `key` names a node among its siblings. It defaults to a slug of `name`, so "Whisk the batter" becomes `whisk-the-batter`.

## Groups, reuse, scaling

```json
{
  "id": "weekend-buttermilk-pancakes",
  "name": "Buttermilk pancakes",
  "yields": "4 servings",
  "makes": "pancakes",
  "steps": [
    { "key": "heat", "name": "Heat the griddle", "time": "8-10 min", "hands": "free", "uses": ["griddle"],
      "settings": {"griddle": "375 F"}, "produces": {"griddle (hot)": "1"}, "after": [] },
    { "name": "Make the batter", "steps": [
      { "name": "Melt the butter", "time": "1-2 min", "uses": ["small bowl"], "do": "Microwave in 20-second bursts.",
        "consumes": {"butter": "4 tbsp"}, "produces": {"butter (melted)": "4 tbsp"} },
      { "name": "Whisk dry ingredients", "time": "2 min", "uses": ["large bowl"],
        "consumes": {"all-purpose flour": "250 g", "sugar": "2 tbsp", "baking powder": "2 tsp", "salt": "1/2 tsp"},
        "produces": {"dry mix": "1"} },
      { "name": "Whisk in wet ingredients", "time": "2-3 min", "uses": ["large bowl"],
        "consumes": {"dry mix": "1", "buttermilk": "2 cup", "egg": "2", "butter (melted)": "4 tbsp"},
        "produces": {"batter": "1 batch"},
        "doneWhen": ["No dry streaks; small lumps are fine"],
        "ifThen": [{"if": "Batter is too thick to pour", "then": "Whisk in milk 1 tbsp at a time"}] }
    ]},
    { "name": "Cook the pancakes", "time": "12-16 min", "after": ["heat", "make-the-batter"], "within": "20 min",
      "requires": ["griddle (hot)"], "uses": ["griddle", "spatula"],
      "consumes": {"batter": "1 batch"}, "produces": {"pancakes": "4 servings"},
      "do": "Pour 1/4 cup per pancake. Flip once when bubbles pop and edges look dry.",
      "doneWhen": ["Both sides golden; center springs back"] }
  ]
}
```

- `"yields": "4 servings"` lets `analyze_plan` scale the recipe to the number of people.
- `"makes"` names what the recipe exists to produce.
- A meal is a group that `use`s saved recipes:
  `{"name": "Saturday brunch", "parallel": true, "steps": [{"key": "pancakes", "use": "weekend-buttermilk-pancakes", "name": "Pancakes"}, …]}`.
- `"scale": 2` on a node doubles it.

## Choices and repeats

```json
{ "name": "Mix the dough", "default": "mixer", "options": [
  { "key": "mixer", "name": "With a stand mixer", "when": "if you own one", "time": "6-8 min", "uses": ["stand mixer"] },
  { "key": "hand",  "name": "By hand", "time": "10-12 min", "doneWhen": ["Smooth and elastic"] } ] }

{ "name": "Intervals", "repeat": { "times": 6, "gap": "0 min" }, "steps": [
  { "name": "Jog", "time": "90 s" }, { "name": "Walk", "time": "2 min" } ] }
```

`analyze_plan` takes `choices: {"<path>": "<option key>"}` to pick an option.

## Places

Steps happen somewhere. `"at": "friend's house"` on a step or group says where (sub-steps inherit it), and the
default is home. Things are tracked where they are. A step somewhere else can only use what is there, so carrying is a step:

```json
{ "name": "Potluck", "steps": [
  { "key": "bake", "name": "Bake the casserole", "time": "45 min", "hands": "free", "uses": ["oven", "casserole dish"],
    "consumes": {"casserole (assembled)": "1"},
    "produces": {"casserole (hot)": {"qty": "1", "keepsFor": "2 h"}} },
  { "key": "carry", "name": "Wrap in towels in an insulated carrier and drive it over", "time": "20 min",
    "moves": {"casserole (hot)": "1"}, "to": "friend's house", "holdsFor": "2 h", "uses": ["insulated carrier"] },
  { "key": "serve", "name": "Serve", "at": "friend's house", "time": "10 min", "consumes": {"casserole (hot)": "1"} }
]}
```

If a step needs something that only exists somewhere else, `analyze_plan` reports **wrong-place**. `holdsFor` is how
long the way it is carried keeps it safe, and it caps the shelf life. If you forget the carrier, the food-safety
check catches it. Mark outdoor places in the profile (`"places": {"trailhead": {"outdoors": true}}`, plus a `location`)
and the analysis warns about outdoor work in the dark.

**The person is in one place at a time.** A step with `to` and no `moves` is a trip:
`{"name": "Walk to the library", "time": "15 min", "to": "library"}`, and a carry step is a trip too. If two
hands-on steps in a row are at different places with no trip between them, `analyze_plan` warns (**one-body**).
Record travel times in the profile's places, and it also checks that each trip and gap is long enough.

## Paths

A node's path is its keys from the root joined by `/`, for example `make-the-batter/whisk-in-wet-ingredients`. Paths are
used for `choices`, `swaps`, `pins` and instance events.
