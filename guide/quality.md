# Quality: writing for someone who has never done this

The reader is standing in a kitchen (or garage, or park) with a phone, has never done this before, and will
follow your plan literally. They need enough time to read each step, do it, notice when it's done, recover from
a mistake, and move on.

Check every step:

- **One action per step.** "Whisk the eggs, then fold in the flour and preheat the oven" is three steps.
  Long steps hide the moment where things go wrong.
- **A realistic time range for a first-timer.** Recipe times are optimistic, and chopping an onion takes a novice twice as long.
  Use ranges.
- **`doneWhen` on every step that ends on a condition.** Use something a person can see, hear, smell, feel or
  measure. "Cook 10 minutes" is a guess; "until the edges are golden and the center is set, about 10 minutes" is
  an instruction.
- **`ifThen` for the common failures** at that step: too thick, too dark, not rising, smoking, split sauce.
- **Waits are marked `"hands": "free"`**, so the plan can say what to do meanwhile and doesn't waste the person's time.
- **Nothing is used before it is introduced.** Each ingredient, tool and state appears in a step before it is needed.
  If an ingredient must be at room temperature, softened or thawed, write that as a step, with its own wait.
- **Parallel work is realistic for the number of people.** One person cannot stir constantly and chop at the same time.
  The scheduler enforces this if you mark attention honestly.
- **The setting is considered.** Think about the time of day and the season (light, heat, cold, weather), the space
  involved, and anything that moves between places. Does something need to stay cold or hot on the way, or does a
  tool have to be carried?
- **Safety is explicit.** Include internal temperatures with a `target`, raw-meat handling, hot pans, sharp tools and allergen
  cross-contact.
- **The person's units and equipment.** Don't require a stand mixer they don't own. Offer a `choice`, or pick a method
  that works with what they have.
- **Things that spoil while waiting say so.** Give perishable in-between products `keepsFor` (cooked food or
  raw meat at room temperature: `"2 h"`; in the fridge: days), and add the step that puts them in the fridge.
  The analysis then catches anything left out too long.
- **Good places to pause** are visible. A long `hands: "free"` wait is one, and so is a step whose product `keepsFor` a while.

After `analyze_plan`, read the rendered plan from top to bottom as the novice would. If you would hesitate at
any line, fix it.
