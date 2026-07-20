# 🖼️🧠 Neural network learns Oliver

A **from-scratch neural network** (vanilla JS, no libraries) that learns to **paint a picture**.
The network's only input is a pixel coordinate **(x, y)**; its output is that pixel's color **(R, G, B)**.
By training on an image, it gradually reproduces the whole thing — watch it go from a blurry smear to a
recognizable portrait of **Oliver** (the hero of [Snový Svet](https://github.com/m1omg/snovy-svet-game)).

**▶ Live:** https://m1omg.github.io/neural-oliver/

## How it works
- **Engine:** a hand-written MLP with flat `Float32Array` weights and preallocated buffers —
  forward, backprop, and optimizer are all from scratch and allocation-free in the hot loop.
- **Positional encodings** (the sharpness trick — without one, an MLP can only produce smooth
  blurs due to *spectral bias*):
  - **Axis-aligned Fourier features** — `sin/cos` at octave frequencies.
  - **Gaussian random Fourier features (RFF)** — a random Gaussian frequency matrix
    (Tancik et al., 2020), usually noticeably sharper than the axis-aligned version.
- **SIREN** — sinusoidal activations `sin(ω₀·z)` with the special ω₀ = 30 initialization
  (Sitzmann et al., 2020). Works straight from raw `(x, y)` coordinates, no encoding needed
  (the UI switches automatically).
- **Optimizers:** Adam, **AdamW** (decoupled weight decay), and SGD + momentum for comparison.
  Switching optimizers keeps the learned weights and resets only the moments.
- **Losses:** MSE, L1, Huber (the displayed loss/PSNR is always MSE so runs stay comparable).
- **Coarse-to-fine frequency annealing** (à la BARF) — high-frequency features are gradually
  "unlocked" during training, so the network learns shapes first, details later.
- **Error-driven importance sampling** — minibatch pixels are picked with a "best of 4 random
  candidates" rule using a live per-pixel error map, focusing compute where the network is worst.
- **LR schedule** — linear warmup + cosine decay, with the effective LR shown live.

## Live views
- **Target / reconstruction** side by side, progressively re-rendered every frame.
- **Error heatmap** (magma colormap) — see exactly where the network still struggles.
- **Loss curve** — log-scale, EMA-smoothed, with adaptive history decimation.
- Stats: step, MSE, PSNR, parameter count, steps/s, effective LR.

## Extras
- **✨ HD render** — the network is a *continuous* image representation, so it can be sampled at
  any resolution: render at 2× display size and download as PNG.
- **🖼 Custom image** — upload or drag & drop any image and watch the network learn it instead.
- Full control over architecture, encoding detail, activation, optimizer, loss, learning rate,
  batch size, and steps per frame.

## 🧭 Bonus demo: Latent space explorer (`latent.html`)
A second from-scratch demo: an **autoencoder** (`6912 → 128 → 32 → 2 → 32 → 128 → 6912`) trains
live in the browser to compress each image down to just **two numbers** and decompress it back.
- **Drag a cursor** around the 2D latent space and the decoder renders what lives there in real time.
- A **mosaic background** is progressively decoded over the whole plane, so you can *see* the map
  the network built; colored dots are the (augmented) training samples, encoded live.
- Latent **noise injection** during training keeps the space smooth between images; each source
  image gets 16 augmented variants (shift / zoom / rotate / mirror) so clusters form.
- **🎬 Tour mode** animates the cursor between class centroids; you can also **add your own image**
  as a new class mid-training and watch the map reorganize.

## Run locally
It must be served over HTTP (so the browser can read the image pixels), not opened as `file://`:
```bash
python3 -m http.server
# then open http://localhost:8000/
```

Single self-contained `index.html` + `oliver.png`. Oliver artwork by **Nero** 💙.
