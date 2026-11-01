# Flor de la Mano

Open your hand in front of the webcam and a flower grows out of your palm.

That's basically it. A stem sprouts from below, a few leaves unfold, a bud forms, and then the petals open up layer by layer as you spread your fingers. Close your hand and it all folds back. When the flower opens all the way there's a little flash, mostly because I thought it looked cool.

Everything runs in the browser, in a single HTML file. No build step, no framework, no backend.

## Running it

The camera only works on `https` or `localhost`, so opening the file directly with `file://` usually won't work. The easiest way:

```bash
python -m http.server 8000
```

Then go to `http://localhost:8000`, click **EMPEZAR**, allow camera access and show your hand.

You need an internet connection the first time because the MediaPipe model and WASM files load from a CDN. I've mostly used it in Chrome. Other browsers should work, but the glow relies on `ctx.filter` blur, so if it looks flat in yours, that's probably why.

## How it works

**Hand tracking.** I'm using MediaPipe's `HandLandmarker` (tasks-vision 0.10.14) in video mode with one hand. It gives back 21 landmarks per frame.

**Measuring how open the hand is.** This took the most trial and error. My first attempts depended too much on hand size and how far you were from the camera. What ended up working:

- For each finger, compare the distance wrist → fingertip with wrist → middle knuckle (PIP). A straight finger has its tip way farther out than its knuckle; a bent one doesn't. Since it's a ratio, it doesn't care about hand size or distance.
- The thumb is measured differently, by how far it is from the base of the pinky.
- On top of that there's a small auto-calibration that stretches the range to whatever your hand actually does, and slowly drifts back over time. If it gets confused after some weird gesture, press **R** to reset it.

The HUD in the corner shows the raw value and the current calibration range, which helped a lot while tuning.

**Growing the flower.** One value from 0 to 1 drives everything, split into overlapping phases:

| Openness | What happens |
|---|---|
| 0 – 34% | stem grows, green sparks climb up |
| ~18 – 60% | leaves unfold, sepals, bud appears |
| 38 – 100% | four rings of petals open, outer first |
| ~94% | flash + shockwave |

Each petal gets a bit of random size and angle so it doesn't look too perfect, and the rings rotate slowly in alternating directions.

**Rendering.** It's all plain Canvas 2D. The flower gets drawn on two offscreen canvases: one sharp (the actual flower) and one with just the light stuff (aura, rings, pollen). The glow is faked by drawing those layers several times with different blur amounts in `lighter` mode, then putting the sharp flower on top so the shape stays readable.

The camera feed is mirrored and uses `object-fit: cover`, so landmark coordinates get mapped through the same crop to make the flower actually land on your palm.

## Privacy

The video never leaves your computer. Detection happens locally in the browser, nothing is uploaded or recorded.

## Things I'd like to add

- Two hands, two flowers
- Different flower types
- Petals that fall when you close your hand quickly
- A mobile-friendly version (it runs, but it's heavy)

## Credits

Hand tracking by [MediaPipe](https://developers.google.com/mediapipe). Everything else is hand-drawn with canvas paths, no images or assets.
