# GPX Cinematic

Web app (a single file, no build step) that turns a GPX track into a 16:9
video animation over satellite imagery, with MP4 export straight from the
browser.

## Running it

**Online**: <https://314rch.github.io/gpx-cinematic/> (GitHub Pages, nothing to
install; your tracks, photos and videos stay on your computer, and nothing is
sent anywhere except the track for OpenStreetMap road detection, on request).

**Locally**:

```bash
cd gpx2mp4 && python3 -m http.server 5178
```

then open <http://localhost:5178/index.html>.

After updating `index.html`, reload the page bypassing the cache
(Cmd+Shift+R): the small Python server does not stop the browser from caching it.

A mini server is required: opening `index.html` by double-clicking it may lead
the browser to block network requests (map tiles).

Tested on Chrome / Edge / Brave and Safari 17+ (MP4 export uses WebCodecs).

## Usage

1. Drop a `.gpx` file (or click the drop zone).
2. Set the title, color, camera and pacing in the left panel.
3. `Space` (or ▶) to preview, the timeline to move through time.
4. **Export MP4**: frame-by-frame rendering at 1920×1080.

## How the animation unfolds

| Phase | What happens |
|---|---|
| Intro | Overview of the track, north up, then zoom in to the start point |
| Route | Tilted camera following the current point and looking ahead, persistent track |
| Outro | Zoom out to the north-up overview + end card: title, **end subtitle** (free text, specific to each track, nothing if empty), then distance, max altitude, duration (if the GPX is timestamped) |

## Multiple tracks

You can load several GPX files at once: multiple selection in the file
picker, or drag and drop several files or a whole folder (subfolders
included) onto the drop zone.
The tracks appear in a list; clicking one makes it **active**.

- The active track is the one the camera follows and the one that is **exported**.
- Each track keeps its own waypoints (start, intermediate waypoints,
  finish, section colors), its title, its subtitle and its duration.
- Other tracks are drawn **in gray** when they are in view, with their starts
  and finishes (gray markers). A gray label that would overlap a marker of
  the active track is hidden.
- A track that is already loaded (same content) is not added twice.

## Tabs

- **🎬 Video**: the 16:9 preview exactly as it will be exported, with playback.
- **🗺 Map & photos**: the real, interactive map (scroll wheel, drag),
  with the active track, waypoints (draggable) and photos; on the right, a
  vertical strip of thumbnails. “🖈 Place on map” opens this tab in
  waypoint-adding mode.
- **✂ Clips**: trimming of the active track’s video clips (see below).
- **🎵 Soundtrack**: the active track’s music, on a timeline under the video
  (see below).

**Theme**: the button at the top right of the panel switches the interface
between ◐ Auto (follows the system setting), ☀ Light and ☾ Dark. The choice
is remembered by the browser. The video preview and the clip editor stay
dark in both themes, since that is what gets exported.

## Photos and videos

**Media are organized by track.** Select a track, then
**📁 Active track folder** (Track photos & videos section): its contents are
attached to that track only. The track list shows what each one contains
(📷 photos, 🎬 videos); the Map & photos tab and the video only use the
active track’s media. A simple layout: one subfolder per day or per GPX.

- **Photos**: jpg, png, webp (heic in Safari only). Location and date are
  read from EXIF, including the `eXIf` chunk in PNGs (iPhone exports).
- **Videos**: mp4, mov, m4v, webm (iPhone HEVC included in Chrome/Safari on
  macOS). Location and date are read from the QuickTime metadata
  (`com.apple.quicktime.location.ISO6709` or `©xyz`, date with time zone),
  reading only the file header. In the video, the clip is played in full
  (or as edited in the ✂ Clips tab) while the marker is stopped, with no
  duration limit. The caption shows playback progress
  (“▶ 0:04 / 0:12”), with a progress bar at the bottom of the clip. On
  export, each frame is synced exactly to the matching moment in the clip.
  Audio is not included.

### Adding media to a track

Select the track, then **📁 Active track folder**. When a project is opened,
folders are found again automatically (see “Project”).

