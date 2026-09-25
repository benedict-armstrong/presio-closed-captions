# Closed Captions for Presio

Live captions of what the presenter says, on the slide, for every viewer.
A [Presio](https://presio.ch) plugin.

- Press **Captions** in the controller's bottom bar (or `t`) to start. Your
  browser asks for the microphone once.
- What you say appears at the bottom of the slide on every viewer screen, and
  on your own current slide (switch that off in the settings).
- Settings (Settings → Plugins → Closed Captions): language, text size,
  position, how many lines, how long they stay after you stop talking.

## Install

In Presio: **Settings → Plugins → Add**, and paste one of these:

```
https://cdn.jsdelivr.net/gh/benedict-armstrong/presio-closed-captions@v0.1.0/
https://raw.githubusercontent.com/benedict-armstrong/presio-closed-captions/v0.1.0/
```

Use a tag (`@v0.1.0`) rather than a branch (`@main`). Viewers load the plugin
themselves and check it against the presenter's copy, byte for byte. A branch
URL is cached for minutes (raw) to hours (jsDelivr), so after a push the
presenter and the audience can get different versions, and then viewers
silently go without captions.

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
