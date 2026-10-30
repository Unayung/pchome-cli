---
name: pchome
description: Use when searching PChome 24h購物 (24h.pchome.com.tw) products, checking prices/stock/specs, monitoring prices, or viewing/adding/removing items in the user's PChome cart (購物車). No checkout.
---

# pchome — PChome 24h購物 from a shell (no checkout)

`~/.local/bin/pchome` → `~/Projects/pchome-cli/pchome`. Stdlib Python, plain HTTPS; Edge on
CDP 9222 is only a read fallback and the opt-in `pchome login --from-browser`. Run `pchome doctor` first.

Read (no login):
- `pchome search <q> --json [--limit N] [--sort relevance|sales|price|price-desc|new] [--min/--max TWD]`
  → id, title, price, orig_price, discount (折), rating/reviews, url
- `pchome product <url|id> --json` → price, orig_price, in_stock, max_qty, shipping
  (24h到貨 / 廠商出貨), promo, `specs[]` with ids (`…-003`) and per-spec stock
- `pchome price <id> <id> … --json` → batch price/stock for monitoring
- Product ids look like `DAAG8M-A900H3TL3`; URLs `24h.pchome.com.tw/prod/<id>` work too.

Login (never ask for or type passwords/OTP, and never ask the user to paste cookies into chat):
- Tell the user: DevTools > Network > click any 24h.pchome.com.tw request > copy the `Cookie:`
  request header to the clipboard, then run `pchome login` themselves (or pipe it in on stdin).
  No usable Cookie header = exit 2 with those steps.
- `pchome whoami` proves the saved session over HTTP; exit 3 = not signed in → `pchome login`.
- `login --from-browser` is opt-in only (Edge on CDP 9222; waits up to `--wait 300`s for the USER
  to sign in in a background tab). `--from-clipboard` / `--cookie-stdin` also work.

Cart (writes need explicit user approval):
1. `pchome cart list --json` — lines with `line`, `id`, `qty`, `price`, `subtotal`, `total`
2. `pchome cart add <id> [--spec 38] [--qty N]` (dry run) → show the user → add `--yes`
3. `pchome cart qty <line> N --yes`, `pchome cart rm <line|id> --yes`
Each write re-reads the cart and reports the verified line; trust that, not the POST.

Pitfalls:
- prodapi path-style URLs answer an empty 200 without `&_callback=` (CLI adds it; matters for `raw`).
- Signing in merges any guest cart into the member cart, so items added while logged out show up.
- Specs: `cart add` without `--spec` on a spec product exits 2 and lists the choices.
- No checkout/payment endpoints exist in the CLI by design; never try to place orders.
