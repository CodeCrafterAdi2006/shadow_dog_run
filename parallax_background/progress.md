# 🌌 2D Parallax Scrolling Background — Complete Progress & Knowledge Guide

A permanent reference document capturing every stage of learning, code evolution, mathematical proofs, architectural decisions, and future milestones for the **HTML5 Canvas Parallax Background** project.

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
| [`index.html`](file:///c:/Web_dev_master/js_game_dev/parallax_background/index.html) | HTML Structure | Defines the `<canvas id="canvas1">` viewport and loads stylesheets and scripts. |
| [`styles.css`](file:///c:/Web_dev_master/js_game_dev/parallax_background/styles.css) | Canvas Styling | Centers the canvas on the screen with a dark backdrop. |
| [`script.js`](file:///c:/Web_dev_master/js_game_dev/parallax_background/script.js) | Core Game Loop & Logic | Manages image assets, calculates layer offsets, and executes `requestAnimationFrame`. |
| `layer-1.png` to `layer-5.png` | Visual Assets | 5 distinct 2400x700px PNG layers representing depth planes. |

---

## 🛠️ 3. Chronological Implementation Stages

### 🔹 Stage 1: Canvas Initialization & Centering Setup
- Initialized Canvas 2D context (`CANVAS_WIDTH = 800`, `CANVAS_HEIGHT = 700`).
- Fixed CSS transform bug in [`styles.css`](file:///c:/Web_dev_master/js_game_dev/parallax_background/styles.css) from `translate(-50%, 50%)` to `translate(-50%, -50%)` to keep the canvas properly centered in the viewport.

---

### 🔹 Stage 2: Single Image Movement (The "Left Void" Problem)
```javascript
let x = 0;
function animate() {
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    ctx.drawImage(backgroundLayer4, x, 0);
    x -= gameSpeed;
    requestAnimationFrame(animate);
}
```
* **Problem**: As `x` moves left past `-800px`, the canvas runs out of image on the right, leaving a blank void.

---

### 🔹 Stage 3: Two-Image Offset Loop (The Naive Approach & The Gap Bug)
To eliminate the void, two identical image instances were drawn side-by-side:
```javascript
let x = 0;
let x2 = 2400;

function animate() {
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    ctx.drawImage(backgroundLayer4, x, 0);
    ctx.drawImage(backgroundLayer4, x2, 0);
    
    if (x < -2400) x = 2400;
    else x -= gameSpeed;
    
    if (x2 < -2400) x2 = 2400;
    else x2 -= gameSpeed;
    
    requestAnimationFrame(animate);
};
```

#### 🔍 Deep Dive: Why the Naive Reset Creates an Empty Gap
1. **The `else` Branch Frame-Skip**:
   - When `x < -2400`, `x` resets to `2400`. Because the `if` branch ran, the `else` branch is skipped.
   - For that single frame, `x` **does not move by `gameSpeed`**, while `x2` **does move by `gameSpeed`** (`-5px`).
   - At the end of that frame:
     - `x = 2400`
     - `x2 = -5`
     - Right edge of `x2` = `-5 + 2400 = 2395px`
     - Left edge of `x` = `2400px`
     - **Gap created** = `2400 - 2395 = 5px` (exactly equal to `gameSpeed`).

---

### 🔹 Stage 3B: Fixing the Gap (Relative Offset Formula)
Instead of resetting to a hardcoded `2400`, the resetting image is dynamically snapped to the companion image's right edge:
```javascript
if (x < -2400) x = 2400 + x2 - gameSpeed;
else x -= gameSpeed;

if (x2 < -2400) x2 = 2400 + x - gameSpeed;
else x2 -= gameSpeed;
```
* **Why it works**: `2400 + x2` attaches `x` directly to `x2`'s edge, and `- gameSpeed` ensures it never skips a frame of movement.

---

### 🔹 Stage 4: Object-Oriented Architecture (`Layer` Class) (Current State)
Managing loose variables (`x1`, `x2`, `speed1`, `speed2`) for 5 separate layers becomes unmaintainable. All layer state and animation logic are encapsulated into an OOP `Layer` class:

```javascript
class Layer {
    constructor(image, speedModifier) {
        this.x = 0;
        this.y = 0;
        this.width = 2400;
        this.height = 700;
        this.x2 = this.width;
        this.image = image;
        this.speedModifier = speedModifier;
        this.speed = gameSpeed * this.speedModifier;
    }

    update() {
        // Dynamic speed adaptation when global gameSpeed changes
        this.speed = gameSpeed * this.speedModifier;

        // Relative reset eliminates tears and gaps
        if (this.x <= -this.width) {
            this.x = this.width + this.x2 - this.speed;
        }
        if (this.x2 <= -this.width) {
            this.x2 = this.width + this.x - this.speed;
        }

        // Math.floor prevents sub-pixel rendering blur and flickering seams
        this.x = Math.floor(this.x - this.speed);
        this.x2 = Math.floor(this.x2 - this.speed);
    }

    draw() {
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
        ctx.drawImage(this.image, this.x2, this.y, this.width, this.height);
    }
}
```

#### Instantiating All Layers:
```javascript
const layer1 = new Layer(backgroundLayer1, 0.5);
const layer2 = new Layer(backgroundLayer2, 0.5);
const layer3 = new Layer(backgroundLayer3, 0.5);
const layer4 = new Layer(backgroundLayer4, 0.5);
const layer5 = new Layer(backgroundLayer5, 0.1);
```

---

---

### 🔹 Stage 5: Interactive UI Speed Slider & Batch Array Rendering (Current State)

#### 1. DOM UI Integration:
An HTML container with a slider and live speed label was added to [`index.html`](file:///c:/Web_dev_master/js_game_dev/parallax_background/index.html):
```html
<div id="container">
    <canvas id="canvas1"></canvas>
    <p>Game Speed: <span id="showGameSpeed"></span></p>
    <input type="range" min="0" max="20" value="5" class="slider" id="slider">
</div>
```

#### 2. JavaScript Event Binding in [`script.js`](file:///c:/Web_dev_master/js_game_dev/parallax_background/script.js):
```javascript
const slider = document.getElementById('slider');
slider.value = gameSpeed;
const showGameSpeed = document.getElementById('showGameSpeed');
showGameSpeed.innerHTML = gameSpeed;

// Upgraded to 'input' event for real-time live scrubbing:
slider.addEventListener('input', function (e) {
    gameSpeed = e.target.value;
    showGameSpeed.innerHTML = gameSpeed;
});
```

#### 3. Batch Array Rendering:
All layer objects are grouped into a `gameObjects` array and processed in a clean `forEach` loop:
```javascript
const gameObjects = [layer1, layer2, layer3, layer4, layer5];

function animate() {
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    gameObjects.forEach(object => {
        object.update();
        object.draw();
    });
    gameFrame--;
    requestAnimationFrame(animate);
};

animate();
```

---

### 🔹 Stage 6: Global `gameFrame` & Modulo (`%`) Offset Technique (Current State)

Instead of stateful position accumulation (`x -= speed` with `if (x <= -width)` reset conditions), the position is computed purely as a mathematical function of a global frame counter:

```javascript
let gameFrame = 0;

class Layer {
    constructor(image, speedModifier) {
        this.x = 0;
        this.y = 0;
        this.width = 2400;
        this.height = 700;
        this.image = image;
        this.speedModifier = speedModifier;
        this.speed = gameSpeed * this.speedModifier;
    }

    update() {
        this.speed = gameSpeed * this.speedModifier;
        // Modulo wrapping: calculates offset strictly from global frame count
        this.x = gameFrame * this.speed % this.width;
    }

    draw() {
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
        ctx.drawImage(this.image, this.x + this.width, this.y, this.width, this.height);
    }
}
```

#### 🧮 Mathematical Breakdown of the Modulo Formula:
1. `gameFrame` decrements every tick (`0, -1, -2, -3, ...`).
2. `gameFrame * this.speed` gives total theoretical distance moved (e.g. `-100 * 5 = -500px`).
3. `% this.width` automatically binds the coordinate to the range `[-(width - 1), 0]`:
   - At `gameFrame * speed = -2399`: `this.x = -2399`.
   - At `gameFrame * speed = -2400`: `this.x = 0` (instant wrap without `if` checks!).
   - At `gameFrame * speed = -2405`: `this.x = -5`.
4. The companion image is drawn at `this.x + this.width`, maintaining zero-gap alignment.

#### ⚖️ Comparison: Stateful Accumulation vs. `gameFrame` Modulo

| Feature | Stateful Accumulation (`x -= speed`) | `gameFrame` Modulo (`gameFrame * speed % width`) |
|---|---|---|
| **Code Simplicity** | Requires tracking state and handling reset boundaries (`if (x <= -width)`). | Extremely concise (1-line calculation, no `if` statements). |
| **Variable Count** | Can require 2 position variables (`x`, `x2`) or manual reset. | Requires only 1 variable (`x`) + global `gameFrame`. |
| **Dynamic Speed Changes** | Smooth acceleration/deceleration without visual jumps. | May cause a visible jump if `gameSpeed` is changed mid-run (since `gameFrame * newSpeed` suddenly changes total offset). |

---

## 🔍 5. Technical Insights: `change` vs `input` Event Listeners
* **`change` Event**: Fires only when the user **releases the mouse** or confirms the slider position.
* **`input` Event**: Fires **continuously in real-time** as the slider thumb is being dragged, providing instantaneous visual feedback.

---

## 🚀 6. Next Planned Milestones

1. **Asset Loading Guard (`window.addEventListener('load')`)**:
   Wrap `animate()` inside `window.load` to guarantee all images are fully loaded before rendering frames.
2. **Layer Speed Tuning**:
   Configure unique `speedModifier`s across all 5 layers (e.g., `0.2`, `0.4`, `0.6`, `0.8`, `1.0`) so all background planes move with realistic depth.
3. **Sprite Character Integration**:
   Add an animated player character running over the scrolling background.


