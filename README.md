# 🌐 Globe3D

**A lightweight, highly-configurable interactive 3D globe built with Three.js and Canvas 2D.**  
No external map dependencies — everything is self-contained in a single HTML file. Designed to embed directly in iOS (WKWebView), Android (WebView), or any web page.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Three.js r128](https://img.shields.io/badge/Three.js-r128-orange.svg)](https://threejs.org)
[![Zero Dependencies](https://img.shields.io/badge/dependencies-0-green.svg)](#)

---

## ✨ Features

- 🌍 **All 250+ countries** rendered from SVG paths directly onto a 3D sphere texture
- 🎨 **Fully themeable** — colors, ocean, grid, glow, stroke, all configurable
- 📱 **Mobile-first** — adaptive texture resolution, touch drag, pinch-zoom
- 🔭 **Smooth camera** — animated zoom-to-country, slerp rotation, inertia-free drag
- 📡 **Native bridge** — iOS WKWebView, Android JSInterface, and web `window` callbacks
- ⚡ **Zero npm** — single HTML file, one Three.js CDN import
- 🛠 **Rich JS API** — colorize countries, swap SVGs, control camera programmatically

---

## 🚀 Quick Start

### Web (iframe)
```html
<iframe src="src/globe3d.html" style="width:100%;height:500px;border:none"></iframe>
```

### Web (direct)
Just open `src/globe3d.html` in any modern browser.

### iOS (WKWebView)
```swift
let webView = WKWebView(frame: view.bounds, configuration: config)
let url = Bundle.main.url(forResource: "globe3d", withExtension: "html")!
webView.loadFileURL(url, allowingReadAccessTo: url.deletingLastPathComponent())
```

### Android (WebView)
```kotlin
webView.settings.javaScriptEnabled = true
webView.addJavascriptInterface(MyBridge(), "Android")
webView.loadUrl("file:///android_asset/globe3d.html")
```

---

## 🗂 Project Structure

```
globe3d/
├── src/
│   ├── globe3d.html                      # ← The globe (self-contained)
│   └── three.min.js                      # Three.js r128 (copy from CDN or npm)
├── demo/
│   └── index.html                        # Full interactive configuration demo (8 tabs)
├── examples/
│   ├── android/
│   │   ├── README.md                     # Android setup guide
│   │   └── app/src/main/
│   │       ├── AndroidManifest.xml
│   │       ├── assets/                   # ← copy globe3d.html + three.min.js here
│   │       ├── java/com/globe3d/example/
│   │       │   ├── GlobeActivity.java    # Java integration (full example)
│   │       │   └── GlobeActivityKt.kt   # Kotlin integration (full example)
│   │       └── res/layout/
│   │           └── activity_globe.xml   # WebView layout with overlay UI
│   └── ios/
│       ├── README.md                     # iOS setup guide
│       └── Globe3DExample/
│           ├── GlobeViewController.swift # UIKit / Swift (full example)
│           ├── GlobeSwiftUI.swift        # SwiftUI wrapper + GlobeController
│           ├── GlobeViewController.m     # UIKit / Objective-C (full example)
│           └── GlobeViewController.h    # Objective-C header
├── docs/
│   └── API.md                            # Complete API reference
├── README.md
└── LICENSE
```

> **Note:** `globe3d.html` references `three.min.js` via `<script src="three.min.js">`.  
> Copy it from [cdnjs](https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js) into `src/`.

---

## ⚙️ Configuration Reference

All configuration lives in clearly marked constant blocks at the top of `globe3d.html`.  
Edit them before deploy, or override them at runtime via the JS API.

### 🎨 Colors

```javascript
// ── Colour config ──────────────────────────────────────────────────
var COUNTRY_FILL   = '#2a6496';          // Default country fill
var COUNTRY_HOVER  = '#4a9fd4';          // Hovered country fill
var COUNTRY_STROKE = 'rgba(0,0,0,0.35)'; // Country border color
var SELECT_STROKE  = 'rgba(255,255,255,0.9)'; // Selected country border
```

### 🌊 Ocean & Grid

The ocean is drawn as a Canvas 2D linear gradient inside `buildBg()`:

```javascript
function buildBg(){
  var g = _bgCtx.createLinearGradient(0, 0, 0, TEX_H);
  g.addColorStop(0,   '#8ee0d8');   // ← Ocean top color
  g.addColorStop(.5,  '#6dccc4');   // ← Ocean mid color
  g.addColorStop(1,   '#8addd6');   // ← Ocean bottom color
  _bgCtx.fillStyle = g;
  _bgCtx.fillRect(0, 0, TEX_W, TEX_H);

  // Grid lines
  _bgCtx.strokeStyle = 'rgba(255,255,255,0.18)'; // ← grid opacity
  _bgCtx.lineWidth   = 0.4;
  // ... draws 30° graticule
}
```

### 🔭 Camera & Zoom

```javascript
var FOV        = isMobile ? 55 : 42;   // Field of view (degrees)
var GLOBE_R    = 1.0;                  // Globe radius (Three.js units)

// Auto-computed from FOV — override if needed:
var FIT_Z      = GLOBE_R / Math.sin(_fovRad / 2);   // camera distance = full fit
var NEAR_Z     = FIT_Z * 0.68;                       // zoom-in distance (68% of fit)
var ZOOM_OUT_Z = FIT_Z * 1.05;                       // default zoom-out (slight pull-back)
var ZOOM_IN_Z  = NEAR_Z;                             // used when country is selected

var MIN_Z = 0.5;   // Hard minimum zoom (scroll/pinch limit)
var MAX_Z = 5.5;   // Hard maximum zoom
```

### 🔄 Rotation

```javascript
var autoSpeed = 0.15;   // Rotation speed (0 = stopped, 1.5 = fast)
// Direction: always around Y-axis (east→west). To reverse, negate autoSpeed.
```

### 📐 Texture Quality

```javascript
var TEX_W = isMobile ? 2048 : 4096;   // Texture width in pixels
var TEX_H = isMobile ? 1024 : 2048;   // Texture height in pixels

// Globe mesh segments (higher = smoother sphere):
var globe = new THREE.Mesh(
  new THREE.SphereGeometry(GLOBE_R, isMobile?56:80, isMobile?40:56),
  globeMat
);
```

---

## 🛠 JavaScript API

All functions are globally accessible on `window` (or `iframe.contentWindow`).

### Color & Setup

| Function | Description |
|---|---|
| `setCountryColor(id, color)` | Set fill of one country by ISO-3166-1 alpha-2 code |
| `colorize(pathOrId, color, strokeColor?)` | Alias with optional stroke |
| `setupCountry(ids[], selColor, fill, bg, stroke)` | Batch setup — selected countries + all colors |
| `setupCountries(ids[], selColor, fgColor)` | Lighter version without background |
| `loadSVG(rawSVG, ids[], selColor, bg, fg)` | Replace SVG paths at runtime |
| `setMapBackground(color)` | Change body/canvas background |
| `getRandomColor()` | Returns a random hex color string |

### Camera & Navigation

| Function | Description |
|---|---|
| `zoomToCountry(idOrElement)` | Smooth-rotate globe so country faces camera (+Z) |
| `animateZoom(targetZ, onDone?)` | Ease camera to Z distance over 40 frames |
| `setZoom(z)` | Instantly set camera Z (clamped to MIN_Z..MAX_Z) |
| `zoomIn()` | Step in by 0.4 units |
| `zoomOut()` | Step out by 0.4 units |
| `pauseGlobe()` | Stop auto-rotation |
| `resumeGlobe()` | Resume auto-rotation |
| `resetView()` | Reset orientation to identity + zoom out |
| `deselectAll()` | Clear selection and restore rotation |

### Selection

| Function | Description |
|---|---|
| `toggleCountry(pathElement)` | Select/deselect a country (internal, but safe to call) |
| `sel` | `Set<string>` of currently selected country IDs |
| `customColors` | `{ [id]: hexColor }` — override per-country fill |
| `customStrokes` | `{ [id]: color }` — override per-country stroke |

---

## 📡 Native Bridge Events

Globe fires events to three bridge types simultaneously. Implement whichever fits your platform:

### Web (recommended for iframe usage)
```javascript
// Inject before / after globe loads:
iframe.contentWindow.NativeBridge = {
  didTapPath: function(jsonString) {
    var data = JSON.parse(jsonString);
    // data = { id, selected, centroid:{x,y}, bounds:{x,y,width,height}, color, stroke }
    console.log('Tapped:', data.id, data.selected);
  }
};
```

### Startup callback
```javascript
// In parent page — called ~200ms after globe init:
window.onGlobeReady = function() {
  console.log('Globe is ready');
};
```

### iOS (WKWebView)
```swift
// In WKScriptMessageHandler:
func userContentController(_ ucc: WKUserContentController,
                           didReceive message: WKScriptMessage) {
    if message.name == "didTapPath" {
        let json = message.body as! String
        // parse json: { id, selected, centroid, bounds, color, stroke }
    }
    if message.name == "globeReady" {
        // globe is loaded and ready
    }
}
// Register handlers:
config.userContentController.add(self, name: "didTapPath")
config.userContentController.add(self, name: "globeReady")
```

Calling back into the globe from Swift:
```swift
webView.evaluateJavaScript("setCountryColor('US','#ff6b35')")
webView.evaluateJavaScript("zoomToCountry('JP')")
```

### Android (WebView + JSInterface)
```kotlin
class GlobeBridge {
    @JavascriptInterface
    fun onCountryTapped(json: String) {
        val data = JSONObject(json)
        val id = data.getString("id")
        val selected = data.getBoolean("selected")
        // handle on UI thread:
        runOnUiThread { handleCountryTap(id, selected) }
    }

    @JavascriptInterface
    fun onGlobeReady() {
        // Globe initialized
    }
}
webView.addJavascriptInterface(GlobeBridge(), "Android")
```

Calling back into the globe:
```kotlin
webView.evaluateJavascript("setCountryColor('CN','#e63946')", null)
```

---

## 🎨 Theming Examples

### Dark Cyberpunk
```javascript
COUNTRY_FILL   = '#1a0a2e';
COUNTRY_STROKE = 'rgba(123,47,255,0.7)';
// Ocean:
g.addColorStop(0, '#0a0020');
g.addColorStop(1, '#050015');
document.body.style.background = '#050505';
```

### Warm Political Map
```javascript
// Color each continent a distinct color
['US','CA','MX'].forEach(id => setCountryColor(id, '#4361ee'));
['CN','IN','JP'].forEach(id => setCountryColor(id, '#e76f51'));
['DE','FR','GB'].forEach(id => setCountryColor(id, '#457b9d'));
```

### Heatmap (data-driven)
```javascript
// countries = [{id:'US', value:0.9}, {id:'CN', value:0.7}, ...]
function heatColor(t) {
  var r = Math.round(255 * t);
  var b = Math.round(255 * (1 - t));
  return 'rgb('+r+',50,'+b+')';
}
countries.forEach(c => setCountryColor(c.id, heatColor(c.value)));
needRedraw = true;
```

---

## 📐 Payload Schema

Every country tap fires this JSON:

```jsonc
{
  "id":       "US",             // ISO 3166-1 alpha-2
  "selected": true,             // toggle state
  "centroid": { "x": 203.4, "y": 312.1 },  // SVG coordinate centroid
  "bounds": {
    "x": 50.2, "y": 290.0,
    "width": 310.5, "height": 120.3        // SVG bounding box
  },
  "color":  "#2a6496",          // current fill (null if default)
  "stroke": null                // current stroke override (null if default)
}
```

---

## 🌐 Browser Support

| Browser | Support |
|---|---|
| Chrome 80+ | ✅ Full |
| Safari 14+ | ✅ Full |
| Firefox 80+ | ✅ Full |
| iOS WKWebView 14+ | ✅ Full |
| Android WebView (Chrome 80+) | ✅ Full |
| Samsung Internet 12+ | ✅ Full |

Requires: WebGL, Canvas 2D, ES6 (`Set`, `const`, arrow functions polyfilled for ES5 targets).

---

## 📁 Adding `three.min.js`

The globe references `<script src="three.min.js">`.  
You can get it in several ways:

```bash
# Option 1: Download directly
curl -o src/three.min.js \
  https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js

# Option 2: npm
npm install three@0.128.0
cp node_modules/three/build/three.min.js src/

# Option 3: Change to CDN in globe3d.html (no local file needed)
# Replace: <script src="three.min.js">
# With:    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js">
```

---

## 🤝 Contributing

1. Fork the repo
2. Create a branch: `git checkout -b feature/my-feature`
3. Make changes to `src/globe3d.html` or `demo/`
4. Open a PR with a clear description

**Areas welcome for contribution:**
- Additional country data layers (population, GDP choropleth)
- Star/atmosphere shader layer
- Marker/pin system (lat/lng → 3D point)
- Offline SVG swap for custom regions
- React / Vue wrapper component

---

## 📄 License

MIT — free for personal and commercial use. See [LICENSE](LICENSE).

---

## 🙏 Credits

- [Three.js](https://threejs.org) — 3D rendering engine (r128)
- SVG world map paths adapted from Natural Earth public domain data
- Built with ❤️ for cross-platform native + web use