**“Unreadable” video**: hover over the label to see the exact reason. A
transient playback error (common with iPhone HDR videos) is retried once
automatically. If the browser really cannot decode it, convert it to H.264
with the macOS tool, then copy its location and date back:

```bash
avconvert --preset Preset1920x1080 --source IMG.mov --output IMG.mp4
```

```bash
exiftool -overwrite_original -TagsFromFile IMG.mov -Keys:GPSCoordinates -Keys:CreationDate IMG.mp4
```

The strip in the Map & photos tab has three filters: **All**,
**Geotagged**, **Unplaced** (with counts).

- **Blue sphere**: photo geotagged by its GPS. **Orange sphere**: location
  placed by hand. **“?”**: no location. Clicking the “?” thumbnail (or 📍 to
  move any photo) switches to placement mode: the next click on the map sets
  its location (Esc cancels).
- A location placed by hand takes priority over GPS. For a photo that has
  both, **⌖** (on the thumbnail) reverts to GPS, and the
  **⌖ Restore GPS positions** button does it for all of them.
- Hand-placed locations and inclusion choices are saved in the browser
  **and** in the `.json` project, track by track (key: file name, size and
  date), and come back when you reopen the same folder. An edited or
  re-exported file (different date) is matched by its name. A reopened
  project flags with ⚠ the tracks whose folder needs reloading. The files
  themselves are never modified.
- **Out-and-back, loops**: when the track passes the same place again, a
  photo is near several passes. The pass chosen is the one that respects the
  chronological order of the shots (a later photo is further along the
  track), even if the GPX is not timestamped. The thumbnail’s **⇄** button
  switches to the other pass; this choice is saved. For a waypoint, the km
  field in the list picks the pass.
- **Detours** (a waterfall 2 km off the road…): an “off track” media item
  can be ticked by hand; it is shown when the marker passes the nearest point
  of the track (“km 66.1 · detour 2.5 km”).
- A photo is attached to the active track if it is within the
  **Max. distance** (200 m by default); otherwise it is marked “off track”.
  Each thumbnail’s **video** checkbox chooses which photos to show.

### Placement by time (to be confirmed)

**🕒 Suggest positions from time** computes a location for photos without
GPS, without applying anything: they appear dotted, and each suggestion can
be accepted (✓) or rejected (✗), or all at once.

1. If a loaded track is **timestamped** and covers the photo’s time, the
   location comes from the GPX. The EXIF time has no time zone: it is
   converted to UTC using the EXIF time zone if there is one, otherwise with
   an offset calibrated automatically from photos that have a GPS time,
   otherwise with the **Clock − UTC (h)** field.
2. Otherwise, the location is **interpolated between the geotagged photos**
   of the active track, in proportion to time (same clock, no offset to
   know). Before the first or after the last geotagged photo, it is
   extrapolated at the average speed and flagged “to check”.

### Presentation in the video

- **Spotlight** (default): the photo flies out of its sphere on the map up to
  a large centered view; the map blurs and darkens behind it, the indicators
  fade out; the photo slowly zooms in (Ken Burns). The caption (date,
  distance, video time, “2 / 4”) is placed **below** the image, on a dark
  plate: on one line under a wide image, on two centered lines under a narrow
  (portrait) image, never over the image. Photos in the same group follow
  one another with a crossfade, then the last one returns to its sphere.
- **Corner frame**: the photo in the top-right corner, like a print, with the
  caption below; the print keeps a minimum width so that a portrait photo has
  room for its caption (on two lines if needed).

In both cases, each selected photo has a sphere on the map at its location
and a dot on the minimap.

**Marker stop**: when reaching a photo or video, the marker brakes firmly
(≈ 0.8 s) and stops in place. It only moves on once the display is over:
per-photo duration elapsed, or video played to the end (or its edit),
including the photo’s fly-out and return. It then speeds up again smoothly.
Media very close to each other (less than 150 m, or less than one second of
travel apart) form a group: a single stop, at the first item, during which
they follow one another. The video gets longer accordingly. Untick
“Stop the marker during photos and videos” to have media shown without
stopping, while the marker keeps moving.

