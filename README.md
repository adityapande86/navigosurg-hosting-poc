# navigosurg-hosting-poc

A single static page used to prove out hosting for NavigoSurg. If the deployed
page renders styled, the host is serving HTML and static assets correctly.

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

The page defaults to **dark mode** and offers a toggle (top right) for light.

`dark` is hard-coded on `<html>` in the markup, so dark holds even with
JavaScript disabled. A blocking inline script in `<head>` removes it before
first paint if the visitor previously chose light, which avoids a flash of the
wrong theme. The choice persists under the `navigosurg-theme` localStorage key.

`dark:` variants are class-driven, not `prefers-color-scheme`-driven — see the
`@custom-variant dark` line in `src/input.css`. The OS setting is deliberately
ignored, since the brief calls for dark by default.

## Deploying

Only `index.html` and `styles.css` are served. `.assetsignore` excludes
everything else from the Cloudflare Workers asset upload — importantly
`node_modules`, which the platform creates during its own `npm install` and
which otherwise fails the build on oversized binaries
(`node_modules/workerd/bin/workerd`).

Add any new build-time-only file to `.assetsignore`.

## Build stamp

The footer shows `env`, `commit`, and `built` values so it is always clear which
deployment you are looking at. They ship as placeholders (`local`, `0000000`,
`not set`); substitute them at deploy time by targeting the
`data-build="..."` attributes in `index.html`.

## Note on installing

`registry.npmjs.org` is unreachable from some networks here (TLS connection
reset). If `npm install` fails that way, the `registry.yarnpkg.com` mirror
works:

```bash
npm install --registry=https://registry.yarnpkg.com
```
