# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

FitFindr is an agent that turns a plain-language thrift query into a
fit card. It searches listings, suggests an outfit using the user's
wardrobe, and writes a caption. If the search comes back empty it
stops and says what to change, rather than calling the next tool with
nothing.

**Stretch feature:** A fourth tool, `compare_prices(new_item)`, which
returns up to three cheaper alternatives in the same category as the
selected item. The loop calls it after choosing the item and stores the
result in `session["alternatives"]`.

---

## Tool Inventory

### `search_listings(description, size, max_price)`

**What it does:** Searches the listings file for items matching a
description, optionally filtered by size and a maximum price, and
returns the best matches first.

- **Inputs:**
  - `description` (str): keywords describing what the user wants.
  - `size` (str or None): size to filter by, or None to skip.
  - `max_price` (float or None): price ceiling (inclusive), or None to
    skip.
- **Returns:** A list of listing dicts, best match first, at most
  `config.SEARCH_RESULT_LIMIT` of them. Each dict has all 11 fields
  (`id`, `title`, `description`, `category`, `style_tags`, `size`,
  `condition`, `price`, `colors`, `brand`, `platform`).
- **When it has nothing:** Returns an empty list `[]`. Not `None`, not
  an exception.

### `suggest_outfit(new_item, wardrobe)`

**What it does:** Given a thrifted item and the user's wardrobe,
suggests one or two outfits that combine the new item with pieces the
user already owns.

- **Inputs:**
  - `new_item` (dict): a listing dict — the item the user is
    considering.
  - `wardrobe` (dict): a wardrobe dict with an `items` key holding a
    list. May be empty (`{"items": []}`).
- **Returns:** A non-empty string with outfit suggestions. When the
  wardrobe has items, the string names specific pieces the user owns.
  When empty, it returns general styling advice for the new item.
- **When it has nothing:** With `wardrobe["items"] == []`, returns
  general styling advice rather than raising or returning an empty
  string.

### `create_fit_card(outfit, new_item)`

**What it does:** Writes a short caption someone would actually post
about the find, in a voice that reads like a real post rather than a
product description.

- **Inputs:**
  - `outfit` (str): the outfit suggestion string from
    `suggest_outfit()`.
  - `new_item` (dict): the listing dict for the item.
- **Returns:** A two-to-four sentence caption that mentions the item,
  its price, and its platform once each, and is specific about the
  vibe.
- **When it has nothing:** If `outfit` is empty or whitespace-only,
  returns a descriptive message rather than raising.

### `compare_prices(new_item)` — STRETCH

**What it does:** Looks through the listings for cheaper items in the
same category as the selected item, so the user can see alternatives
before committing.

- **Inputs:**
  - `new_item` (dict): a listing dict — the item the user is
    considering.
- **Returns:** A list of up to 3 listing dicts from the same category,
  all cheaper than `new_item`, sorted cheapest first.
- **When it has nothing:** Returns an empty list `[]`.

---

## Planning Loop

**Branch rule:** If `search_listings` returns `[]`, set `session["error"]`
and stop. Otherwise select the first result, check for cheaper
alternatives (stretch), then call `suggest_outfit` and `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex. `under $N` → `max_price`, `size X` → `size`,
remainder → `description`.

**What moves through the session:** `query` → `parsed` → `search_results` →
`selected_item` → `alternatives` (stretch) → `outfit_suggestion` →
`fit_card`. On empty search: `error` set, later fields `None`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask '...'

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]

```

```
$ python -c "from tools import suggest_outfit; ..."

% python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

Here are two distinct outfit combinations using the new Vintage Levi's 501 Jeans and pieces from your existing wardrobe:

### Outfit 1: Casual & Layered (Streetwear Vibe)
This look plays with proportions by pairing the structured, medium-wash 501s with relaxed layers, adding contrast and texture with black accessories.

*   **New Item:** Vintage Levi's 501 Jeans (Medium Wash)
*   **Top:** White ribbed tank top + Oversized grey crewneck sweatshirt (layer the sweatshirt over the tank, or drape it over your shoulders)
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

### Outfit 2: Edgy & Defined (Off-Duty Cool)
This combination tucks in a fitted top to highlight the high waist of the 501s, pulling in the waist with a belt and grounding the look with heavier footwear and outerwear.

*   **New Item:** Vintage Levi's 501 Jeans (Medium Wash)
*   **Top:** Black cropped zip hoodie
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt + Black crossbody bag

```

```
$ python -c "from tools import create_fit_card; ..."

% AI201_CACHE=0 python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

AI201_CACHE=0 python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

AI201_CACHE=0 python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Nothing beats the fit of a truly broken-in pair of vintage Levi's 501 jeans, especially paired with fresh white sneakers for that effortless everyday look. Scoring these medium wash beauties for just $38.00 on Depop was an absolute steal. Grab them before I change my mind and keep them forever!
Nothing beats the perfect vintage fit, especially when it’s a classic pair of Levi's 501 jeans in that ideal medium wash. Paired with my go-to white sneakers, this is literally my everyday uniform. Snagged these for just $38.0 over on depop and I am never taking them off!
Nothing beats the fit of broken-in denim, and these vintage Levi's 501 jeans in the absolute best medium wash are living proof. Paired with my go-to white sneakers, this is my ultimate effortless weekend uniform. Snagged them for just $38.0 on depop before someone else beat me to it!

```

```
% python -c "from tools import compare_prices; from utils.data_loader import load_listings; print(compare_prices(load_listings()[0]))"

[{'id': 'lst_026', 'title': 'Biker Shorts — Black, Shiny', 'description': 'Shiny black biker shorts in a slightly glossy material. High-waisted. Y2K-coded. Can be styled athletic or fashion.', 'category': 'bottoms', 'style_tags': ['y2k', 'athletic', 'streetwear', '2000s'], 'size': 'S', 'condition': 'excellent', 'price': 14.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_016', 'title': 'High-Waisted Denim Shorts — Cutoff', 'description': "DIY cutoff denim shorts from Levi's 501s. Raw hem, slightly frayed. High-waisted. Perfect summer length.", 'category': 'bottoms', 'style_tags': ['vintage', 'denim', 'summer', 'classic'], 'size': 'W27', 'condition': 'good', 'price': 24.0, 'colors': ['light blue', 'blue'], 'brand': "Levi's", 'platform': 'poshmark'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}]

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
