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
