# Cartograph

A single-page, client-only web app that reads GPS EXIF data from JPEG photos
and plots them on an interactive world map. Photos never leave your browser —
there is no backend, no upload, no persistence.

![Cartograph — Geo-tagged Photo Mapper](https://img.shields.io/badge/status-experimental-c8553d?style=flat-square)
![License](https://img.shields.io/badge/license-Educational%20Use-1a1f2e?style=flat-square)

## Features

- Drag-and-drop or browse to add JPEGs
- **Import from a public Google Drive folder** (see [Drive import](#importing-from-google-drive))
- Automatic GPS coordinate extraction from EXIF metadata
- Manual lat/lng entry for photos without GPS data
- Photo thumbnails as map markers, with circular crops
- **Spiderfy on hover** — when multiple photos share a location, hovering
  any one of them fans the whole stack out so each thumbnail is visible
- Click a marker for a popup with the full photo, coordinates, and timestamp
- **Timeline scrubber** with autoplay — slide through time and watch photos
  appear in the order they were taken (see [Timeline](#timeline))
- **Share** the current view as a PNG, an animated GIF, or a WebM video
  of the timeline playing back (see [Sharing](#sharing))
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

| Concern             | Library                                                                  |
| ------------------- | ------------------------------------------------------------------------ |
| Map rendering       | [Leaflet](https://leafletjs.com) 1.9.4                                   |
| Tile source         | CartoDB Positron (with sepia CSS filter)                                 |
| EXIF extraction     | [exifr](https://github.com/MikeKovarik/exifr) 7.1.3                      |
| DOM → canvas        | [html-to-image](https://github.com/bubkoo/html-to-image) 1.11.13         |
| GIF encoding        | [gif.js](https://github.com/jnordberg/gif.js) 0.2.0                      |
| Video encoding      | Native `MediaRecorder` + `canvas.captureStream`                          |
| Everything else     | Vanilla HTML / CSS / JavaScript                                          |

State lives in a single `state` object (`photos`, `activeId`, `timeline`);
mutations are followed by explicit `render()` and/or `redrawMarkers()` calls.
Markers are Leaflet `divIcon`s containing the photo as an `<img>` clipped to a
circle. Overlapping markers are detected by projecting their lat/lng into
screen-space pixels after every zoom and grouping any that fall within ~36 px
of each other.

## Timeline

If at least two of your located photos have EXIF timestamps and those
timestamps are not all identical, a **timeline bar** appears at the bottom
of the map.

- **Slider** — drag the thumb to set a cutoff time. Only photos taken at or
  before that moment stay visible. Photos without a timestamp are hidden
  while the filter is active.
- **▶ Play / ⏸ Pause** — autoplays the slider from the earliest to the
  latest photo over about 9 seconds, revealing markers as their timestamps
  pass. Click again to pause; dragging the slider also pauses.
- **Show All** — clears the filter and brings every photo back, including
  those without timestamps. The button is highlighted whenever the filter
  is active.

## Sharing

Once you have at least one located photo, the **Share** button (top right)
opens a menu with three formats:

- **PNG image** — a clean snapshot of the current map view (cursor
  coordinates and zoom controls are hidden during capture). Fastest option,
  smallest file, works everywhere. Use this for static posts.
- **Animated GIF** — drives the timeline cutoff through 30 frames from the
  earliest to the latest photo, captures each frame, then encodes a GIF in
  web workers. Slowest option, but plays inline in chat apps and on social
  platforms that don't accept video.
- **WebM video** — same 48-frame capture, then encoded via
  `MediaRecorder` + `canvas.captureStream`. Smaller files than GIF, but not
  every platform accepts WebM (notably iOS share sheets).

A progress dialog shows captured-frame and encoding status, and has a
**Cancel** button. When ready, the file is offered to the native share sheet
via `navigator.share` if your browser supports file sharing; otherwise it
downloads as `cartograph-…`.

GIF and WebM both require **at least two timestamped photos** with different
timestamps. PNG works on any located photo.

## Importing from Google Drive

Cartograph can pull all images out of a public Google Drive folder. Click
**Import from Google Drive** below the dropzone and provide:

1. **Folder URL or ID** — anything like
   `https://drive.google.com/drive/folders/<ID>` or just `<ID>`. The folder
   must be shared as **Anyone with the link can view**.
2. **Drive API key** — a free key from your own Google Cloud project.
   It is stored in `localStorage` on your machine and never sent anywhere
   except to Google's API.

The app lists every image in the folder, downloads each one, then runs them
through the same EXIF/GPS pipeline as locally-added files. Subfolders are
not traversed.

### One-time API key setup

1. Open the [Google Cloud console](https://console.cloud.google.com/) and
   create (or select) a project.
2. Go to **APIs & Services → Library**, search for **Google Drive API**,
   and click **Enable**.
3. Go to **APIs & Services → Credentials → Create credentials → API key**.
   Copy the generated key.
4. *(Recommended)* Click the new key and restrict it to the **Google Drive
   API** under "API restrictions", and to your origin under "Application
   restrictions".
5. Paste the key into the Drive API key field. It will be remembered for
   future sessions.

If imports fail, the most common causes are: the folder is not set to
"Anyone with the link can view"; the Drive API has not been enabled for
the project; or the API key has restrictions that block this origin.

## Privacy

Cartograph runs entirely in your browser. Locally-added photos and their GPS
data are never sent to any server. The only network requests the app makes
are for the CDN-hosted libraries (Leaflet, exifr, html-to-image, gif.js) and
the map tiles themselves (CartoDB / OpenStreetMap).

If you use the **Google Drive import** feature, your Drive API key and the
folder ID are sent to `googleapis.com` in order to list and download the
images — that is unavoidable, but the requests go directly from your browser
to Google.

The **Share** feature renders images, GIFs, and videos locally; nothing is
uploaded. If your browser exposes the Web Share API (`navigator.share`) and
you choose to share through it, only the destination app you pick from the
share sheet receives the file.

## License

Released under a custom **Educational Use License** — an MIT-style license
modified to restrict use to **educational and non-commercial purposes only**.

You are free to use, study, modify, and redistribute the software for
learning, teaching, coursework, personal projects, and non-funded research.
Commercial use requires separate permission from the copyright holder.

See [`LICENSE`](LICENSE) for the full text.
