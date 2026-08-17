# Handcraft Burgers &amp; Brew — Spin to Win

A single-file spin-to-win promo page for [Handcraft Burgers &amp; Brew](https://handcraftburgers.com/)
(110 W 40th St, New York, NY). Guests give the wheel one turn and always walk away with
something to redeem at the register.

No build step, no dependencies, no external assets. `index.html` is the whole app — drop it on
any static host.

## Prizes

| Prize | Code the guest says | Odds |
|---|---|---|
| Free French Fries | `DISHIO FRIES` | 12 |
| Free Signature House Sauce | `DISHIO SAUCE` | 33 |
| Free Fountain Soda | `DISHIO DRINK` | 22 |
| 10% Off Your Next Order | `DISHIO 10% OFF` | 33 |

Odds are relative weights, not percentages — they're normalized at draw time, so you can use any
scale you like. Free fries is the priciest giveaway, so it's weighted rarest.

There is no losing wedge. Every guest wins something.

## Configuration

Everything you'd normally want to change lives in two places near the top of `index.html`.

**Prizes and odds** — the `PRIZES` array in the script block. Edit a `weight` to change how often
a prize hits; edit a `code` to change the redemption phrase. The wheel, the legend beneath it, and
the prize ticket all render from this array, so adding or removing a prize updates the whole page.

Segment count is `PRIZES.length * 2` — each prize appears twice, directly opposite itself, so the
wheel reads full rather than sparse.

**Brand colors** — the `:root` block at the top of the stylesheet. Every color on the page is a
variable there:

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
  "prizeId":  "sauce",
  "prize":    "Free Signature House Sauce",
  "code":     "DISHIO SAUCE",
  "expires":  "August 31, 2026",
  "issuedAt": "2026-08-17T15:53:08.836Z",
  "source":   "spin-to-win"
}
```

The `fetch` is commented out — point it at your CRM/POS endpoint to go live. It's deliberately
fire-and-forget so a network hiccup never blocks a guest mid-spin. If you drop the `?g=` param
entirely, `guestId` is simply `null` and the page still works standalone.

## Redemption codes are shared, not unique

Every guest who wins the sauce gets the same `DISHIO SAUCE` phrase. That's intentional — it's
fast to say at the counter and needs no lookup — but it means the code can be screenshotted and
passed around, and the POS can't tell two redemptions apart. The 14-day expiry and the one-spin
lockout are the only limits.

If you need per-guest validation later, have `makeCode()` append a short random suffix and check
it against whatever `reportPrize()` recorded.

## One spin per guest

Enforced client-side via `localStorage` under the key `hc_spin_v1`. Returning visitors see the
prize they already won instead of re-spinning. This stops casual re-rolls, not determined ones —
a private window or a different phone resets it. Move the check server-side if the giveaway is
expensive enough to be worth gaming.

## Local preview

Any static server works:

```bash
python -m http.server 4321
```

Then open <http://localhost:4321>.

## Accessibility &amp; support

Spin results are announced via an `aria-live` region. `prefers-reduced-motion` shortens the spin
and skips the confetti. Layout is phone-first (most traffic will be QR scans at the table) and
caps to a centered column on desktop.
