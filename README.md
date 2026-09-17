# navigosurg-hosting-poc

The NavigoSurg marketing site: a single static page, no framework, no runtime
dependencies. Tailwind CSS is compiled ahead of time, so the host only ever
serves `index.html` and `styles.css`.

Sections are self-contained and each carries its own heading block, so any one
of them can be lifted into its own route later without restructuring. The
section-to-route map is in the comment at the top of `index.html`.

## Editing styles

`styles.css` is **generated** — do not edit it. Author styles in
`src/input.css`, then rebuild:

```bash
npm install
npm run build
```

Use `npm run dev` to rebuild automatically while editing.

The compiled `styles.css` is committed so a plain static host can serve this
repo with no build step. Rebuild and commit it alongside any `src/input.css`
change.

## Previewing locally

Open over HTTP rather than `file://`, so the relative stylesheet link resolves:

```bash
python -m http.server 8080
```

Then visit http://localhost:8080.

## Theming

The page **follows the OS colour-scheme preference** by default, and offers a
toggle (top right) to override it. An explicit choice wins over the OS setting
from then on, and persists under the `theme` localStorage key.

A blocking inline script in `<head>` — before the stylesheet link, so before
first paint — reads the stored choice, falls back to
`matchMedia('(prefers-color-scheme: dark)')`, and adds `.dark` to `<html>`.
Running it early is what avoids a flash of the wrong theme; keep it inline and
keep it first.

With JavaScript disabled the page renders light, since nothing adds the class.
That is the trade-off for honouring the OS preference: `dark:` variants are
class-driven rather than `prefers-color-scheme`-driven (see `@custom-variant
dark` in `src/input.css`), which is what lets the toggle override the OS at all.

Colours are semantic CSS variables (`--bg`, `--surface`, `--text`, `--accent`,
…) declared once for `:root` and once for `.dark`, then mapped into Tailwind via
`@theme inline`. Add a colour by adding a token pair, not a `dark:` utility.
Two conventions worth preserving: light mode carries elevation as a soft shadow
(`--elev`) while dark mode uses a raised surface plus a hairline border and no
shadow at all; and `color-scheme` is set per theme so native scrollbars and form
controls follow along.

## Deploying

Only `index.html` and `styles.css` are served. `.assetsignore` excludes
everything else from the Cloudflare Workers asset upload — importantly
`node_modules`, which the platform creates during its own `npm install` and
which otherwise fails the build on oversized binaries
(`node_modules/workerd/bin/workerd`).

Add any new build-time-only file to `.assetsignore`.

## Build stamp

No build stamp is visible on the page — the footer is the company footer now.
The identifiers live in the comment block at the top of `index.html` as
`__ENVIRONMENT__`, `__COMMIT__` and `__BUILD_TIME__`; substitute them at deploy
time if you need to tell deployments apart. Reading them means viewing source,
which is the right trade for a public marketing page.

## Motion and accessibility

Animation is **opt-in**: everything animated sits inside
`@media (prefers-reduced-motion: no-preference)`, so a visitor who has asked for
reduced motion gets the final state with no transition rather than a
disabled-after-the-fact one. Scroll reveals add `.in` via `IntersectionObserver`;
if that never runs the content is still in the document and visible.

Interactive controls carry their own state: the theme toggle is a
`role="switch"` with `aria-checked`, and the mobile menu button syncs
`aria-expanded`.

The skip link is the first focusable element and targets `<main id="content">`,
which carries `tabindex="-1"` so focus actually moves rather than only the
scroll position. It uses the `skip-link` utility rather than
`sr-only`/`focus:not-sr-only` — `not-sr-only` resets `position`, so it fights
whatever positioning utility you pair it with depending on emit order. Instead
it is parked off-screen with a transform and slides in on `:focus`, which keeps
it in the tab order and the accessibility tree at all times.

## A note on cascade layers

Two rules must stay **unlayered** (outside `@utility`) to work, and both say so
in a comment. `@utility` output lands in Tailwind's `utilities` layer alongside
core utilities like `border` and `max-w-*`, so a declaration that needs to beat
one of those loses if you move it inside:

- `.card-rule, .card-rule-muted, .card-rule-faint { border-top-width: 0 }` —
  the gradient bar replaces the card's top hairline; inside `@utility` it loses
  to `border` and every card gains a stray 1px top border.
- `.max-w-6xl { max-width: 80rem }` — widens Tailwind's own 72rem value.

If you tidy this file, re-check those two before assuming they are redundant.

## Note on installing

`registry.npmjs.org` is unreachable from some networks here (TLS connection
reset). If `npm install` fails that way, the `registry.yarnpkg.com` mirror
works:

```bash
npm install --registry=https://registry.yarnpkg.com
```
