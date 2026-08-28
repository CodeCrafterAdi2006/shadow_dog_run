# 🌌 2D Parallax Scrolling Background — Progress & Knowledge Guide

A comprehensive guide and reference document covering the core concepts, math, architecture, and step-by-step evolution of the **Parallax Background** project in HTML5 Canvas and JavaScript.

---

## 📌 1. Project Overview & Architecture

### What is Parallax Scrolling?
**Parallax scrolling** is a visual technique where background layers move at different speeds relative to the foreground, creating an illusion of 3D depth in a 2D game environment.
- **Distant layers** (e.g., sky, far mountains) move **slowly**.
- **Middle layers** (e.g., trees, hills) move at a **moderate speed**.
- **Foreground layers** (e.g., ground, close structures) move **fastest**.

```
[Layer 1: Sky / Clouds]    ──> Speed: 0.2x (Moves very slowly)
[Layer 2: Distant Hills]   ──> Speed: 0.4x
[Layer 3: Middle Trees]    ──> Speed: 0.6x
[Layer 4: Foreground Bush] ──> Speed: 0.8x
[Layer 5: Front Ground]    ──> Speed: 1.0x (Moves with game speed)
```

---

## 📁 2. File Structure & Responsibilities

| File | Purpose | Key Responsibilities |
|---|---|---|
| [`index.html`](file:///c:/Web_dev_master/js_game_dev/parallax_background/index.html) | HTML Structure | Defines the `<canvas id="canvas1">` viewport and imports scripts/stylesheets. |
| [`styles.css`](file:///c:/Web_dev_master/js_game_dev/parallax_background/styles.css) | Canvas Styling | Centers the canvas on the screen with a dark backdrop. |
| [`script.js`](file:///c:/Web_dev_master/js_game_dev/parallax_background/script.js) | Core Game Loop & Logic | Loads image assets, calculates layer offsets, manages redraw loop (`requestAnimationFrame`). |
| `layer-1.png` to `layer-5.png` | Visual Assets | 5 distinct png layers representing depth planes (typically 2400x700px or similar). |

---

## 🛠️ 3. Current Progress & Code Breakdown

### Step 1: Canvas Initialization & Layout
In [`script.js`](file:///c:/Web_dev_master/js_game_dev/parallax_background/script.js):
```javascript
const canvas = document.getElementById('canvas1');
const ctx = canvas.getContext('2d');
const CANVAS_WIDTH = canvas.width = 800;
const CANVAS_HEIGHT = canvas.height = 700;
let gameSpeed = 5;
```
* **`canvas.width` / `canvas.height`**: Sets the internal coordinate resolution of the canvas.
* **`ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT)`**: Clears the canvas buffer every frame before drawing new positions to prevent visual smearing.

In [`styles.css`](file:///c:/Web_dev_master/js_game_dev/parallax_background/styles.css):
```css
#canvas1 {
    position: absolute;
    border: 3px solid white;
    width: 800px;
    height: 700px;
    transform: translate(-50%, -50%); /* Correct 2D centering */
    top: 50%;
    left: 50%;
}
```

---

## 📐 4. Mathematical Foundations of Infinite Scrolling

### The Problem: Running Out of Image
When an image moves left (`x -= gameSpeed`), it eventually scrolls off the screen (`x < -width`), leaving a blank void on the canvas.

### The Solution: The Two-Image Wrapping Pattern
To create a seamless, infinite loop:
1. **Draw Image A** at `x`.
2. **Draw Image B (duplicate)** immediately following it at `x + width` (or `x2`).
3. Once `x <= -width`, reset `x = 0` (or `x = width + x2 - gameSpeed`).

```
Frame 0:
[   Canvas Viewport   ]
[   Image 1 (x = 0)   ][ Image 2 (x2 = 2400) ]

Frame N (scrolling left):
         [   Canvas Viewport   ]
...[ Image 1 (x = -800) ][ Image 2 (x2 = 1600) ]
```

---

## 🏗️ 5. Moving from Procedural to Object-Oriented Architecture (OOP)

Managing 5 separate layers with loose variables (`x1`, `x2`, `speed1`, `speed2`) quickly becomes messy and error-prone. The standard game dev pattern is to encapsulate each layer inside a `Layer` class.

### Recommended `Layer` Class Pattern:
```javascript
class Layer {
    constructor(image, speedModifier) {
        this.x = 0;
        this.y = 0;
        this.width = 2400; // Native width of the layer image
        this.height = 700;
        this.image = image;
        this.speedModifier = speedModifier;
        this.speed = gameSpeed * this.speedModifier;
    }

    update() {
        this.speed = gameSpeed * this.speedModifier;
        if (this.x <= -this.width) {
            this.x = 0;
        }
        this.x = Math.floor(this.x - this.speed);
    }

    draw() {
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
        ctx.drawImage(this.image, this.x + this.width, this.y, this.width, this.height);
    }
}
```

### Initializing the Layers:
```javascript
const layer1 = new Layer(backgroundLayer1, 0.2);
const layer2 = new Layer(backgroundLayer2, 0.4);
const layer3 = new Layer(backgroundLayer3, 0.6);
const layer4 = new Layer(backgroundLayer4, 0.8);
const layer5 = new Layer(backgroundLayer5, 1.0);

const gameObjects = [layer1, layer2, layer3, layer4, layer5];

function animate() {
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    gameObjects.forEach(object => {
        object.update();
        object.draw();
    });
    requestAnimationFrame(animate);
}
```

---

## ⚠️ 6. Common Pitfalls & Key Learnings

1. **Pixel Gaps / Seams Between Tiles**:
   - Using floating-point values for `x` can cause sub-pixel rounding artifacts in canvas rendering, creating a 1px flickering line between seamless images.
   - **Fix**: Use `Math.floor()` when calculating `this.x`.

2. **Image Loading Race Condition**:
   - Calling `animate()` before images have finished downloading will result in blank frames or skipped draws.
   - **Fix**: Place `animate()` inside a `window.addEventListener('load', ...)` block.

3. **Dynamic Speed Scaling**:
   - If `gameSpeed` is adjusted dynamically (e.g. by a UI slider or player movement), layers must recalculate `this.speed = gameSpeed * this.speedModifier` inside `update()`.

---

## 🚀 7. Future Extension Ideas

- [ ] **UI Game Speed Slider**: Add an `<input type="range">` in HTML to dynamically speed up, slow down, or reverse the background in real-time.
- [ ] **Character Sprite Integration**: Place an animated running character in the foreground on top of the moving background.
- [ ] **Day/Night Cycle Tinting**: Use `ctx.globalCompositeOperation` or a semi-transparent colored rectangle overlay to transition between daytime, sunset, and night.
- [ ] **Vertical Parallax**: Add slight Y-offset shifts synchronized with player jumps or camera elevation.
