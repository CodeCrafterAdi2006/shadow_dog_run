# 👾 NPC & Enemy Movement Patterns — Complete Progress & Knowledge Guide

A permanent, comprehensive reference guide detailing every chronological stage, mathematical formula, object-oriented design decision, and rendering mechanic for **Enemy Movement / NPC Animations** in HTML5 Canvas and Vanilla JavaScript.

---

## 📑 Table of Contents
1. [Project Overview & Architecture](#-1-project-overview--architecture)
2. [File Structure & Component Roles](#-2-file-structure--component-roles)
3. [Chronological Implementation Stages (All Stages Preserved)](#-3-chronological-implementation-stages-all-stages-preserved)
   - [Stage 1: Single Enemy Baseline (Manual State Mutation)](#-stage-1-single-enemy-baseline-manual-state-mutation)
   - [Stage 2: Multi-Entity Swarm & `strokeRect` Wireframe](#-stage-2-multi-entity-swarm--strokerect-wireframe)
   - [Stage 3: Sprite Sheet Integration & 9-Argument `drawImage()`](#-stage-3-sprite-sheet-integration--9-argument-drawimage)
   - [Stage 4: Instance Image Encapsulation & Asynchronous Flap Staggering](#-stage-4-instance-image-encapsulation--asynchronous-flap-staggering)
   - [Stage 5: Movement Pattern 1 — Organic Jitter / Hover & Boundary-Safe Spawning (Enemy 1 Final)](#-stage-5-movement-pattern-1--organic-jitter--hover--boundary-safe-spawning-enemy-1-final)
   - [Stage 6: Enemy Type 2 — Horizontal Flight & Screen Wrapping](#-stage-6-enemy-type-2--horizontal-flight--screen-wrapping)
   - [Stage 7: Movement Pattern 2 — Amplitude-Scaled Sine Wave Swoop (Enemy 2 Final)](#-stage-7-movement-pattern-2--amplitude-scaled-sine-wave-swoop-enemy-2-final)
   - [Stage 8: Movement Pattern 3 — Parametric Sine & Cosine Orbit (Enemy 3 Initial)](#-stage-8-movement-pattern-3--parametric-sine--cosine-orbit-enemy-3-initial)
   - [Stage 9: Full-Canvas Lissajous Flow (3:1 Frequency Ratio & 200-Enemy Stream - Enemy 3 Final)](#-stage-9-full-canvas-lissajous-flow-31-frequency-ratio--200-enemy-stream---enemy-3-final)
   - [Stage 10: Movement Pattern 4 — Autonomous Target Relocation & Fractional Easing (Enemy 4 Final State)](#-stage-10-movement-pattern-4--autonomous-target-relocation--fractional-easing-enemy-4-final-state)
4. [Mastering `ctx.drawImage()`: The 9-Parameter Formula](#-4-mastering-ctxdrawimage-the-9-parameter-formula)
5. [🎯 Physics Deep Dive: Fractional Easing / Damping (`dx / divisor`)](#-5-physics-deep-dive-fractional-easing--damping-dx--divisor)
6. [🎡 The Geometric Secret: How `Math.sin()` and `Math.cos()` Create Complex Patterns](#-6-the-geometric-secret-how-mathsin-and-mathcos-create-complex-patterns)
7. [🎛️ The Tinkerer's Playground: 8+ Radical Movement Patterns by Changing 1 Line](#-7-the-tinkerers-playground-8-radical-movement-patterns-by-changing-1-line)
8. [Mathematical Deep Dive & Reference Formulas](#-8-mathematical-deep-dive--reference-formulas)
9. [Animation Staggering Math: Global `gameFrame` & Modulo (`%`) Throttling](#-9-animation-staggering-math-global-gameframe--modulo--throttling)
10. [Visual Techniques: `strokeRect` vs `fillRect` vs `drawImage`](#-10-visual-techniques-strokerect-vs-fillrect-vs-drawimage)
11. [Array-Based Entity Lifecycle & Batch Processing](#-11-array-based-entity-lifecycle--batch-processing)
12. [Critical Bugs & Debugging Insights (Including `NaN` Bug Analysis)](#-12-critical-bugs--debugging-insights-including-nan-bug-analysis)
13. [Complete Current Code Listing](#-13-complete-current-code-listing)
14. [Reusable Blueprint for Future Game Projects](#-14-reusable-blueprint-for-future-game-projects)

---

## 📌 1. Project Overview & Architecture

This project builds an autonomous **Non-Player Character (NPC) / Enemy Movement & Sprite Animation Engine** using HTML5 Canvas and Object-Oriented JavaScript.

### Core Architectural Pillars:
- **Object-Oriented Design (OOP)**: Encapsulates state (`x`, `y`, `newX`, `newY`, `interval`, `width`, `height`, `frame`, `flapSpeed`, `image`) and behaviors (`update()`, `draw()`) inside an `Enemy` class.
- **Autonomous Target Seeking (Periodic Relocation)**: Enemies continuously select new randomized targets on the canvas every `50` to `250` frames.
- **Fractional Easing / Damping**: Employs delta distance division (`this.x -= dx / 70`) to create naturalistic ease-out acceleration (darting fast when distant, decelerating smoothly as target approaches).
- **Sprite Sheet Slicing**: Dynamically crops 6 animation frames per character from multi-character spritesheet assets (`enemy1.png`, `enemy2.png`, `enemy3.png`, `enemy4.png`).
- **60 FPS Game Loop**: Continuous canvas clearing (`clearRect`), batch state updates (`update`), batch rendering (`draw`), and frame progression (`gameFrame++`) synchronized via `requestAnimationFrame()`.

```
                        ┌──────────────────────────────────────────────┐
                        │              Main Animation Loop             │
                        │                                              │
                        │  1. ctx.clearRect(0, 0, width, height)       │
                        │  2. enemiesArray.forEach(enemy => {          │
                        │         enemy.update(); // Ease to (newX,newY│
                        │         enemy.draw();   // Crop & draw sprite│
                        │     })                                       │
                        │  3. gameFrame++;        // Advance frame tick│
                        │  4. requestAnimationFrame(animate)           │
                        └──────────────────────────────────────────────┘
```

---

## 📁 2. File Structure & Component Roles

| File | Purpose | Key Responsibilities |
|---|---|---|
| [`index.html`](file:///c:/Web_dev_master/js_game_dev/enemy_movement/index.html) | Viewport Structure | Declares `<canvas id="canvas1">` viewport and imports style and script assets. |
| [`style.css`](file:///c:/Web_dev_master/js_game_dev/enemy_movement/style.css) | Presentation & Layout | Centers the canvas in the viewport (`position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%)`) and defines a distinct `3px solid black` boundary. |
| [`script.js`](file:///c:/Web_dev_master/js_game_dev/enemy_movement/script.js) | Engine Logic & Rendering | Loads sprite assets, defines `class Enemy`, populates `enemiesArray`, and drives the 60 FPS animation loop. |
| `enemy1.png` | Sprite Asset 1 | 6-frame bat sprite sheet (`293x155px` per frame). |
| `enemy2.png` | Sprite Asset 2 | 6-frame flying monster sprite sheet (`266x188px` per frame). |
| `enemy3.png` | Sprite Asset 3 | 6-frame floating ghost sprite sheet (`218x177px` per frame). |
| `enemy4.png` | Sprite Asset 4 | 6-frame spinning/saw blade creature sprite sheet (`213x213px` per frame). |

---

## 🛠️ 3. Chronological Implementation Stages (All Stages Preserved)

### 🔹 Stage 1: Single Enemy Baseline (Manual State Mutation)
- Created a single instance `const enemy1 = new Enemy()` with fixed starting coordinates `(x: 10, y: 50)`.
- The animation loop manually mutated coordinates (`enemy1.x++`, `enemy1.y++`) and called `ctx.fillRect()`.

---

### 🔹 Stage 2: Multi-Entity Swarm & `strokeRect` Wireframe
- **Class-Level Methods**: Added `update()` and `draw()` directly inside `class Enemy`.
- **Mass Spawning**: Instantiated 100 unique `Enemy` objects in an array (`enemiesArray`).
- **Randomized Positions & Speeds**: Range `[-2, +2]`.
- **Wireframe Rendering (`strokeRect`)**: Replaced solid boxes with transparent outlines.

---

### 🔹 Stage 3: Sprite Sheet Integration & 9-Argument `drawImage()`
- **Asset Loading**: Created image objects and loaded `enemy1.png`.
- **Sprite Dimensions**: `spriteWidth = 293`, `spriteHeight = 155`, scaled by `/ 3`.
- **Frame Cycling via Ternary Operator**: `this.frame > 4 ? this.frame = 0 : this.frame++;`.
- **9-Argument `ctx.drawImage`**: Cropped and rendered frame slices dynamically.

---

### 🔹 Stage 4: Instance Image Encapsulation & Asynchronous Flap Staggering
- **Per-Instance Image Ownership**: `this.image = new Image(); this.image.src = 'enemy1.png';`.
- **Global `gameFrame` Counter**: Increments by `1` every tick.
- **Randomized `flapSpeed`**: `this.flapSpeed = Math.floor(Math.random() * 3 + 1);`.
- **Modulo-Based Animation Throttling**: `if (gameFrame % this.flapSpeed === 0)`.

---

### 🔹 Stage 5: Movement Pattern 1 — Organic Jitter / Hover & Boundary-Safe Spawning (Enemy 1 Final)
- **Scaled Swarm Size**: `numberOfEnemies = 10`.
- **Boundary-Safe Spawning**: `this.x = Math.random() * (canvas.width - this.width);`.
- **Brownian Jitter Math**: `this.x += Math.random() * 5 - 2.5; this.y += Math.random() * 5 - 2.5;`.

---

### 🔹 Stage 6: Enemy Type 2 — Horizontal Flight & Screen Wrapping
- **Asset Switch (`enemy2.png`)**: Frame dimensions `266x188px`.
- **Unidirectional Speed Range**: `this.speed = Math.random() * 4 + 1;` (`[1, 5]` px/frame).
- **Seamless Screen Wrap**: `if (this.x + this.width < 0) this.x = canvas.width;`.

---

### 🔹 Stage 7: Movement Pattern 2 — Amplitude-Scaled Sine Wave Swoop (Enemy 2 Final)
- **Physics**:
  ```javascript
  this.x -= this.speed;
  this.y += Math.sin(this.angle) * this.curve;
  this.angle += this.angleSpeed;
  ```

---

### 🔹 Stage 8: Movement Pattern 3 — Parametric Sine & Cosine Orbit (Enemy 3 Initial)
- **Asset Switch (`enemy3.png`)**: Frame dimensions `218x177px`, scaled by `/ 2`.
- Parametric circular equations using fixed radius `this.curve`.

---

### 🔹 Stage 9: Full-Canvas Lissajous Flow (3:1 Frequency Ratio & 200-Enemy Stream - Enemy 3 Final)
- **Swarm Density**: `numberOfEnemies = 200`.
- **3:1 Asymmetric Frequency Lissajous Wave**:
  ```javascript
  this.x = canvas.width / 2 * Math.cos(this.angle * Math.PI / 90) + canvas.width / 2 - this.width / 2;
  this.y = canvas.height / 2 * Math.sin(this.angle * Math.PI / 270) + canvas.height / 2 - this.height / 2;
  ```

---

### 🔹 Stage 10: Movement Pattern 4 — Autonomous Target Relocation & Fractional Easing (Enemy 4 Final State)
- **Asset Switch (`enemy4.png`)**: Frame dimensions `213x213px`, scaled by `/ 2` (`width: 106.5px`, `height: 106.5px`).
- **Swarm Size**: `numberOfEnemies = 20`.
- **Independent Random Target Destination & Interval Timer in `constructor()`**:
  ```javascript
  this.newX = Math.random() * (canvas.width - this.width);
  this.newY = Math.random() * (canvas.height - this.height);
  this.interval = Math.floor(Math.random() * 200 + 50); // 50 to 250 frames (approx 1 to 4 sec)
  ```
- **Autonomous Retargeting & Fractional Easing in `update()`**:
  ```javascript
  if (gameFrame % this.interval === 0) {
      this.newX = Math.random() * (canvas.width - this.width);
      this.newY = Math.random() * (canvas.height - this.height);
  }
  let dx = this.x - this.newX;
  let dy = this.y - this.newY;
  this.x -= dx / 70;
  this.y -= dy / 70;
  ```
- **Visual Impact**: Enemies dart dynamically across the canvas, accelerating rapidly toward a chosen spot, then gracefully slowing down (easing out) as they arrive, before choosing a new point. Creates lifelike, intelligent predatory or curious hovering behavior!

---

## 🖼️ 4. Mastering `ctx.drawImage()`: The 9-Parameter Formula

```javascript
ctx.drawImage(
    this.image, 
    this.frame * this.spriteWidth, 0, 
    this.spriteWidth, this.spriteHeight, 
    this.x, this.y, 
    this.width, this.height
);
```

---

## 🎯 5. Physics Deep Dive: Fractional Easing / Damping (`dx / divisor`)

### Why does `this.x -= dx / 70` produce smooth deceleration?

1. **Distance Vector**:
   - `let dx = this.x - this.newX;` calculates the exact distance remaining between the enemy and its target.
2. **Fractional Step (Zeno's Paradox Principle)**:
   - On each frame, the enemy moves by $\frac{1}{70}\text{th}$ ($\approx 1.43\%$) of the remaining distance.
   - **Far from target** (`dx = 350px`): Moves $\frac{350}{70} = \mathbf{5.0\text{ px/frame}}$ (Fast dash).
   - **Midway** (`dx = 140px`): Moves $\frac{140}{70} = \mathbf{2.0\text{ px/frame}}$ (Moderate glide).
   - **Near target** (`dx = 14px`): Moves $\frac{14}{70} = \mathbf{0.2\text{ px/frame}}$ (Smooth ease-out stop).

```
Start (dx = 350px)                                               Target (newX, newY)
[Enemy] ═══════► ═════► ════► ═══► ══► ═► ► . . . [★]
       (Fast Dash)          (Gentle Deceleration)   (Soft Landing)
```

> [!TIP]
> **Tuning the Divisor**:
> - `dx / 20`: Snappy, aggressive darting (fast chaser).
> - `dx / 70`: Smooth, graceful floating (current saw blade / ghost).
> - `dx / 150`: Lazy, dreamy underwater drifting.

---

## 🎡 6. The Geometric Secret: How `Math.sin()` and `Math.cos()` Create Complex Patterns

- `cos` (horizontal) and `sin` (vertical) are 90° out of phase.
- Adjusting frequency divisors (e.g. `Math.PI / 90` vs `Math.PI / 270`) generates complex **Lissajous Curves** and multi-lobe geometric orbits.

---

## 🎛️ 7. The Tinkerer's Playground: 8+ Radical Movement Patterns by Changing 1 Line

| Pattern | Code Tweak in `update()` | Game Feel / Use Case |
|---|---|---|
| **1. Perfect Circle** | `x = 200*cos(a); y = 200*sin(a);` | Orbiting shields, planetary drones |
| **2. Oval / Ellipse** | `x = 200*cos(a); y = 80*sin(a);` | Stretched 3D orbital ring |
| **3. 3:1 Lissajous Stream** | `x = W/2*cos(a/90); y = H/2*sin(a/270);` | Full-canvas 3-loop river |
| **4. 2:1 Figure-8 (`∞`)** | `x = 200*cos(a); y = 200*sin(2*a);` | Flying fairies, evasive bosses |
| **5. 3-Leaf Floral Knot** | `x = 200*cos(2*a); y = 200*sin(3*a);` | 3-petal flower dance |
| **6. Diagonal Pendulum** | `x = 200*sin(a); y = 200*sin(a);` | Straight diagonal swing |
| **7. Expanding Spiral** | `radius += 0.5; x = r*cos(a); y = r*sin(a);` | Whirlpools, black hole pull |
| **8. Periodic Target Easing** (Current) | `dx = x - newX; x -= dx / 70;` | Intelligent curious/stalker enemies |

---

## 🧮 8. Mathematical Deep Dive & Reference Formulas

| Formula Pattern | Geometry Produced | Game Application |
|---|---|---|
| `x += vx; y += vy;` | Linear Straight Line | Projectiles, chargers, lasers |
| `x += Math.random() - 0.5;` | Brownian Micro-Jitter | Flying insects (Enemy 1), fluttering bats |
| `x -= speed; y += sin(a) * curve;` | Wavy S-Curve / Swoop | Swooping predators (Enemy 2), birds, fish |
| `x = cos(a)*r; y = sin(a)*r;` | Circular Orbit | Rotating shields, ghost orbs |
| `x = cos(a/90)*W; y = sin(a/270)*H;` | 3:1 Lissajous Wave | Multi-loop canvas stream (Enemy 3) |
| `dx = x - newX; x -= dx / 70;` | Exponential Target Easing | Intelligent roaming enemies (Enemy 4 Final) |

---

## ⏱️ 9. Animation Staggering Math: Global `gameFrame` & Modulo (`%`) Throttling

```javascript
if (gameFrame % this.flapSpeed === 0) {
    this.frame > 4 ? this.frame = 0 : this.frame++;
}
```

---

## 🎨 10. Visual Techniques: `strokeRect` vs `fillRect` vs `drawImage`

| Method | Visual Output | Use Case |
|---|---|---|
| **`ctx.fillRect()`** | Solid black shape | Prototyping single objects / hitbox solids |
| **`ctx.strokeRect()`** | Transparent wireframe box | Debugging swarms & collision boundaries |
| **`ctx.drawImage()`** | Full-color animated sprite | Production game graphics & animated characters |

---

## 🔄 11. Array-Based Entity Lifecycle & Batch Processing

```
[ Instantiate ] ──> for (i = 0..20) enemiesArray.push(new Enemy())
                           │
                           ▼
[ Game Loop ]   ──> enemiesArray.forEach(enemy => {
                         enemy.update(); // Ease toward target point
                         enemy.draw();   // Render sprite frame
                     })
                     gameFrame++;
```

---

## ⚠️ 12. Critical Bugs & Debugging Insights (Including `NaN` Bug Analysis)

### 🐛 1. The `NaN` (Not a Number) Invisible Sprite Bug
* **Cause**: `undefined * Math.cos(...)` produces `NaN`.
* **Fix**: Ensure all multiplier variables have default fallbacks or use canvas dimensions directly.

### 🐛 2. Radian vs Degree Conversion
* **Fix**: Always multiply degrees by `Math.PI / 180`.

---

## 📄 13. Complete Current Code Listing

```javascript
/** @type {HTMLCanvasElement} */
const canvas = document.getElementById('canvas1')
const ctx = canvas.getContext('2d')
const CANVAS_WIDTH = canvas.width = 500
const CANVAS_HEIGHT = canvas.height = 1000
const numberOfEnemies = 20;
const enemiesArray = [];

let gameFrame = 0;

class Enemy {
    constructor() {
        this.image = new Image();
        this.image.src = 'enemy4.png';
        this.speed = Math.random() * 4 + 1;
        this.spriteWidth = 213;
        this.spriteHeight = 213;
        this.width = this.spriteWidth / 2;
        this.height = this.spriteHeight / 2;
        this.x = Math.random() * (canvas.width - this.width);
        this.y = Math.random() * (canvas.height - this.height);
        this.newX = Math.random() * (canvas.width - this.width);
        this.newY = Math.random() * (canvas.height - this.height);
        this.frame = 0;
        this.flapSpeed = Math.floor(Math.random() * 3 + 1);
        this.interval = Math.floor(Math.random() * 200 + 50);
    }
    update() {
        if (gameFrame % this.interval === 0) {
            this.newX = Math.random() * (canvas.width - this.width);
            this.newY = Math.random() * (canvas.height - this.height);
        }
        let dx = this.x - this.newX;
        let dy = this.y - this.newY;
        this.x -= dx / 70;
        this.y -= dy / 70;
        // this.x = 0;
        // this.y = 0;
        if (this.x + this.width < 0) {
            this.x = canvas.width;
        }
        if (gameFrame % this.flapSpeed === 0) {
            this.frame > 4 ? this.frame = 0 : this.frame++;
        }
    }
    draw() {
        ctx.drawImage(this.image, this.frame * this.spriteWidth, 0, this.spriteWidth, this.spriteHeight, this.x, this.y, this.width, this.height);
    }
};


for (let i = 0; i < numberOfEnemies; i++) {
    enemiesArray.push(new Enemy());
}


function animate() {
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    enemiesArray.forEach(enemy => {
        enemy.update();
        enemy.draw();
    })
    gameFrame++;
    requestAnimationFrame(animate);
}
animate();
```

---

## 🏗️ 14. Reusable Blueprint for Future Game Projects

```javascript
// 1. Canvas Setup
const canvas = document.getElementById('canvas1');
const ctx = canvas.getContext('2d');
const CANVAS_WIDTH = canvas.width = 500;
const CANVAS_HEIGHT = canvas.height = 1000;
let gameFrame = 0;

// 2. Intelligent Target-Seeking Entity Blueprint
class RoamingEnemy {
    constructor(imageSrc, spriteW, spriteH, scale, totalFrames, easeFactor = 70) {
        this.image = new Image();
        this.image.src = imageSrc;
        this.spriteWidth = spriteW;
        this.spriteHeight = spriteH;
        this.width = this.spriteWidth / scale;
        this.height = this.spriteHeight / scale;
        this.x = Math.random() * (canvas.width - this.width);
        this.y = Math.random() * (canvas.height - this.height);
        this.newX = Math.random() * (canvas.width - this.width);
        this.newY = Math.random() * (canvas.height - this.height);
        this.frame = 0;
        this.totalFrames = totalFrames;
        this.flapSpeed = Math.floor(Math.random() * 3 + 1);
        this.interval = Math.floor(Math.random() * 200 + 50);
        this.easeFactor = easeFactor;
    }
    update() {
        // Periodic Target Selection
        if (gameFrame % this.interval === 0) {
            this.newX = Math.random() * (canvas.width - this.width);
            this.newY = Math.random() * (canvas.height - this.height);
        }
        // Fractional Distance Easing
        let dx = this.x - this.newX;
        let dy = this.y - this.newY;
        this.x -= dx / this.easeFactor;
        this.y -= dy / this.easeFactor;

        // Animation cycling
        if (gameFrame % this.flapSpeed === 0) {
            this.frame >= this.totalFrames - 1 ? this.frame = 0 : this.frame++;
        }
    }
    draw() {
        ctx.drawImage(
            this.image,
            this.frame * this.spriteWidth, 0,
            this.spriteWidth, this.spriteHeight,
            this.x, this.y,
            this.width, this.height
        );
    }
}

// 3. Spawning & Animation Loop
const enemies = Array.from({ length: 20 }, () => new RoamingEnemy('enemy4.png', 213, 213, 2, 6, 70));

function animate() {
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    enemies.forEach(enemy => {
        enemy.update();
        enemy.draw();
    });
    gameFrame++;
    requestAnimationFrame(animate);
}
animate();
```
