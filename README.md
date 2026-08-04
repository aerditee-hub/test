# Room Scene Viewer

A single self-contained HTML file (`room-scene.html`) that renders a walkable
3D room in the browser using Three.js, with an embedded "J.A.R.V.I.S." voice
assistant HUD. There is no build step and no editor UI in this file — it just
loads baked scene data and lets the user walk around and look at it.

## What it does

- **3D room**: A simple boxed room (floor, four walls, red ceiling) built
  from `THREE.BoxGeometry` primitives, lit with a 3-point light rig (key,
  fill, rim) plus a hemisphere light and an image-based studio environment
  for realistic reflections on PBR materials.
- **First-person movement**: Click the canvas to lock the pointer, then
  `WASD` to walk and mouse to look around. `Esc` releases the pointer / closes
  overlays. Player movement is clamped to the room bounds and collides with
  any object marked "solid".
- **Baked scene data**: All room contents — 3D models (as embedded base64
  GLB), a looping video plane, and interactive "trigger" zones — are stored
  as JSON in a `<script id="savedSceneData" type="application/json">` tag
  near the end of the file. The viewer parses this on load and populates the
  scene; there's no external scene file to fetch.
- **Trigger zones**: Invisible circular areas in the room. Walking into one
  shows a "press E" style prompt; pressing `E` opens an iframe overlay
  loading `html{N}.html` (a numbered file expected to sit next to this HTML
  file) full-screen, closable via `Esc` or the on-screen close button.
- **Video plane**: A billboard mesh textured with an HTML5 `<video>` element
  (`video1.mp4`, expected alongside the file). Starts muted/autoplaying
  (browser requirement) and unmutes on the user's first click.
- **J.A.R.V.I.S. HUD**: A slide-in panel (bottom-left styling, cyan sci-fi
  theme) driven by the Web Speech API:
  - Says "Hey Jarvis" (wake phrase) to activate; "Sleep Jarvis" / "Goodbye
    Jarvis" to deactivate.
  - Once active, spoken questions are first checked against a small local
    keyword knowledge base, then — if no local match — sent to a configured
    backend URL (`BACKEND_URL`, currently a placeholder Vercel endpoint) via
    `POST` with `Content-Type: text/plain` (to avoid a CORS preflight).
  - Answers are shown in the panel and spoken aloud via
    `speechSynthesis`.
  - Auto-sleeps after 4 questions per wake session.
  - Keeps a single long-lived microphone stream and recognition instance
    open for the whole page session (restarting only the recognition
    *session*, not the object) to avoid repeated browser permission prompts,
    plus a watchdog that force-restarts recognition if it goes silent for
    too long ("zombie" detection).

## Requirements to run

This is a static file, but it depends on external assets that are **not**
included in the HTML itself:

1. Any 3D model / video referenced in `savedSceneData` — e.g. `video1.mp4` —
   must be placed in the same folder as this HTML file.
2. Any trigger-zone target pages (`html1.html`, `html2.html`, …) must also
   sit alongside it if triggers are defined.
3. A working backend at the `BACKEND_URL` in the script (or update that
   constant) if you want J.A.R.V.I.S. to answer questions beyond its tiny
   local keyword knowledge base.
4. Serve it over `http://` or `https://` (not `file://`) — pointer lock,
   getUserMedia (microphone), and video autoplay policies generally require
   a real origin. A simple local server (e.g. `npx serve` or `python3 -m
   http.server`) works fine.
5. Voice recognition (`SpeechRecognition`) only works in Chromium-based
   browsers (Chrome/Edge); the page will show a message if unsupported.
6. Three.js and its GLTFLoader/RoomEnvironment addons are loaded from CDN
   (`cdnjs.cloudflare.com`, `cdn.jsdelivr.net`) via an import map, so an
   internet connection is required.

## Controls summary

| Action | Key/Input |
|---|---|
| Look around | Mouse (after clicking to lock pointer) |
| Move | `W A S D` |
| Interact with a trigger zone | `E` |
| Release mouse / close overlay | `Esc` |
| Unmute video + enable audio | First click on the page |
| Activate voice assistant | Say "Hey Jarvis" |
| Deactivate voice assistant | Say "Sleep Jarvis" / "Goodbye Jarvis" |

## File structure notes

Currently `savedSceneData` in this particular file has an **empty**
`objects` array and an **empty** `triggers` array — only one video plane is
defined. So as shipped, this file will render an empty room with a floating
video screen; no interactive props or trigger doorways are present unless
you add them to that JSON block.
