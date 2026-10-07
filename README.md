# JV

Browser build of **Just Volleyball** — the complete Godot 4.7.2 web export, playable
in any modern desktop browser. Nothing is re-implemented in HTML or JavaScript: the
page is Godot's own export shell and the game itself lives in the package, so this
plays the same build the desktop version ships.

## Play

Serve the folder over HTTP — `file://` cannot fetch a wasm/package pair:

```bash
python -m http.server 8000
```

Then open <http://127.0.0.1:8000/>. The first load transfers the whole engine and
package (~150 MB), so it takes a moment on a cold cache; later loads come from the
browser cache.

Any static host works, with one condition: serve `index.wasm` as
`application/wasm` and, if you can, compress `index.wasm` and `index.pck` (they are
already compressed well by the stock export and reach roughly 92 MB together). The
build uses the non-threaded template, so no `Cross-Origin-Opener-Policy` /
`Cross-Origin-Embedder-Policy` headers are needed.

## Contents

| file | size | what it is |
| --- | --- | --- |
| `index.html` | 5 KB | the export shell (loader, splash, canvas, progress bar) |
| `index.js` | 273 KB | the engine bootstrap the export generates |
| `index.wasm` | 37.7 MB | Godot 4.7.2 engine, web build |
| `index.pck` | 105.7 MB | every game asset and script the game ships |
| `index.audio.*.worklet.js` | 10 KB | AudioWorklet shims used for audio output |
| `index*.png` | 114 KB | favicon, splash and touch icon |

`index.wasm` and `index.pck` are tracked with [Git LFS](https://git-lfs.com). Install
it before cloning, or the clone will contain pointer files instead of the payload:

```bash
git lfs install
git clone https://github.com/PrismDev3/jv-source
```

## What to expect

* The engine only exposes the **Compatibility (WebGL 2.0)** renderer on the web, so
  the browser build renders through a simpler pipeline than the desktop version.
  The game's own environments never use SDFGI, SSAO, SSIL, volumetric fog or SSR,
  so the visible gap is mostly shadow and lighting quality.
* Chromium logs `GL_INVALID_OPERATION: Active draw buffers with missing fragment
  shader outputs` once 3D content is on screen. It comes from the Compatibility
  renderer, it does not stop anything from drawing, and it is the usual complaint
  WebGL reports for multi-attachment framebuffers.
* First run follows the same flow as the desktop build: a notice, then "would you
  like to configure your settings first?", then the character locker. Browsers keep
  audio suspended until the first click, and the game waits for one rather than
  failing.
* Saves, settings and the in-browser log live in IndexedDB under the site origin.
* The shipped texture cache only holds S3TC/BPTC VRAM variants, so desktop browsers
  are fine but **mobile** browsers (which need ETC2/ASTC) will fail to load the VRAM
  textures.
* The online court browser (WebRTC rooms) is STUN-only with no TURN relay, so hosts
  behind symmetric NAT or restrictive corporate proxies may not connect. Platforms
  without the Steam client simply skip all Steam lobbies, invites and achievements.