### Editing videos (✂ Clips tab)

On the left, the active track’s videos (edit duration, number of segments,
km or status: unplaced, excluded, off track). On the right, the editor’s
player (with sound, which the exported video does not have; 🔇/🔊) and a
timeline illustrated with frames from the clip.

A clip can keep **several segments**, played in order:
- click the timeline to move there (and select the segment located there);
  drag the **yellow handles** to adjust the start or end of a segment, the
  frame at the cut point is shown;
- **⟦ In here** / **Out here ⟧** (keys **I** / **O**): segment edge at the
  current position;
- **✂ Split here** (**S**): splits the segment in two; **🗑 Segment**
  (**Delete**) removes the selected segment. Splitting twice then deleting
  the middle piece removes a passage;
- **✚ Segment here**: new 4 s segment starting at the current position,
  outside existing segments;
- **↺ Whole clip**: reverts to the whole clip.

Below the timeline, the list of segments and, between two segments, the
**transition**: hard cut, crossfade, slide (the next image pushes the
previous one out), fade to black, fade to white, blur. Their duration is set
in “Transitions” (0.8 s by default). **▶ Play edit** (**M**) plays the result
as it will appear in the video. Space = play/pause the file,
← → = ±1 s, ⇧← ⇧→ = ±1 frame.

In the video, the marker stays stopped for the whole edit. Two players take
turns on the same file: while one plays a segment, the other is already
waiting, cued to the next one, which gives clean cuts and makes crossfades
possible.

## Notes (travel journal)

Your own words for the stretches without photos, drawn like a travel
journal: handwritten ink (Caveat) on cream paper with torn edges and bits of
tape. Two styles, chosen per note:

- **Caption**: a paper strip above the figures, while the marker keeps
  moving. For short remarks (“Cool morning, the road climbs through the
  larches”).
- **Story card**: the map blurs and dims, and a page of ruled notebook paper
  (red margin, title, text, km) fills the screen; **the marker stops** while
  it is read, then sets off again — like a photo stop.

Section **Notes** of the panel: put the playhead where the note belongs, then
**+ At playhead**; type an optional title and the text (line breaks are
kept), and pick the style. The reading time comes from the length of the
text (about 3 words per second for a caption, a little slower for a card);
type a duration to override it. ▸ jumps to the note.

Notes are tied to a km of the track, so they stay in place when the route
duration or the photos change. A caption that would fall on a photo stop or
a story card is moved just after it, and captions never overlap each other.
**⇥ Next gap** moves the playhead to the next stretch of at least 12 s with
no photo, clip or note on screen — the stretches worth filling. In the
🎵 Soundtrack tab, the video lane shows story cards (✎), captions (cream
strips) and those gaps (hatched).

Notes are saved in the project and follow the track if its GPX is updated.

## Soundtrack (🎵 Soundtrack tab)

Each track has its own music: a folder of audio files (mp3, m4a, aac, wav,
ogg, opus, flac), for example `music/20260816/` next to `photos/20260816/`.
**📁 Music folder** picks it; when a project is opened, it is found again
automatically (the folder recorded in the project, otherwise a folder named
after the GPX’s name or date that contains audio files — a folder under
`music/` or `audio/` wins). The track list shows 🎵 with the number of songs.

The tab keeps the video preview on top and shows below it:
- a **timeline**: time ruler, then the video — intro / route / outro,
  waypoints, photo and clip stops with their thumbnails — then the songs on
  two lanes, so that crossfades show as overlaps. Each song block shows its
  waveform (as you will hear it) and its volume envelope. Click or drag on
  the timeline to move the playhead; click a song to select it;
- the **playlist**: drag ≡ (or ▲ ▼) to change the order, untick a song to
  skip it, set its **volume** (0–150 %) and its **fade in / fade out** (s).
  “▶ 0:17” tells when it starts in the video, “not reached” when the video
  ends before it.

