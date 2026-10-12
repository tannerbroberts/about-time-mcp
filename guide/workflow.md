# Workflow: from a goal to a plan someone can follow

1. **Understand the people.** Call `get_profile` if one exists, or `set_profile` with what you were told: number of people,
   equipment owned, what is on hand, and constraints. Encode every constraint the user states and do not paraphrase it away.
   If something important is unknown, such as whether they own a specific pan, the plan should either avoid needing it or
   flag it. Do not assume.

2. **Find a source.** Search for well-regarded recipes or how-to guides. Prefer sources that give weights, times
   **and** sensory cues ("until golden"). Judge fit against the profile *before* investing in a candidate. Check the
   cuisine, the total time, the equipment, the cost, and the ingredients that conflict with constraints. Look for hidden
   ones too: sauces, spice blends and stocks often hide allergens.

3. **Write the plan** (see `guide("authoring")` and `guide("quality")`):
   - Mirror the source's structure as a hierarchy. Group steps the way a person thinks of them ("Make the dough", then
     "Knead"). Steps run in the order written unless you say otherwise.
   - Mark waiting steps `"hands": "free"` (rising, simmering, roasting, chilling). This is what lets one person do two
     things at once. Use `after` / `with` so that work can happen during waits.
   - Every step gets a realistic `time` range and a `doneWhen` cue. Add `ifThen` for the common ways it goes wrong.
   - Name resources in plain words, and reuse existing names (`lookup`) so totals and shopping lists line up.
   - Give each reusable recipe an `id` so other plans can `use` it.
   - **If you mention an alternative, model it.** "Roll it with a pasta machine, or by hand with a rolling pin" must be a `choice` with full
     steps for *each* option. A person who takes the alternative gets no instructions if it only exists in your prose.
   - **Things that must happen earlier get their own time.** Buying, thawing, soaking and borrowing a tool belong in steps
     scheduled before the main event (`wait`, or `pins` such as the evening before), not at minute zero of the activity.
   - **Later follow-ups** ("re-tighten the wheel nuts after 50 miles", "water the seedlings again in 3 days") get `wait: "1 day"`, so they don't land
     in the middle of today's session.
   - **Connect steps through what they share.** If a step uses something another step prepares (a hot pan, a
     marinade, a primed wall, a soaked bean), list it in that step's `consumes` or `requires`. That guarantees the order,
     and it lets the analysis catch preparations that never get used.
   - **Say where.** If anything happens away from home (work, a store, a park, a drop-off), use `at`, and carry
     things there with `moves` and `to`. Then the analysis can check that what's needed is actually there, and
     that it stayed safe on the way.
   - **Every tool goes in `uses`,** including gloves, a torque wrench and a funnel. If it is not in `uses`, it is not on the equipment list.

4. **Save it** with `save_plan`. Read every diagnostic:
   - `error`: fix it and save again.
   - `similar-term`: you probably meant an existing resource name. Use it, or `merge_resources` if they really are the same.
   - `minted-term`: a new resource, which is fine.

5. **Add facts** the constraints depend on (`guide("facts")`). For allergies, every ingredient needs a `contains`
   fact, or the allergy check will report it as unverifiable. That is deliberate. Give every fact a real source.

6. **Analyze** with `analyze_plan` for the profile, the number of servings (`servings` scales recipes that `yields`
   servings; nothing is scaled unless you ask) and, if known, a start or finish time. Then:
   - fix ordering and resource problems
   - resolve constraint failures: swap an ingredient, choose a different recipe, or add a missing fact
   - read the rendered plan *as a first-timer would*. Would they know what to do, how long to wait, how to tell it's
     done, and what to do if it goes wrong?

7. **Rehearse it.** Read the plan as the person will (`analyze_plan` with `forPerson: true`) and walk through it in
   your head, step by step, as someone who has never done this:
   - Where would they hesitate?
   - What do they need that was never mentioned?
   - What happens between steps, while things wait?
   - What is the setting at that time and place?
   - What could hurt them?

   Fix what you find, then analyze again.

8. **Schedule it** with `schedule_plan`. Give it the profile, the number of servings, and when it happens: a `start`, a
   `finishBy` (such as lunch ready at noon), or `pins` for activities spread over days (such as a lesson at 5pm every Tue/Thu).
   The scheduled id is what the person opens. Each node's `{path}` appears in `analyze_plan` output.
   Only pass `choices` for decisions the person has actually made. Unchosen options stay visible to them as
   alternatives, with full steps. A finish time schedules work back from it, so things are ready just in time.

9. **Hand off.** Tell the person exactly what the analysis says. Include the shopping list, the equipment to confirm,
   and the **checks** listed under "Before you start, check". These are things only they can verify, such as reading a product label.
   Never say a check passed when the analysis says it could not be verified.

To swap part of a plan (for example a different main dish), save the new recipe and call `analyze_plan` with
`swaps: {"<path>": "<recipe id>"}`. The shopping list and schedule are recomputed, and you never edit the old plan.
