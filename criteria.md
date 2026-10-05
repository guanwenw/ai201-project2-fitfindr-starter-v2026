# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:** Keyword overlap is the sole matching mechanism;
phrasings that fall just outside the overlap threshold will miss even
when a human would accept the listing. 4 of 5 allows for one such miss
without treating the search as broken.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:** This criterion guards the branch itself. Failure
means the loop is a fixed sequence of calls, not an agent. The check
is binary — `suggest_outfit` was called or it was not — so no
tolerance is warranted.

---

## 3. State carries the item through

The item `search_listings` returns is the same item `suggest_outfit`
receives. Across 5 queries that match at least one listing, in all 5
`session["selected_item"]["id"]` equals the `id` of the first listing
in `session["search_results"]`.

**Why this target:** This criterion detects a loop that re-searches or
re-parses rather than reading what the previous tool returned. The id
comparison is exact, so any deviation is a genuine defect and no
tolerance applies.

---

## 4. Fit card is specific and varies

The fit card is non-empty and mentions the item's price and platform
at least once. Across 5 runs of `create_fit_card` on the same item, all
5 outputs mention both, and at least 3 of the 5 differ from each other
word-for-word.

**Why this target:** The fit card is model-generated, so identical
wording across runs would indicate a cache or temperature fault rather
than intended determinism. 3 of 5 permits occasional phrasing
collisions without masking a frozen model. The price and platform
requirement is separate: it distinguishes a real post from a generic
description, and it is objectively checkable.

---

## 5. Agent responds within 30 seconds

Given a query that matches at least one listing, the agent returns a
fit card within 30 seconds — in at least 4 of 5 tries.

**Why this target:** Latency is not covered by the other four criteria.
The agent makes three tool calls, two of which reach the model; a
pacing misconfiguration or a retry loop can push a single query into
minutes. 30 seconds is generous for two paced model calls, and 4 of 5
tolerates one slow run without flagging the system as broken.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
