# about-time: overview

about-time turns what a person wants to do into a **plan they can actually follow**. A plan is nested,
ordered and timed. It knows what it consumes and produces, it is checked against the person's long-lived
constraints, and a person can tick it off step by step on a phone.

You, the agent, do the research and the data entry. The tools do the arithmetic: scheduling, totals,
shopping gaps, unit conversion, food-safety timing and constraint checks. **Never compute times, totals or
nutrition yourself.** Describe the activity faithfully and let `analyze_plan` tell you what follows from it.

## Mental model

| Concept | What it is |
|---|---|
| **Plan** (template) | The *idea* of an activity: steps, sub-steps, order, resources. It is saved, versioned and reusable. A recipe is a plan. A dinner made of two recipes is a plan that `use`s them. |
| **Step** | A leaf. It is one thing a person does (or waits for), with a duration, a "done when" cue, and the resources it takes, makes, borrows or needs. |
| **Resource** | Anything that flows: ingredients (consumed), things made (produced), equipment (borrowed while the step runs), and states ("oven (hot)", "dough (risen)"). |
| **Fact** | A sourced claim about a resource: nutrition, price, allergens, or "1 cup = 120 g". Facts can disagree. Missing facts are reported as unknown, never as zero. |
| **Profile** | Long-lived facts about the people: how many there are, what equipment they own, what is in the pantry, and their constraints (allergies, budget, targets). |
| **Instance** | One real attempt at a plan, recorded as events: started, done, skipped, swapped. It is not something you author. |

## Tools at a glance

- `guide`: these docs. The topics are `overview`, `workflow`, `authoring`, `quality`, `facts` and `profiles`.
- `set_profile` / `get_profile`: save the people's long-lived constraints.
- `lookup`: check whether a resource already exists in the vocabulary, and what is known about it.
- `save_plan`: save a plan written in the authoring format. It returns diagnostics to act on.
- `analyze_plan`: schedule a plan for a profile and number of servings. It returns a readable plan with each node's
  `{path}`, the shopping list, equipment, the checks the person must do, constraint verdicts and problems.
- `schedule_plan`: fix when and for whom it happens. The resulting id is what the person opens.
- `get_plan` / `list_plans`: fetch a saved plan back in authoring format to edit it, or browse saved plans.
- `add_facts`: record nutrition, price, allergen and conversion facts **with their sources**.
- `nutrition_search` / `nutrition_apply`: find an ingredient in USDA FoodData Central and record its nutrients and
  portion weights in one step.
- `merge_resources`: declare that two resource names mean the same thing.
- `start_instance` / `record_event` / `instance_status`: run a plan for real.

Start with `guide("workflow")`.
