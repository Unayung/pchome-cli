# pchome-cli

Agent-native CLI for PChome 24h購物 (24h.pchome.com.tw, TWD). Zero dependencies (Python 3.8+ stdlib).
Search, product details with specs, price checks, and your 購物車 (list / add / rm / qty). **No checkout.**

Portable: every call is plain HTTPS to the same JSON/JSONP endpoints the web app uses (measured
2026-10-29, none bot-walled). If a read ever gets a 403/HTML reply it is retried in a **background**
tab of a CDP browser (`PCHOME_CDP`, default `127.0.0.1:9222`); `--via http|browser` forces one.

```
pchome doctor                                  # APIs, browser, login, cart
pchome search 衛生紙 [--limit 20] [--sort relevance|sales|price|price-desc|new]
                    [--min 300 --max 500] [--page 2]
pchome product <url|id>                        # alias: item. price, list price, 折, stock,
                                               # 24h到貨 / 廠商出貨, promo text, specs + ids
pchome price <url|id> [<url|id> ...]           # quick price/stock check (monitoring)
pchome raw 'prod/button&id=DAAG8M-A900H3TL3&fields=Id,Price'   # read-only GET passthrough
```

## Login (no password)

PChome signs in with password + reCAPTCHA (+ OTP), so the CLI never takes credentials.

```
pchome login                    # = --from-browser: copy the session from Edge on CDP 9222.
                                #   Not signed in there? opens the login page in a background tab
                                #   and waits (--wait 300) until you sign in yourself
pchome login --from-clipboard   # copy DevTools' "Cookie" request header first
pchome login --cookie-stdin     # or paste / pipe it
pchome whoami                   # verifies the SAVED session over plain HTTP (email, mobile, P幣)
pchome logout
```

The session (the `ECWEBSESS` cookie is the one that matters) is verified with
`member/v2/member/islogin` + `member/id` before saving to `~/.config/pchome/state.json` (0600,
`PCHOME_HOME` moves it) or the OS keyring with `--keyring`. `PCHOME_COOKIE` overrides both.

## Cart (購物車)

```
pchome cart list                                  # lines, spec, qty, price, subtotal, total
pchome cart add <url|id> [--qty 2] [--spec 38] --yes
pchome cart qty <line#|id> 3 --yes
pchome cart rm  <line#|id> --yes
```

Without `--yes` add/qty/rm only print what they would do. After every write the cart is re-read
and the change confirmed (`verified in cart: line 4 … qty 0 -> 1`); a reply without a visible
change is an error. Products with specs (size/color) need `--spec <text or id>`; `product` lists
them, and sold-out specs are refused. Cart commands always use the saved session, never the
browser's. A guest cart (no login) exists too; PChome merges it into the member cart on sign-in.

Every command takes `--json` → `{ok, data|error}`. Exit codes: 0 ok, 1 api, 2 usage, 3 auth,
4 net/blocked.

No checkout, payment or order placement by design. Unofficial private API — may break anytime.
