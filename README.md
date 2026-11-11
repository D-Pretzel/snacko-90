# J-Flight Snacko Store

The snack stand for J Flight ("Jungle Flight") of the 90th Flying Training Squadron, the Boxin' Bears. A tap-to-pay honor-system stand: customers tap an NFC tag, pick what they grabbed, and pay through Venmo in a single tap. No Square, no card reader, no monthly fees.

- **Live page:** https://d-pretzel.github.io/snacko-90/
- **Menu editor:** https://d-pretzel.github.io/snacko-90/admin.html

> **This is not the `snacko` or `snacko-89` repository.** Separate stands run the same code out of `D-Pretzel/snacko` and `D-Pretzel/snacko-89` (the 89th's A-Flight), each with its own Worker, its own GitHub token, and its own password. They are kept apart in three places, and all three have to stay right:
>
> | | this repo | `snacko-89` | `snacko` |
> | :--- | :--- | :--- | :--- |
> | `WORKER_URL` in `admin.html` | `snacko-90.petzoldavid02.workers.dev` | `snacko-89.petzoldavid02.workers.dev` | `snacko.petzoldavid02.workers.dev` |
> | `GH_REPO` in `worker/wrangler.toml` | `D-Pretzel/snacko-90` | `D-Pretzel/snacko-89` | `D-Pretzel/snacko` |
> | `PASS_KEY` in `admin.html` | `snacko90.editPass` | `snacko89.editPass` | `snacko.editPass` |
>
> The target repository lives in the Worker and is never sent by the browser, so a Worker can only write to the repository it was deployed with. That is the real safeguard. It also means a wrong `WORKER_URL` fails silently, publishing this stand's menu over another stand's with no error anywhere. `PASS_KEY` is separate because GitHub Pages serves every project from one origin and `localStorage` is scoped to the origin rather than the path; a shared key would let signing in to one editor hand the other a password that does not work there.

---

## How it works

1. A customer taps their phone on the tag stuck to the snack container.
2. Their phone opens the page, which loads the menu from `menu.json`.
3. They tap the items they took, and a running total builds itself. On a long menu there is a search box for finding an item by name.
4. They pay. Venmo is the main button and fills in the amount and a plain-English itemized note. PayPal, Cash App and Zelle get a smaller button each when the stand has a username for them; those copy the note for the customer to paste (see *Payment methods* below).

Because the tag only stores the page's URL, you never re-write a tag when prices change. You just change the menu.

The customer page has no server behind it: GitHub Pages hands out `index.html` and `menu.json`, and the browser does the rest. **Editing** the menu does depend on a small Cloudflare Worker, because something has to hold the GitHub credential. If Cloudflare is unavailable, the stand keeps selling; only editing stops.

---

## Changing the menu

Open **https://d-pretzel.github.io/snacko-90/admin.html**, enter the editor password, and change what you need. There is no GitHub account, no code, and no file to find.

You can:

- Add, rename, reorder, and delete categories and items
- Set prices, and move an item from one category to another
- Set the Venmo, PayPal, Cash App and Zelle accounts, so a change of snacko needs no code change
- Add a short description under any item ("12 oz cans")
- Put an item on sale by a percentage, with an optional end date
- Hide an item without deleting it, for when you are out of stock
- Preview the customer view before you save

Press **Save changes**. The live page catches up within about a minute. If a field is wrong, it is highlighted with the reason and nothing is sent until it is fixed. If a save fails, usually because of poor signal, every edit stays on screen, so nothing is lost; press Save again.

### Add to Home Screen

Do this on the first visit. The editor ships a `manifest.json` and an app icon, so this produces a real icon and opens without browser chrome, like an app. On iOS use Share → Add to Home Screen; on Android, the browser menu offers Install.

### Sales

A sale is stored as a percentage off, never as a second price. The discounted price is worked out when the page renders, so the original price is never lost. A sale with an end date stops applying the day after that date, judged by the phone's own clock, and the sale itself stays in the file until you clear it.

### Editing `menu.json` directly

`menu.json` is a plain data file in this repository, and it is the only place menu data lives. If the editor is ever unavailable, open `menu.json` on GitHub, click the pencil, and edit it by hand:

```json
{
  "name": "Snack-O",
  "venmoUsername": "your-venmo-username",
  "paypalHandle": "YourPayPalName",
  "cashtag": "YourCashtag",
  "zelleRecipient": "you@example.com",
  "categories": [
    {
      "label": "Drinks",
      "items": [
        { "name": "Red Bull", "price": 2.50, "description": "12 oz cans" },
        { "name": "Water", "price": 1.00, "sale": { "percentOff": 20, "until": "2026-08-01" } },
        { "name": "White Monster", "price": 3.00, "hidden": true }
      ]
    }
  ]
}
```

- `name`: what shows in the header and the browser tab.
- `venmoUsername`: the Venmo handle, without the `@`.
- `paypalHandle`: the part after `paypal.me/` in your link, with no `@` and no slashes.
- `cashtag`: the Cash App $cashtag, without the `$`.
- `zelleRecipient`: the email or US phone number the bank has registered with Zelle.
- All four payment fields are optional and an empty one hides that button, but at least one must be set. The editor and the Worker both refuse a menu with none.
- `price`: dollars, at most two decimal places.
- `description`: optional. Leave it out and the row looks exactly as it always has.
- `sale`: optional. `percentOff` is a whole number from 1 to 99; `until` is optional and formatted `YYYY-MM-DD`, and the last day counts.
- `hidden`: optional. `true` keeps the item in the file but off the customer page.

Every past version of the menu is in the repository's commit history, so a bad edit is always one revert away.

---

## Hosting

There is no build step. GitHub Pages serves this repository's root from the `main` branch, so anything committed to `main` is live at `https://d-pretzel.github.io/snacko-90/` within about a minute. The setting is under **Settings → Pages**: source **Deploy from a branch**, branch `main`, folder `/ (root)`. Confirm it is switched on for this repository. It is a per-repository setting, and having it on for `snacko` does nothing for `snacko-90`.

`index.html` and `menu.json` must stay in the repository root, since the page fetches `menu.json` as a relative path.

---

## The editor's plumbing

Full detail, including first-time setup, lives in [`worker/README.md`](worker/README.md).

The Worker is `snacko-90` on Cloudflare, serving `https://snacko-90.petzoldavid02.workers.dev`, and `admin.html` points at it. It holds two secrets: a GitHub fine-grained token scoped to **this** repository with Contents read and write, and the shared editor password. Give it a different password from the `snacko` and `snacko-89` stands. The editors share an origin, so one password across them means whoever can edit one can edit all.

The token never touches the browser. The editor only ever sends the Worker a password and a menu; the Worker checks the password, re-validates the menu against the schema, and makes the commit.

**Changing the password**, when the job changes hands or you think it got out, is one command from `worker/`. No redeploy, effective immediately:

```bash
wrangler secret put EDIT_PASSWORD
```

Type it at the prompt rather than passing it as an argument, so it stays out of your shell history. Anyone still signed in on the old password gets returned to the login screen the next time they save, with their unsaved edits intact.

**The GitHub token expires.** When it does, saving fails and the Worker says so ("GitHub rejected the credential; the access token has likely expired"). Mint a new fine-grained token with the same scope, run `wrangler secret put GH_TOKEN`, and revoke the old one. Keep the expiration date somewhere you will actually see it.

**Changing the Worker's code** means `wrangler deploy` from `worker/`. The `name` in `wrangler.toml` must match the Worker's name on the Cloudflare account exactly; if it does not, `wrangler secret put` fails with `This Worker does not exist on your account. [code: 10007]`, which is a name mismatch and not an authentication problem.

---

## Writing the NFC tag

1. Buy blank NTAG213 or NTAG215 stickers. They are inexpensive and widely available.
2. Install a free app like **NFC Tools** (iOS or Android).
3. Choose **Write → Add a record → URL**, and enter `https://d-pretzel.github.io/snacko-90/`.
4. Hold the sticker to your phone to write it, then stick it on the container.

Add a small "Tap to pay" label near the tag so customers know what it is.

---

## Testing before you launch

**Every payment account lives in `menu.json`** where the editor can reach it, so a change of snacko does not need a developer.

Send yourself a real payment of a dollar or two through every button before the tags go out. A handle that does not exist does not throw an error. Venmo opens a page that goes nowhere, or worse, to someone else with a similar handle.

Then test on **both an iPhone and an Android** before sticking tags on anything. The browser-to-Venmo handoff behaves a little differently across phones, so confirm that the pay button opens Venmo with the correct amount already filled in, going to the right person. Read the note on the Venmo screen too, not just the amount: the words should be separated by spaces, and any `+` between them means the app link is not being taken and the handoff is re-encoding the note (see *Good to know*).

Worth testing on a phone **without Venmo installed** as well. The button should fall back to the Venmo website and behave exactly as it does on a desktop, amount and note intact.

To test locally, note that `index.html` now fetches `menu.json`, and browsers block `fetch` from `file://`. Opening the file by double-clicking it will show the error state. Serve the folder instead:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000/`.

The **editor** cannot be tested this way. The Worker only accepts requests from `https://d-pretzel.github.io`, so `admin.html` served from localhost is refused. That is the origin check working as intended, not a bug. Test the editor at its live URL, or see [`worker/README.md`](worker/README.md) for temporarily pointing `ALLOWED_ORIGIN` at a local server.

Because every stand shares that origin, the origin check cannot tell this editor from the others. Whether a save lands in the right repository is decided entirely by `WORKER_URL` in `admin.html`, so verify the commit shows up under **`snacko-90`** the first time you save.

---

## Good to know

- **Both pages adapt to light and dark mode** automatically.
- **The menu has a search box once it reaches 10 items**, and none below that, where scrolling is quicker than reaching for a keyboard. It matches on item name only, ignoring case and accents, and every word you type has to appear somewhere in the name, so "bar protein" finds "Ready Protein Bar". Results filter as you type.
- **Search never touches the cart.** An item you have already added stays in the total and in the Venmo note even while the search is hiding it, and clearing the search brings every quantity back exactly as it was. The totals bar is always the truth about what is being bought, which is why it is worth a glance before paying.
- **The pay buttons stay disabled** until at least one item is added, so nobody sends a zero payment.
- **That is why the Venmo button asks "Open in Venmo?" first.** A custom scheme like `venmo://` is unverified, since any app could claim it, so iOS always confirms before leaving Safari. A *universal link*, which is an ordinary `https://` address the app's owner has registered to it, opens silently instead because iOS has already verified the domain. Which of the two fired is worth knowing whenever a hand-off starts misbehaving, and the dialog is how you tell.
- **The payment note is itemized** and written to read like a sentence, for example `J-Flight Snacko Store: 2x Cookie and Soda`, so you can see what each sale was. It is prefixed with the stand name from `menu.json`, drops the `1x` for single items, and is trimmed at Venmo's 280-character limit.
- **Sale prices are what customers are charged.** The total and the Venmo amount both use the discounted price.
- **Item names are treated as plain text.** An apostrophe, an accent, or a stray `<` in a name shows up literally and cannot break the page.
- **The Venmo link is built with `encodeURIComponent`, not `URLSearchParams`.** `URLSearchParams` encodes spaces as `+`, and Venmo prints the note exactly as given, plus signs included. If the note ever comes back full of `+`, that is why.
- **On a phone the Venmo button opens the app directly, not `venmo.com`.** Tapping the web link on a phone does not go to Venmo in one hop: it loads Venmo's page, which rebuilds the query as form-encoded data on its way into the app, and form encoding writes a space as `+`. The app then prints the note exactly as it was handed over. That is why the same note read correctly in a browser and arrived full of `+` on a phone: the mangling happened on Venmo's side of the handoff, after this page was done with it. The button now hands the app its own link (`venmo://paycharge?txn=pay&recipients=…`), which skips the rebuild, so the `%20`s survive as spaces.
- **The web link is still the button's `href`.** The app link is tried first on a phone and the web link takes over if nothing answers within about a second, so a phone without Venmo installed still gets somewhere useful, and long-press-to-copy still yields a URL that means something. Desktop never tries the app link at all.
- **Venmo's amount-prefill through links is undocumented** and has changed over the years. It works today, but if it ever breaks, the fix is on this page, not on the physical tags.

---

## Payment methods

Venmo is the main button because it is the only one of the four whose link carries both the amount and the itemized note. The others appear as a row of smaller buttons under it, one for each method with a username set. With no Venmo username, that row becomes the main row.

What each one can and cannot fill in:

| Method | Link | Amount filled in | Note filled in |
| :--- | :--- | :--- | :--- |
| Venmo | `venmo://paycharge…` on a phone, `venmo.com/…` otherwise | Yes | Yes |
| PayPal | `paypal.me/<handle>/<amount>` | In a browser; the PayPal app drops it | No: PayPal.me has no note field |
| Cash App | `cash.app/$<cashtag>/<amount>` | Yes | No: there is no note parameter |
| Zelle | None; Zelle exists only inside each bank's app | No | No |

So PayPal, Cash App and Zelle open a sheet instead of leaving the page. The tap that opens it also copies the itemized note to the clipboard, which only a tap is allowed to do. The sheet shows who to pay, the amount, and the note, each with its own Copy button and a line saying where to paste. PayPal and Cash App then have an **Open PayPal** / **Open Cash App** button; Zelle tells the customer to open their bank's app. The standalone Zelle app was shut down in 2025, so there is no app to link to or send people to the App Store for.

**Why PayPal is the free link and not a checkout link.** A PayPal checkout link (`paypal.com/cgi-bin/webscr?cmd=_xclick…`) would carry both the amount and the note, even in the app. It was turned down because it is a goods-and-services payment, which costs 3.49% + $0.49 per sale (more than a 25¢ water), and because PayPal has deprecated that link type. Two other fixes for the dropped amount were tried and failed: `paypal.com/paypalme/…` is claimed by the app just the same, and a currency suffix (`/5.00USD`) makes the app drop the amount entirely.

**The Worker must know every field.** Its validator rebuilds the menu from fields it recognizes and drops the rest. A Worker older than these fields will silently erase `cashtag` and `zelleRecipient` on the next save, and it will refuse any menu with no Venmo username. After changing the payment fields in code, run `wrangler deploy` from `worker/` before anyone saves from the editor.

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The customer page. Renders the menu, totals, and the hand-off to each payment method. |
| `menu.json` | The menu itself. The only place menu data lives. |
| `admin.html` | Password-protected menu editor. |
| `manifest.json` | Makes the editor installable to a phone home screen. |
| `worker/wrangler.toml` | Worker configuration: name, `GH_REPO`, `ALLOWED_ORIGIN`. Secrets are never in here. |
| `worker/src/index.js` | The Worker itself. Checks the password, re-validates the menu, commits `menu.json`. |
| `worker/README.md` | Worker setup, secret rotation, and local testing. |
| `J-Flight-Patch.png` | J Flight "Welcome to the Jungle" patch. Source art for every image below. Only 381×386, so the 512px icon is upscaled slightly; drop in a larger copy and regenerate if you get one. |
| `jflight-logo.png` | The J Flight patch shown in the header of both pages. White field knocked out to transparent, so it sits on the page in light and dark mode alike. |
| `favicon-64.png` | Browser-tab icon for both pages. Cropped tighter than the app icons so it still reads at the 16px a tab actually draws. |
| `icon-192.png`, `icon-512.png` | Home-screen icons for the editor, and the `apple-touch-icon` for both pages. |
| `venmo-logo.png` | Venmo mark shown on the pay button. |
| `paypal-mark.png` | PayPal monogram, cropped square from `paypal_logo.png`. Not loaded since the secondary payment buttons went text-only. |
| `paypal_logo.png` | Original full PayPal lockup. Kept as the source art; the page does not load it. |
| `snacko-editor-spec.md` | The build specification the editor was written from. |
| `.gitignore` | Keeps wrangler's local cache, which holds the Cloudflare account id, out of the repository. |
| `README.md` | This document. |

The 89th FTS patch inherited from `snacko-89` has been replaced by the J Flight patch, which is now the only emblem on either page. The 89th art remains in history before this commit.
