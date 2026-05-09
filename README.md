# Cartograph

A single-page, client-only web app that reads GPS EXIF data from JPEG photos
and plots them on an interactive world map. Photos never leave your browser —
there is no backend, no upload, no persistence.

![Cartograph — Geo-tagged Photo Mapper](https://img.shields.io/badge/status-experimental-c8553d?style=flat-square)
![License](https://img.shields.io/badge/license-Educational%20Use-1a1f2e?style=flat-square)

## Features

- Drag-and-drop or browse to add JPEGs
- Automatic GPS coordinate extraction from EXIF metadata
- Manual lat/lng entry for photos without GPS data
- Photo thumbnails as map markers, with circular crops
- **Spiderfy on hover** — when multiple photos share a location, hovering
  any one of them fans the whole stack out so each thumbnail is visible
- Click a marker for a popup with the full photo, coordinates, and timestamp
- One-click **GeoJSON export** of all located photos
- Live cursor coordinates and zoom controls
- Sepia-toned base map for a warm, cartography-inspired aesthetic

## Running

The app is a single file: [`index.html`](index.html). To run it:

```bash
# Option 1: open directly
open index.html

# Option 2: serve statically (needed for some browsers' EXIF/file APIs)
python3 -m http.server
# then visit http://localhost:8000
```

There is no build step and no install step. External libraries are loaded
from CDNs at runtime.

## Stack

| Concern         | Library                                  |
| --------------- | ---------------------------------------- |
| Map rendering   | [Leaflet](https://leafletjs.com) 1.9.4   |
| Tile source     | CartoDB Positron (with sepia CSS filter) |
| EXIF extraction | [exifr](https://github.com/MikeKovarik/exifr) 7.1.3 |
| Everything else | Vanilla HTML / CSS / JavaScript          |

State lives in a single `state` object; mutations are followed by explicit
`render()` and/or `redrawMarkers()` calls. Markers are Leaflet `divIcon`s
containing the photo as an `<img>` clipped to a circle. Overlapping markers
are detected by projecting their lat/lng into screen-space pixels after every
zoom and grouping any that fall within ~36 px of each other.

## Privacy

Cartograph runs entirely in your browser. Your photos and their GPS data are
never sent to any server. The only network requests the app makes are for the
CDN-hosted libraries (Leaflet, exifr) and the map tiles themselves
(CartoDB / OpenStreetMap).

## License

Released under a custom **Educational Use License** — an MIT-style license
modified to restrict use to **educational and non-commercial purposes only**.

You are free to use, study, modify, and redistribute the software for
learning, teaching, coursework, personal projects, and non-funded research.
Commercial use requires separate permission from the copyright holder.

See [`LICENSE`](LICENSE) for the full text.
