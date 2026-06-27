# 🖼️🧠 Neural network learns Oliver

A tiny **from-scratch neural network** (vanilla JS, no libraries) that learns to **paint a picture**.
The network's only input is a pixel coordinate **(x, y)**; its output is that pixel's color **(R, G, B)**.
By training on an image, it gradually reproduces the whole thing — watch it go from a blurry smear to a
recognizable portrait of **Oliver** (the hero of [Snový Svet](https://github.com/m1omg/snovy-svet-game)).

**▶ Live:** https://m1omg.github.io/neural-oliver/

## How it works
- **Network:** a small MLP (e.g. `48·48·48`), `(x,y)` → `(R,G,B)`, sigmoid output, MSE loss.
- **Fourier features (positional encoding):** the input `(x,y)` is expanded into
  `sin/cos` at several frequencies. Without this, an MLP can only produce smooth blurs
  (a.k.a. *spectral bias*) — turn it off in the UI and you'll see the mush. With it, a small net
  gets surprisingly sharp.
- **Adam optimizer** for fast convergence (plain SGD fits images painfully slowly).
- **Minibatch** of random pixels per step; the reconstruction is re-rendered every couple of frames.

## Controls
- **Network size** — bigger = sharper, slower.
- **Fourier features** — off / low / medium / high (the sharpness knob).
- **Activation** — tanh / ReLU.
- **Learning rate**, **steps per frame**, **play/pause**, **reset**.

## Run locally
It must be served over HTTP (so the browser can read the image pixels), not opened as `file://`:
```bash
python3 -m http.server
# then open http://localhost:8000/
```

Single self-contained `index.html` + `oliver.png`. Oliver artwork by **Nero** 💙.
