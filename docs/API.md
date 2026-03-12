# Globe3D — API Reference

## Global Variables (configurable at init)

| Variable | Type | Default | Description |
|---|---|---|---|
| `COUNTRY_FILL` | string | `'#2a6496'` | Default fill for all countries |
| `COUNTRY_HOVER` | string | `'#4a9fd4'` | Fill when mouse hovers |
| `COUNTRY_STROKE` | string | `'rgba(0,0,0,0.35)'` | Country border |
| `SELECT_STROKE` | string | `'rgba(255,255,255,0.9)'` | Border when selected |
| `autoSpeed` | number | `0.15` | Auto-rotation speed (0 = stopped) |
| `paused` | boolean | `false` | Whether rotation is paused |
| `FOV` | number | `55` (mobile) / `42` (desktop) | Camera field of view |
| `GLOBE_R` | number | `1.0` | Globe sphere radius |
| `MIN_Z` | number | `0.5` | Minimum camera distance |
| `MAX_Z` | number | `5.5` | Maximum camera distance |
| `TEX_W` / `TEX_H` | number | `2048×1024` (mobile) / `4096×2048` | Texture resolution |

---

## Functions

### `setCountryColor(id, color)`
Set the fill color of a single country.
- `id` — ISO 3166-1 alpha-2 code (e.g. `'US'`, `'DE'`, `'EG'`)
- `color` — Any CSS color string (hex, rgb, rgba, named)

```javascript
setCountryColor('EG', '#e63946');
setCountryColor('JP', 'rgba(255,100,0,0.8)');
```

---

### `setupCountry(selectedCountries, selectedColor, fillColor, backgroundColor, strokeColor)`
Batch-configure the entire globe in one call.
- `selectedCountries` — Array of country IDs to highlight
- `selectedColor` — Fill for highlighted countries
- `fillColor` — Fill for all other countries
- `backgroundColor` — Body/canvas background color
- `strokeColor` — Border color for all countries

```javascript
setupCountry(['US','CA','MX'], '#ff6b35', '#2a6496', '#020810', 'rgba(0,0,0,0.3)');
```

---

### `setupCountries(selectedCountries, selectedColor, foregroundColor)`
Lighter version — does not touch background.

---

### `loadSVG(rawSVG, selectedCountries, selectedColor, backgroundColor, foregroundColor)`
Swap in a custom SVG world map at runtime. Useful for custom projections or regions.
- `rawSVG` — SVG markup string with `<path id="XX">` elements
- Rest of parameters same as `setupCountry`

---

### `colorize(pathOrId, color, strokeColor?)`
Alias for `setCountryColor` that also accepts a path element reference.

---

### `setMapBackground(color)`
Change the globe background (ocean/space color) independently of the canvas texture.

---

### `zoomToCountry(idOrElement)`
Smoothly rotate the globe so the specified country faces the camera.
```javascript
zoomToCountry('JP');
zoomToCountry('AU');
```

---

### `animateZoom(targetZ, onDone?)`
Animate the camera to a specific Z distance over ~40 frames (ease in-out quad).
```javascript
animateZoom(ZOOM_IN_Z);   // zoom in
animateZoom(ZOOM_OUT_Z);  // zoom out
animateZoom(1.8, function() { console.log('done'); });
```

---

### `setZoom(z)`
Instantly set camera Z. Clamped to `MIN_Z`..`MAX_Z`.

---

### `pauseGlobe()` / `resumeGlobe()`
Stop or start auto-rotation.

---

### `resetView()`
Return globe to identity quaternion and default zoom.

---

### `deselectAll()`
Clear all selections and resume auto-rotation.

---

### `getPathsForCountry(id)`
Returns the SVG path element for a country ID.

---

### `getRandomColor()`
Returns a random 6-digit hex color string.

---

## Events (Native Bridge)

The globe fires all events to three bridge types simultaneously:

### `NativeBridge.didTapPath(json)` — Web
### `webkit.messageHandlers.didTapPath.postMessage(json)` — iOS  
### `Android.onCountryTapped(json)` — Android

**Payload:**
```typescript
{
  id:       string;           // ISO country code, empty string if deselected
  selected: boolean;          // true = now selected, false = deselected
  centroid: { x: number, y: number };  // SVG space centroid
  bounds:   { x: number, y: number, width: number, height: number };
  color:    string | null;    // current custom fill (null = default)
  stroke:   string | null;    // current custom stroke (null = default)
}
```

### `window.onGlobeReady()` — Web
Called 200ms after globe initialization.

### `NativeBridge.countriesInfo(json)` — Web (from `setupCountry`)
Fires with an array of all country bounds:
```typescript
Array<{ id: string, x: number, y: number, width: number, height: number }>
```
