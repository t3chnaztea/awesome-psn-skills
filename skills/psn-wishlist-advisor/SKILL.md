---
name: psn-wishlist-advisor
description: >-
  Use when deciding what to actually buy off a PlayStation Store wishlist:
  "rank my wishlist", "what should I buy next", "is this sale worth it", "what
  on my wishlist will I actually finish", "clean up my wishlist", "should I grab
  X while it's discounted". Reads the local wishlist export plus the taste
  profile and ranks each item by play-and-finish likelihood (not hype), flags
  items that match the player's avoid-signals, makes buy-next and
  quietly-remove calls, and factors current discounts into timing. Not for
  recommending from games already owned (psn-taste-profile), triaging the
  unfinished backlog (psn-backlog-triage), or producing the export
  (psn-export).
compatibility: Needs wishlist.json from `psnstats --wishlist` and, for taste matching, preferences.json from `--analyze`, both in ./psn-export/.
---

# PSN Wishlist Advisor

A wishlist is a pile of wants, sorted by nothing. This skill sorts it by the
only thing that matters after purchase: **will they actually play and finish
it.** Hype and discounts are inputs, not the ranking. The output is a buy-next
call, a quietly-remove list, and timing advice tied to real prices.

<data_source>
Two files, both in `./psn-export/` (see `psn-export`):

- **`wishlist.json`** from `psnstats --wishlist` — the items and their live
  store prices. Shape is `{"wishlist": [ ... ]}`, each item:
  - `name`, `product_id`
  - `kind` — Sony's type: **`Product`** (a purchasable SKU with price and
    platforms) or **`Concept`** (an unreleased title with no SKU: empty
    platforms/classification/price). A Concept can't be bought yet — treat it
    as a watch-item, never a buy-next.
  - `platforms` (e.g. `["PS5","PS4"]`), `classification` (e.g. `FULL_GAME`)
  - `base_price`, `discounted_price`, `discount_text` — Sony's **localized
    display strings** kept verbatim (`"$19.99"`, `"-25%"`), empty for Concepts.
  - `box_art_url`
- **`preferences.json`** from `--analyze` — the taste profile, for matching
  wishlist items against what this player actually engages with.

Never fetch the store yourself and never invent a title or a price. Prices are
whatever Sony returned at export time: if `wishlist.json` is more than a few
days old, note that a discount may have changed and offer a refresh via
`psn-export`. If the wishlist export is missing, route there first.

If Sony rotated the persisted wishlist query, `--wishlist` fails at export time
(a 400) — that's a `psn-export` problem, covered in that skill.
</data_source>

<the_ranking>
Rank each buyable `Product` by **play-and-finish likelihood**, built from the
taste profile, not from review scores:

1. **Fit to the palette.** Does the item resemble the player's high-enjoyment
   lane — session shape (`preferred_session_style`), commitment style, the
   genres/difficulty of their `top_games`? A roguelite for a proven
   snackable-roguelite player ranks high; a 100-hour JRPG for someone whose
   `friction_tolerance` is 17.9 and who abandons long games ranks low, however
   acclaimed.
2. **Avoid-signal matches.** Check the profile's `agent_features.avoid_signals`
   (e.g. "drawn-out mid-game", "long-session requirement"). An item that walks
   straight into a documented avoid-signal is a likely non-finish — flag it
   explicitly, even if it's cheap or hyped.
3. **Positive-signal matches.** Items that hit `positive_signals` (short-session
   loops, completable games, a favored platform) are the safe buys.
4. **Then, and only then, price.** A strong-fit game at full price beats a
   bad-fit game at 80% off. Discounts break ties and set timing; they don't
   make a game the player won't finish worth buying.
</the_ranking>

<the_calls>
Produce three things:

- **Buy next (1-3 items).** The best fit-to-taste items, with the signal that
  earns the rank and the current price. If a buy-next item is discounted, say
  "and it's on sale, so now"; if it's full price but a strong fit, say "worth
  full price" or "wait for a sale if you're patient" per how strong the fit is.
- **Quietly remove.** Items that clash with the palette or hit an avoid-signal
  and have sat unbought. Be honest: "this has been on the list a while and it's
  a bad match for how you actually play; let it go." Removing noise makes the
  list usable.
- **Timing notes.** For fits that are currently full price, say what to wait
  for; for strong fits that are discounted right now, say to move. Cite the
  actual `base_price` / `discounted_price` / `discount_text` strings. For a
  `Concept`, note it's not purchasable yet: keep watching, no action.
</the_calls>

<worked_example>
Say the wishlist holds a short, high-replay roguelite at `"$19.99"` marked
`"-25%"` → `"$14.99"`, and the profile shows `preferred_session_style` leaning
snackable with a "short-session loops" positive_signal and a top-tier
roguelite already at high enjoyment. That's the **buy-next**: it hits the
positive-signal, matches the proven lane, and it's discounted right now, so the
timing is also right — call it.

Now say the list also holds a 90-hour narrative RPG at full price, on a profile
whose `avoid_signals` include "drawn-out mid-game" and whose `friction_tolerance`
is low. However well-reviewed, that's a likely non-finish: **flag it, lean
remove**, and if they insist, tell them to wait for a deep discount so a
probable DNF costs less. A wishlisted `Concept` with empty price/platforms is
an unreleased watch-item: note it, take no buy action.
</worked_example>

<constraints>
- Rank by likelihood to play and finish, from the taste profile. Never rank by
  review reputation or discount depth alone.
- Price is a tiebreaker and a timing input, not the ranking. Don't recommend a
  bad-fit game because it's cheap.
- Only `Product` items are buyable. Never issue a buy-next for a `Concept`
  (no SKU, no price) — it's a watch-item.
- Quote prices verbatim from the file (`base_price`, `discounted_price`,
  `discount_text`); they're localized display strings. Don't compute or convert
  them, and flag if the export is old enough that a sale may have moved.
- Flag every avoid-signal match explicitly, even for hyped or cheap items.
- Commit to buy-next and remove calls. A wishlist re-sorted with no verdicts
  helped no one.
- Never invent a title, platform, or price. If it isn't in `wishlist.json`, it
  isn't on the list. If the file is missing, route to `psn-export`.
</constraints>

<tone>
Direct and a little protective of the player's time and money. You're the
friend who says "you're never going to finish that one, but grab this while
it's cheap." No hype, no em dashes (use colons or commas).
</tone>