Overall settings, per track: **Volume** (master), **Crossfade** (overlap
between consecutive songs, faded out / in, 3 s by default), **End fade-out**
(the music fades out over the last seconds of the video, 3 s by default) and
**Loop** (the playlist starts again if it is shorter than the video). A line
under the playlist says whether the music covers the whole video.

The songs start with the video (intro included). Fades use equal-power
curves, so a crossfade keeps a steady loudness. In the preview, the
soundtrack plays with the picture (🔊 / 🔇 in the transport bar mutes it
without affecting the export); playback is clocked on the audio, so picture
and sound stay in sync, and both wait together if tiles are still loading.
Only the songs the video actually reaches are decoded.

The music is mixed into the exported MP4 as a 48 kHz stereo AAC track. The
order, volumes, fades and settings are saved in the project; the audio files
stay in their folder.

### Voice comments

Recorded comments (mp3, m4a, wav…) sit in their own folder per track, for
example `audio/20260816/` — **🎙 Voice folder** in the 🎵 Soundtrack tab, or
found automatically when the project is opened (a folder named after the
track under `audio/`, `voice/`, `voix/`, `comments/`, `narration/`; music is
never taken from those folders, and comments never from `music/`).

- **Placing**: drag a comment from the list onto the **VOICE** lane of the
  timeline, or put the playhead where it should start and click **⤓ Here**.
  Drag a block on the lane to move it; ✕ takes it off the timeline, ▸ jumps
  to it. A new comment is not placed until you do so.
- **Anchoring**: each comment is tied to a km of the track plus an offset
  in seconds, so it stays at its place when the route duration, the photos
  or the notes change — including inside a photo stop, where the marker is
  still (tested: the offset from the start of the photo is kept to within
  10 ms whatever the duration).
- **Ducking**: while a comment plays, the music goes down automatically to
  **Music under voice** (25 % by default, i.e. −12 dB), over **Duck fade**
  seconds before the comment, and comes back up the same way afterwards.
  The music’s volume envelope on the timeline shows these dips.
- **Volumes**: **Voice** for all comments (up to 200 %), and a volume per
  comment in the list.

Comments play in the preview and are mixed into the exported MP4 with the
music. Their placement, volumes and settings are saved in the project and
follow the track if its GPX is updated. The track list shows 🎙 with the
number of comments.

## Project

**One project = one folder**, for example:

```
replay/
├── full.gpxcine.json             (anywhere in the folder)
├── gpx/                          the original .gpx files (authoritative if they change)
├── photos/
│   ├── 20260815/                 photos and videos from 15 August
│   └── 20260816/
├── music/
│   ├── 20260815/                 the soundtrack of 15 August (mp3, m4a, wav…)
│   └── 20260816/
└── audio/
    ├── 20260815/                 recorded voice comments of 15 August
    └── 20260816/
```

- **📂 Open** (at the top of the panel): choose the project folder. The most
  recent `.gpxcine.json` file is opened, then each track’s photos/videos are
  **reloaded automatically**: from the folder recorded in the project,
  otherwise from the subfolder named after the GPX’s name or date
  (`20260816` ↔ `20260816.gpx`). A folder with no project but containing
  GPX files becomes a new project. You can also drag and drop the folder
  onto it.
- **Default folder**: the browser cannot target a path
  (`/Users/…/replay`) on its own, but it remembers the last project folder
  opened or saved. 📂 Open then starts in that folder, and a
  **↻ Reopen “replay”** button reopens it in one click, without going through
  the picker (Chrome just asks you to confirm access; choosing “Allow on
  every visit” even avoids that confirmation).
- **Modified GPX files**: the `.gpx` files in the project folder
  (e.g. `replay/gpx/`) are authoritative. On opening, a track whose file has
  changed is updated: waypoints are moved onto the new track based on their
  location, start and finish go to the new ends, titles and settings are kept
  (subtitle and duration are recomputed if they were still at their default
  value). A new GPX in the folder is added to the project. Then save to keep
  the update.
- **💾 Save** (or ⌘S): writes directly to the project file; the first time,
  choose the folder to create it in. Media folders are stored in it as
  relative paths.
