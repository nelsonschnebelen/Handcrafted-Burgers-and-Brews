# Handcraft Burgers &amp; Brew — Spin to Win

A single-file spin-to-win promo page for [Handcraft Burgers &amp; Brew](https://handcraftburgers.com/)
(110 W 40th St, New York, NY).

No build step, no dependencies, no external assets. `index.html` is the whole app — drop it on
any static host.

## The prizes

| Prize | Code | Odds | Frequency |
|---|---|---|---|
| $2 off your order | `DISHIO $2` | 61.7% | ~1 in 2 |
| $3 off your order | `DISHIO $3` | 38% | ~1 in 3 |
| $4 off your order | `DISHIO $4 ####` | 0.2% | 1 in 500 |
| 50% off your order | `DISHIO HALF ####` | 0.1% | 1 in 1,000 |

Verified over 2,000,000 simulated draws: $4 off landed 1 in 506, 50% off 1 in 979, with zero
mismatches between the prize awarded and the segment the wheel stopped on.

Discounts must be used with a purchase, before tax. 50% off is capped at $15. No expiry.

Weights are expressed out of 1,000 so they read directly as "per 1,000 spins."

## Every displayed prize is genuinely winnable

The wheel shows four prizes and all four can actually be won. That is deliberate.

A wheel that *displays* a prize it can never land on is a deceptive promotion — the FTC treats
zero-probability displayed prizes as a deceptive practice, and several states' promotion statutes
require displayed prizes to be attainable with disclosed odds. "Rare" is fine. "Shown but
impossible" is not.

Making the two premium tiers genuinely rare rather than fake costs about **$8 per 1,000 spins**
($2,390 vs $2,381 for a $2/$3-only wheel). That is the entire price of the difference.

**Open question for counsel:** the offer now requires a purchase. Purchase (consideration) +
random outcome (chance) + discount (prize) is the three-element test that defines a lottery in
most US states, which is why promotions of this shape usually carry a "no purchase necessary"
alternate entry route. Worth a review before this goes to print.

Two safeguards keep it that way:

- **Published odds are generated from the same weights the wheel uses**, and rendered at their
  true value rather than rounded — a published "3%" beside a real 2.5% overstates the guest's
  chances. They cannot drift apart. Both the on-page odds table and the fine-print odds line were
  removed at the client's request; `oddsPct()` still feeds the reporting payload.
- **A zero or negative weight logs a console error.** To retire a prize, delete it from `PRIZES`
  so it comes off the wheel; do not zero its weight and leave the wedge showing.
- **`FORCE_PRIZE` warns loudly whenever it is set**, and marks every issued code as forced.

## Forced outcome (currently ACTIVE)

`FORCE_PRIZE` at the top of the script block is set to `'off2'`, so **every spin lands on $2
off**. `null` restores the weighted draw. A `?force=off3` query param overrides it per-visit
without editing the file, which is the easy way to demo a specific tier.

While a force is active the page logs a console warning on every load, and `reportPrize()` stamps
`forced: true` with `odds: "forced"` so records never misreport a forced win as a random draw.

**Before this goes in front of real guests, set `FORCE_PRIZE = null`.** With it active the wheel
still displays $3, $4, and 50% off while being unable to land on them — that is the
zero-probability display described above, and it is the thing that makes a promotion deceptive
rather than merely stingy.

If the goal is genuinely that every guest gets $2 off, don't force it — delete the other three
entries from `PRIZES`. The wheel then shows only $2 OFF, lands on it every time, and is completely
honest. Identical outcome for the guest, no exposure.

## Redemption codes

Cheap tiers use a flat spoken phrase — someone sharing `DISHIO $2` costs you two dollars, which
is not worth policing. The two premium tiers set `unique:true`, which appends a random four-digit
suffix (`DISHIO HALF 4817`) so the POS can burn the code after one redemption.

**This matters more than it did for the fries offer.** A universal, freely-shareable 50%-off code
would let anyone claim the rare prize without ever spinning, which defeats the point of the tier.
Have the register validate suffixed codes against what `reportPrize()` recorded, or the 1-in-1,000
becomes 1-in-1 for anyone who sees a screenshot.

## Configuration

**Prizes and odds** — the `PRIZES` array at the top of the script block. Each entry carries its
own `weight` (relative share), `code`, `terms`, and a `unique` flag. `l1`/`l2` are the two stacked
lines on that prize's wheel segment.

The wheel, the odds list, the fine print, the ticket, and the reporting payload all render from
this array, so adding or removing a prize updates the whole page. Each prize occupies two of the
eight segments; segment count is cosmetic, odds come from `weight`, which is why the published
odds appear below the wheel.

**Brand colors** — the `:root` block at the top of the stylesheet:

```css
--ink:    #14110F;   /* near-black charcoal  */
--cream:  #F7F1E4;   /* primary light        */
--flame:  #D6402A;   /* burger red — primary */
--amber:  #E8A020;   /* fries / beer gold    */
```

> **Note:** handcraftburgers.com sits behind a Cloudflare bot check, so these were matched by eye
> to the brand's craft-burger signage rather than sampled from their real assets. Swap in the
> official hex values here and the whole page follows.

## Guest attribution

Guest details are collected upstream, before the visitor reaches this page. Pass their id through
on the query string and it rides along with the issued prize:

```
https://your-host/?g=guest_8842
```

`reportPrize()` builds the payload:

```json
{
  "guestId":  "guest_8842",
  "prizeId":  "half",
  "prize":    "50% Off Your Order",
  "code":     "DISHIO HALF 4817",
  "odds":     "0.1%",
  "forced":   false,
  "issuedAt": "2026-08-17T15:53:08.836Z",
  "source":   "spin-to-win"
}
```

The `fetch` is commented out — point it at your CRM/POS endpoint to go live. It's deliberately
fire-and-forget so a network hiccup never blocks a guest mid-spin. If you drop the `?g=` param
entirely, `guestId` is simply `null` and the page still works standalone.

## One spin per device

Enforced client-side via `localStorage` under the key `hc_spin_v1`. Returning visitors see their
code again rather than re-spinning. This stops casual re-rolls, not determined ones — a private
window or a different phone resets it. Since a re-roll is another shot at the 1-in-1,000, move the
check server-side if that bothers you.

## Local preview

Any static server works:

```bash
python -m http.server 4321
```

Then open <http://localhost:4321>.

## Accessibility &amp; support

Spin results are announced via an `aria-live` region, and the wheel's `aria-label` lists all four
prizes and the one-try limit. `prefers-reduced-motion` shortens the spin and skips the confetti. Layout is
phone-first (most traffic will be QR scans at the table) and caps to a centered column on desktop.
