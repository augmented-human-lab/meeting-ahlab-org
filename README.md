# meeting-ahlab-org

Static site at <https://meeting.ahlab.org> that wraps the
[`ahl-meeting-appscript`](https://github.com/augmented-human-lab/ahl-meeting-appscript)
Google Apps Script web app in a full-viewport iframe so the public URL
stays clean.

## How it works

Apps Script web apps can only be served from
`https://script.google.com/macros/s/<deploymentId>/exec` — Google
doesn't let you point a custom hostname at that URL directly.

This repo hosts a single `index.html` that fills the browser viewport
with an iframe pointing at the Apps Script deployment. The user sees
`meeting.ahlab.org` in the address bar; the app renders inside.

The Apps Script side sets
`HtmlService.XFrameOptionsMode.ALLOWALL` in
[`Code.js`](https://github.com/augmented-human-lab/ahl-meeting-appscript/blob/main/src/Code.js)
so the iframe is allowed.

## PWA install

The wrapper page is an installable PWA — visiting `meeting.ahlab.org`
prompts users to install it as a standalone app (icon on desktop / home
screen, opens in its own window without browser chrome).

- [`manifest.json`](manifest.json) — name, icons (AHL favicon from
  cdn.ahlab.org), `display: standalone`, AHL purple theme color.
- [`sw.js`](sw.js) — minimal service worker. Caches the wrapper shell
  (`index.html`, `manifest.json`) for offline launch; the Apps Script
  iframe content goes straight to the network (we can't cache cross-origin
  Google traffic). Cache name is `meeting-ahlab-v1` — bump the version
  string when you want users to pick up wrapper-side changes immediately.

Once installed, the app opens at `https://meeting.ahlab.org/?source=pwa`
so installed launches are distinguishable in analytics from browser
visits.

## DNS / hosting

- DNS: `meeting.ahlab.org` → GitHub Pages (`augmented-human-lab.github.io`).
- GitHub Pages: served from the `main` branch root of this repo.
- `CNAME` file ensures GitHub Pages serves it on `meeting.ahlab.org`.

## When the Apps Script deployment URL changes

The deployment ID is baked into `index.html`. If you ever create a
*new* deployment in Apps Script (instead of redeploying against the
existing stable deployment), update the `src` attribute in
[`index.html`](index.html) and push.

The stable deployment is intentionally not rotated — see
[`ahl-meeting-appscript/deploy.js`](https://github.com/augmented-human-lab/ahl-meeting-appscript/blob/main/deploy.js).
