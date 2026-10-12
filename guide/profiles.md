# Profiles and constraints

```json
{
  "id": "harbor-street-household",
  "label": "Harbor Street household",
  "people": 3,
  "actors": 1,
  "timeZone": "America/Chicago",
  "location": { "lat": 41.88, "lon": -87.63 },
  "places": { "backyard": { "outdoors": true }, "community garden": { "outdoors": true }, "gym": { "travel": { "home": "10 min" } } },
  "equipment": { "oven": 1, "burner": 4, "large skillet": 1, "sheet pan": 2, "blender": 0 },
  "inventory": { "olive oil": "500 ml", "salt": "1 cup", "rice": "2 kg" },
  "constraints": [
    { "id": "shellfish", "label": "No shellfish (Sam, allergic)", "property": "contains", "op": "excludes",
      "value": ["shellfish"], "severity": "hard", "ifUnknown": "fail", "evidence": ["label", "user"] },
    { "id": "gluten", "label": "Gluten-free (Ana, celiac)", "property": "contains", "op": "excludes",
      "value": ["gluten", "wheat"], "severity": "hard", "ifUnknown": "fail" },
    { "id": "budget", "label": "Groceries under $25 per dinner", "property": "price", "op": "<=", "value": "25 USD",
      "severity": "soft", "ifUnknown": "warn" },
    { "id": "sodium", "label": "Under 700 mg sodium per serving", "property": "sodium", "op": "<=", "value": "700 mg",
      "scope": { "perServing": true }, "severity": "soft", "ifUnknown": "warn" },
    { "id": "quick", "label": "Ready within 40 minutes of starting", "property": "elapsed", "op": "<=", "value": "40 min",
      "severity": "soft", "ifUnknown": "warn" },
    { "id": "fiber-day", "label": "At least 25 g fiber per day", "property": "fiber", "op": ">=", "value": "25 g",
      "scope": { "tag": "meal", "per": "day", "measure": "absorbed" }, "severity": "soft", "ifUnknown": "warn" }
  ]
}
```

- `equipment` lists what they own and how many. The scheduler won't use two of something they have one of.
  List `0` for things you know they don't have, and those are reported as **missing**. Equipment you don't list is
  reported as "make sure you have", because it is unknown rather than absent.
- `location` and `places` say where the profile's people are, and which places are outdoors. They are used for daylight warnings.
  `travel` gives the time to get between two places (either direction), so the analysis can check the person has time to get there.
- `inventory` is what they already have. `analyze_plan` subtracts it to make the shopping list.
- `people` is how many people eat. It does **not** scale anything by itself, because a 5-lunch meal prep for one person
  is 5 servings. Pass `servings` to `analyze_plan` or `schedule_plan` to scale.
- `actors` is how many people are cooking or doing the work.

## Constraints

| Field | Meaning |
|---|---|
| `property` | Which fact: `contains`, `energy`, `protein`, `price`, `sodium`, … Also `elapsed` (start to finish) and `hands-on` (active time), measured on the schedule |
| `op` | `<=`, `>=`, `excludes` (none of these tags), or `includes` (all of these tags) |
| `scope` | Omit it for the whole plan's ingredients. `{"perServing": true}` divides by `yields`. `{"tag": "meal", "per": "each" \| "day" \| "all", "measure": "absorbed"}` judges what is actually eaten, per meal or per calendar day |
| `severity` | `hard`: the plan is not acceptable if it fails. `soft`: a warning |
| `ifUnknown` | What to conclude when facts are missing. Allergies and medical restrictions should use `fail` |
| `evidence` | Which source kinds count as proof. For a serious allergy use `["label", "user"]`. A database entry for a spice is not proof against cross-contact, and the label is. Anything without such evidence becomes a check the person does. **An `estimate` never proves a hard constraint**, whatever this says. Hard "contains" rules default to label, user or database evidence |

Encode long-lived conditions such as diabetes, a low-sodium diet or a training goal as several concrete constraints, each with a `why` naming its
source. A plan only knows what it is told.
