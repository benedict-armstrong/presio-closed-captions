# Closed Captions for Presio

Live captions of what the presenter says, on the slide, for every viewer.
A [Presio](https://presio.ch) plugin.

- Press **Captions** in the controller's bottom bar (or `t`) to start. Your
  browser asks for the microphone once.
- What you say appears at the bottom of the slide on every viewer screen, and
  on your own current slide (switch that off in the settings). Where a screen
  has black bars around the slide, the captions go in the bar instead of over
  the slide.
- Drag the captions on your current slide to move them on every screen;
  double-click them to put them back.
- Viewers can drag them to their own spot too, and turn them off or on with
  the **CC** pill in the corner. Both stay on their device.
- The arrow beside the button picks the microphone (Chrome and Edge; other
  browsers use the system's). The choice stays on this device.
- If the button says **Mic in use elsewhere**, another tab or window has
  captions on: Chrome listens in one place at a time.
- Settings (Settings → Plugins → Closed Captions): language, text size,
  position, how many lines, how long they stay after you stop talking.

## Install

In Presio, go to **Settings → Plugins → Add plugin** and enter:

```
github:benedict-armstrong/presio-closed-captions
```

Presio looks up the latest release on GitHub (or the newest tag, or else the
commit `main` is on right now) and pins the plugin to it. Viewers load the
plugin themselves and check it's byte for byte the presenter's copy, so a
moving branch URL could leave them without it. To pick a version, use
`github:benedict-armstrong/presio-closed-captions@v0.3.0`. A
`https://github.com/benedict-armstrong/presio-closed-captions` link works too.

Under the hood this is served by jsDelivr, e.g.
`https://cdn.jsdelivr.net/gh/benedict-armstrong/presio-closed-captions@v0.3.0/`,
which you can also paste directly.

## Browser support

Captions come from the browser's Web Speech API, on the presenter's device
only. Viewers need nothing.

| Presenter's browser | Works | Notes |
|---|---|---|
| Chrome, Edge | ✅ | Audio is sent to Google's / Microsoft's speech service. Needs a network connection. |
| Safari (macOS 14.1+, iOS 17+) | ✅ | On-device or Apple's service. |
| Firefox | ❌ | No speech recognition; the button shows "No captions here". |

## How it works

One `index.html` and no build step, running on two of Presio's plugin surfaces:

- **background** (the presenter's device): runs `SpeechRecognition`. It
  restarts after the browser ends it (silence, the one-minute limit), backing
  off if it keeps failing, and sends:
  - `state` `{ on }`, retained, so late joiners and a reloaded controller
    pick it up;
  - `live` `{ text }`, volatile: the sentence so far;
  - `line` `{ text }`: a finished sentence.
- **slide** (the presenter's current slide and every viewer): draws the last
  few lines at the bottom of the page and fades them out after a pause.

It stays one file on purpose: a single HTML file with inline script and style
loads from *any* static host, including `raw.githubusercontent.com`, which
refuses to run separate `.js` files (see below).

## Developing

```sh
npx http-server -p 5174 --cors   # any static server that sends CORS headers
```

Then add `http://localhost:5174/` in Presio. Presio fetches the manifest
from another origin, so the server **must** send
`Access-Control-Allow-Origin` (Vite's dev server does; `python -m http.server`
doesn't).

## Hosting a Presio plugin from GitHub: what works

Checked against Presio's plugin loader (`client/src/lib/plugins/registry.ts`).

| Host | Single-file plugin | Plugin with separate `.js`/`.css` | Why |
|---|---|---|---|
| `raw.githubusercontent.com/<user>/<repo>/<tag>/` | ✅ | ❌ | Sends CORS `*`, but serves everything as `text/plain` with `nosniff`, so browsers won't run `<script src>`. Inline scripts are fine: Presio fetches the HTML and writes it into its frame. |
| `cdn.jsdelivr.net/gh/<user>/<repo>@<tag>/` | ✅ | ✅ | Correct MIME types, CORS `*`, global CDN, tags cached for good. The files must be committed in the repo at that tag. |
| GitHub Pages (`<user>.github.io/<repo>/`) | ✅ | ✅ | Must be switched on in the repo's settings; always serves the latest deploy (no pinning). |
| Release assets (`/releases/download/…`) | ❌ | ❌ | No CORS headers, and served as `application/octet-stream`. |
| Private repo | ❌ | ❌ | raw needs a token, jsDelivr only serves public repos. |

## License

MIT
