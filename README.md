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

## The two gate screens

Browsers that withhold the Google session from a third-party frame (Safari,
Firefox strict mode, private windows) get Google's sign-in page in the frame
instead of the app — and Google refuses to be framed, so the app cannot even
show its own card there. Two screens cover that:

- **"Loading the meeting"** covers the frame from the first paint, with the AHL
  logo spinning as the progress indicator, so whatever Google puts in the frame
  is never visible.
- **"Meeting ready"** replaces it after 7 seconds if the app has not reported
  in, offering a button that opens the meeting in a full tab.

The app posts `{ahl:'ready', user}` to `window.top` from the viewer, the
sign-in card and the access-denied page. Receiving it hides both screens. The
`user` it carries is cached in `localStorage` (`ahl-meeting-user-v1`) and used
to show the member's photo on the button — or their email when there is no
usable photo. A browser where the app has never loaded inside the frame has
nothing cached, so the button carries neither.

## Which URL goes where — get this wrong and the app looks broken

| Address | In the frame | On a button |
|---|---|---|
| plain `/macros/s/<id>/exec` | **use this** — the app always runs, so it can show its own card | with several Google sessions open, Google rewrites it to `/macros/u/<n>/s/…`, which dead-ends on "Sorry, unable to open the file at present." |
| domain `/a/macros/ahlab.org/s/<id>/exec` | Google's own `401. That's an error.` page when there is no @ahlab.org session | **use this** — it resolves to the @ahlab.org session whatever else is signed in |

The frame also appends `?embed=1`, which is how the app tells a frame from a
full tab; it cannot work that out itself, because Apps Script always nests page
HTML in its own frame.

The domain-scoped address pins to the @ahlab.org session regardless of which
account is picked in Google's chooser, so the chooser is not used as a first
step. It survives only as the "Add your AHL account" link, for a browser signed
in to no AHL account at all.

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

The iframe uses the **plain** `/macros/s/<deploymentId>/exec` URL. The
domain-scoped `/a/macros/ahlab.org/s/<deploymentId>/exec` form returns Google's
own `401. That's an error.` page when the browser's default Google account is a
personal Gmail, and that error renders inside the frame. The plain form always
runs the app, which then shows its own card and offers Google's account chooser.
The domain-scoped URL is only ever used as the `continue=` target of that
chooser, once the AHL account has been picked.

An AHL loading screen covers the frame until the app posts `{ahl:'ready'}`, so
nobody sees whatever Google put there in the meantime.

Bump `CACHE` in [`sw.js`](sw.js) on every `index.html` change, or installed
copies keep serving the old shell.

The deployment ID is baked into `index.html`. If you ever create a
*new* deployment in Apps Script (instead of redeploying against the
existing stable deployment), update the `src` attribute in
[`index.html`](index.html) and push.

The stable deployment is intentionally not rotated — see
[`ahl-meeting-appscript/deploy.js`](https://github.com/augmented-human-lab/ahl-meeting-appscript/blob/main/deploy.js).