- Contents: GPX tracks, titles, subtitles, durations, waypoints (name,
  exact location, color), start/finish, settings, active track, and for each
  media item its hand-placed location, its inclusion choice and, for a video,
  its edit (segments, transitions, transition duration), plus each track’s
  roads, notes, soundtrack (music folder, order, volumes, fades, crossfade, end
  fade-out, loop) and voice comments (folder, placement, volumes, ducking). The photo, video and audio files themselves stay in their
  folders.
- Opening and saving to a folder require Chrome or Edge. Elsewhere: a `.json`
  file is opened, saving is done by download, and media folders must be
  chosen again (⚠ in the track list).

## Waypoints

- **Start** and **Finish** are named in the two fields at the top of the
  section: their markers are created at the ends of the track. If the GPX
  already contains a `<wpt>` at one end, its name is used.
  Clearing a field removes the marker.
- The GPX’s `<wpt>` tags are imported and snapped to the track.
- **Place on map**: the map becomes interactive (zoom/pan), each click adds
  a waypoint snapped to the track; markers can be dragged.
- **+ At playhead**: adds a waypoint at the timeline’s current position.
- In the list, the right-hand field (km) moves the waypoint along the track.

When the marker passes a waypoint, its name is shown in a banner and its
marker lights up.

### Section colors

A waypoint’s color swatch sets the track color **from that waypoint
onwards**, up to the next waypoint that defines another one. A faded swatch
means “inherited color”: the previous section continues. The first section
takes the general color (Overlay section), and the finish has no swatch
since no section starts from it. ↺ reverts to the inherited color.

The upcoming track (dotted) already takes the color of its sections. The
current section’s color also applies to the position marker, the displayed
distance, the elevation profile fill, the true-heading triangle on the
compass rose and the minimap. It is saved
in the `.json` project.

## Roads

Below the minimap, a label shows the current road: the number on a sign
(color by prefix: A and N red, D yellow, E green, M royal blue, other
numbers blue), followed by the name. When the road changes, the old label
fades out as the new one appears. **Current road** checkbox in Overlay.

**Roads** section of the panel:
- each row gives the road followed **from that km**, up to the next row.
  Put the playhead where the road changes, then
  **+ At playhead**: a row is created at that km, you just type the name.
  The km can be corrected by hand, ▸ moves the playhead to the start of the
  road;
- type `1 · Hringvegur` or `D902 · Col du Galibier`: what comes before the
  “·” is shown on the sign if it looks like a road number; otherwise the
  name is shown alone, with a road icon. An empty name shows nothing on that
  section (dirt track, off-road…);
- **🛣 Detect (OSM)** suggests the list from OpenStreetMap: the track is
  sent in chunks to the Overpass service (overpass-api.de), each point is
  matched to the nearest way running in the same direction, and sections
  shorter than 400 m (junctions, bridges) are smoothed out. Allow a few
  seconds per 10 km (≈ 20 s for 80 km). The result can then be freely
  edited.

Roads are specific to each track and saved in the project. If the GPX is
updated, each road start is moved based on its location.

## Displayed figures

During the journey: **distance**, **altitude**, **max altitude** reached so
far (and speed if the GPX is timestamped). On the end card: distance,
**max altitude**, duration.

**Elevation gain** is no longer shown by default: on a planned route
(gpx.studio, Komoot…), altitudes come from a terrain model that “sees” the
cliffs and embankments along the road, and the elevation gain is heavily
overestimated (on a 357 km drive through the Westfjords: from 3,200 to
4,800 m depending on smoothing). The **Show elevation gain** checkbox
(Overlay) brings it back, for a track recorded with an altimeter.

## Compass rose

A marine compass in the top-left corner shows the map orientation: the dial
rotates with it (the red N always points to true north), the fixed white
lubber line at the top and the heading shown below it (“NE · 042°”) give
the camera direction. A second triangle, in the current section’s color,
shows the **true heading** (direction of travel along the track): the gap
between the two triangles shows how much the smoothed camera differs from the road. It is part of the exported image; *Compass rose* checkbox
in Overlay to hide it.

## Minimap

