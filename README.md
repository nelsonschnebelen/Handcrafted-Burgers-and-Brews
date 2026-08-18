# Handcraft Burgers &amp; Brew — Spin to Win

A single-file spin-to-win promo page for [Handcraft Burgers &amp; Brew](https://handcraftburgers.com/)
(110 W 40th St, New York, NY).

No build step, no dependencies, no external assets. `index.html` is the whole app — drop it on
any static host.

## The offer

**Free hand-cut fries with any order of $25 or more.** Universal code, no expiry:

```
DISHIO FRIES
```

## This is a guaranteed-win wheel, on purpose

All eight segments are the same prize. That is a deliberate design decision, not an oversight.

A wheel that *displays* four prizes but always lands on one is a rigged wheel. Guests compare
results, and a promotion that shows odds it doesn't honor invites real trouble — FTC deceptive-
practice rules and state promotion law both care about this. So the wheel shows only what you can
actually win. The spin stays fun and tactile; nobody is told they had a shot at something they
didn't.

The economics work because the **$25 minimum** caps the cost, not the code's secrecy. That's why
a universal, non-expiring, freely-shareable code is safe here: the worst case is someone spends
$25 and gets fries, which is the deal you're offering anyway.

## Configuration

**The prize** — the `PRIZE` object at the top of the script block:

```js
var PRIZE = {
  id:'fries', l1:'FREE', l2:'FRIES',
  name:'Free French Fries',
  code:'DISHIO FRIES',
  minSpend: 25
};
```

`l1`/`l2` are the two stacked lines on each wheel segment. `minSpend` feeds the ticket condition
line, the terms, and the reporting payload — change the number in one place and it updates
everywhere. Segment count is `SEG_COUNT` (8), alternating red/charcoal.

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

### Going back to a multi-prize wheel

If you ever want real variable prizes again: restore an array of prizes with `weight` values, add
a weighted pick, and have `rotationFor()` target that prize's segments instead of a random one.
The wheel, terms, and ticket all render from the config, so the rest follows. Just keep the
displayed segments honest about what's actually winnable.

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
  "prizeId":  "fries",
  "prize":    "Free French Fries",
  "code":     "DISHIO FRIES",
  "minSpend": 25,
  "issuedAt": "2026-08-17T15:53:08.836Z",
  "source":   "spin-to-win"
}
```

The `fetch` is commented out — point it at your CRM/POS endpoint to go live. It's deliberately
fire-and-forget so a network hiccup never blocks a guest mid-spin. If you drop the `?g=` param
entirely, `guestId` is simply `null` and the page still works standalone.

## One spin per device

Enforced client-side via `localStorage` under the key `hc_spin_v1`. Returning visitors see their
code again rather than re-spinning. With a universal code this is presentation rather than
enforcement — the real limit is the $25 minimum and the one-per-visit term at the register.

## Local preview

Any static server works:

```bash
python -m http.server 4321
```

Then open <http://localhost:4321>.

## Accessibility &amp; support

Spin results are announced via an `aria-live` region, and the wheel's `aria-label` states the
guaranteed prize. `prefers-reduced-motion` shortens the spin and skips the confetti. Layout is
phone-first (most traffic will be QR scans at the table) and caps to a centered column on desktop.