To the right of the compass rose, an **always north-up** thumbnail shows the
active track’s extent enlarged by 20%: basemap, full track, route covered so
far in each section’s color, current position and a cone showing the
camera direction. Other loaded tracks appear in gray if they pass through
the area. It appears when the title fades out and disappears for the final
view. *Minimap* checkbox in Overlay.

## Notable settings

- **Active track width** (1 to 2.5×, default 1.5×): the followed track is
  drawn thicker than inactive tracks, which stay thin and gray.

- **Follow zoom / Pitch**: camera height and angle.
- **Marker position on screen**: places the current point lower or higher,
  hence more or less visibility “ahead”.
- **Look-ahead**: distance ahead used to compute the heading.
- **Heading smoothing** (m): geometric smoothing of the track direction.
- **Camera smoothness** (s): rotation inertia, applied in the animation’s
  time domain (zero-phase filter, so the camera does not lag behind the
  path).
- **Max rotation** (°/s): angular speed cap. This is the decisive setting
  on mountain switchbacks: on the test track, the raw heading of the track
  reaches 88 °/s while the camera stays at 25 °/s.

- **Marker freedom on screen** (%): lets the marker drift from the center
  (up to this fraction of the frame width). The camera then follows the
  average line of the route instead of every bend: in a series of hairpins,
  the map no longer sweeps from one edge to the other. 0% = marker always
  centered. On the test track, at 15% (default), camera acceleration is
  divided by 8.5 on average and by 19 at peak.

These settings depend on the route duration: the heading is
recomputed on every change, and stays fully deterministic so that the
frame-by-frame export is identical to the preview.
- **Pace**: constant speed, or following the GPX timestamps
  (stops are then visible).
- **3D terrain**: elevation from Terrarium tiles (Mapzen/AWS).

## Export

- MP4 H.264 1920×1080, 24/30/60 fps, 8 to 28 Mbps, with the soundtrack as
  a 48 kHz stereo AAC track (192 kbps) when the track has music.
- Deterministic rendering: each frame waits for tiles to be fully loaded,
  no blurry tiles or stutter. Export continues if the window goes into the
  background.
- **Preload tiles** (ticked by default): before rendering frames, the app
  runs through the camera path, lists the tiles each view will need and
  downloads them in parallel into an in-memory cache. The
  **⚡ Preload tiles** button does the same on demand.
- **During playback** (▶), the same pass runs in the background, up to
  20 s ahead of the playhead; upcoming photos and videos are also decoded in
  advance. If the network cannot keep up, playback pauses
  (“Loading tiles…”) until it is 2.5 s ahead, rather than showing a blurry
  map. On the test track, with a cold network: 12 times fewer missing tiles
  on screen.
- Measured on the test track: ~145 ms per frame without preloading,
  **~45–60 ms** with it. A 54 s video at 30 fps (1,620 frames) comes out in
  **1 min 41 s**, preloading included, instead of about 4 min 30 s.
- The cache holds up to 700 MB of tiles for the session (status shown below
  the button); since terrain tiles have no HTTP cache header, it is what
  avoids downloading them again.
- The file is assembled in memory: allow ~2 MB per second of video.
- Browser without WebCodecs: falls back to real-time capture (WebM, or MP4 in Safari).

## Data sources

- Satellite basemap: Esri World Imagery (© Esri, Maxar, Earthstar Geographics)
- Topographic basemap: OpenTopoMap (© OpenStreetMap contributors)
- Terrain: Terrarium tiles (Tilezen / Mapzen, hosted by AWS)
- Rendering: MapLibre GL JS · MP4 muxing: mp4-muxer

Personal use: check these services’ terms of use for commercial or heavy
use.

## Files

- `index.html` — the whole app (interface, animation engine, export)
- `samples/photos/` — test photos geotagged along the tracks (including
  a group of 3 at Plan Lachat, one off track and one without GPS)
- `samples/` — adjoining test tracks: Télégraphe → Valloire, Valloire →
  Galibier, Galibier → Lautaret (to try loading a folder)
- `*.gpxcine.json` — saved projects (track + settings + waypoints)
