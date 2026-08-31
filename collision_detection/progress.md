# 💥 2D Collision Detection & Trigger-Based Particle/Sprite Animations — Master Progress Guide

A permanent, comprehensive reference guide detailing 2D collision detection mathematics (**AABB & Pythagorean Circle Detection**), **Viewport-to-Canvas Coordinate Translation**, **Event-Driven Sprite Animations (`boom.png`)**, **Audio Effects Integration (`boom.wav`)**, **True Center Canvas Transformations & Rotation Math (`save`, `translate`, `rotate`, `restore`)**, **Array Mutation & Garbage Collection (`splice` + `i--`)**, and **Debugging Strategies with `console.log`**.

---

## 📑 Table of Contents
1. [Project Overview & Architecture](#-1-project-overview--architecture)
2. [File Structure & Component Roles](#-2-file-structure--component-roles)
3. [Deep Dive: Mathematical Collision Detection](#-3-deep-dive-mathematical-collision-detection)
   - [Part A: Rectangle vs. Rectangle (AABB Collision)](#part-a-rectangle-vs-rectangle-aabb-collision)
   - [Part B: Circle vs. Circle (Pythagorean Theorem & Sum of Radii)](#part-b-circle-vs-circle-pythagorean-theorem--sum-of-radii)
   - [⚡ Performance Optimization: Squaring Both Sides (Skipping `Math.sqrt`)](#-performance-optimization-squaring-both-sides-skipping-mathsqrt)
   - [Part C: Circle vs. Rectangle (Clamping Method)](#part-c-circle-vs-rectangle-clamping-method)
4. [Deep Dive: Event-Triggered Sprite Animations (`boom.png`)](#-4-deep-dive-event-triggered-sprite-animations-boompng)
   - [The Viewport vs. Canvas Coordinate Problem](#the-viewport-vs-canvas-coordinate-problem)
   - [The Magic of `canvas.getBoundingClientRect()`](#the-magic-of-canvasgetboundingclientrect)
   - [Frame Rate Throttling Math (`timer % 10 === 0`)](#frame-rate-throttling-math-timer--10--0)
   - [Sprite Sheet Cropping & 9-Argument `ctx.drawImage()`](#sprite-sheet-cropping--9-argument-ctxdrawimage)
5. [🎨 Canvas Transformation Deep Dive: True Center Rotation Math](#-5-canvas-transformation-deep-dive-true-center-rotation-math)
6. [🔊 Audio Integration in Web Games (`Audio()` & Autoplay Policies)](#-6-audio-integration-in-web-games-audio--autoplay-policies)
7. [🗑️ Array Mutation & Garbage Collection Math: Why `i--` After `splice` is Essential](#-7-array-mutation--garbage-collection-math-why-i---after-splice-is-essential)
8. [💡 Why `console.log` is a Superpower in Web & Game Development](#-8-why-consolelog-is-a-superpower-in-web--game-development)
9. [Line-by-Line Breakdown of the Current Code](#-9-line-by-line-breakdown-of-the-current-code)
10. [Chronological Implementation Stages (All Stages Preserved)](#-10-chronological-implementation-stages-all-stages-preserved)
11. [Complete Current Code Listing](#-11-complete-current-code-listing)
12. [Complete Production-Grade Reference Blueprint](#-12-complete-production-grade-reference-blueprint)

---

## 📌 1. Project Overview & Architecture

This project connects two fundamental concepts in game development:
1. **Collision Detection Math**: Detecting bounding intersections between shapes (AABB and Euclidean Circle-Circle).
2. **Interactive Trigger Animation & Audio Engine**: Spawning responsive visual effects (`boom.png`) and sound effects (`boom.wav`) on demand whenever a trigger/click occurs.

### Complete Event, Sound & Rendering Pipeline:

```
[ User Click Event ]              ──► window.addEventListener('click', createAnimation)
             │
             ▼
[ Coordinate Normalization ]      ──► positionX = e.x - canvasPosition.left
             │                        positionY = e.y - canvasPosition.top
             ▼
[ Entity Instantiation ]          ──► explosions.push(new Explosions(posX, posY))
             │                        - Sets pivot: this.x = x, this.y = y
             │                        - Random angle: this.angle = Math.random() * 6.2
             │                        - Audio setup: this.sound = new Audio('boom.wav')
             ▼
[ 60 FPS Animation Loop ]         ──► 1. ctx.clearRect(0, 0, width, height)
             │                        2. explosions[i].update(); 
             │                           - if (frame === 0) play sound!
             │                           - timer % 10 === 0 -> frame++
             │                        3. explosions[i].draw();   
             │                           - save -> translate(x,y) -> rotate(angle) 
             │                           - drawImage(-width/2, -height/2) -> restore
             ▼
[ Memory Cleanup (GC) ]           ──► if (frame > 5) { explosions.splice(i, 1); i--; }
```

---

## 📁 2. File Structure & Component Roles

| File | Purpose | Key Responsibilities |
|---|---|---|
| [`index.html`](file:///c:/Web_dev_master/js_game_dev/collision_detection/index.html) | Viewport Structure | Declares `<canvas id="canvas1">`, imports `<link rel="stylesheet" href="style.css">`, and loads `script.js`. |
| [`style.css`](file:///c:/Web_dev_master/js_game_dev/collision_detection/style.css) | Presentation & Layout | Centers the canvas via `#canvas1 { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: black; border: 3px solid black; }`. |
| [`script.js`](file:///c:/Web_dev_master/js_game_dev/collision_detection/script.js) | Engine Logic & Rendering | Manages canvas dimensions, tracks `getBoundingClientRect()`, instantiates `Explosions` with rotation and audio playback, renders transformations, and cleans up memory via `splice`. |
| `boom.png` | Visual FX Sprite Sheet | 5-frame horizontal explosion sprite sheet (`200x179px` per frame). |
| `boom.wav` | Audio SFX Asset | Explosion sound effect triggered synchronously on frame 0. |

---

## 📐 3. Deep Dive: Mathematical Collision Detection

### Part A: Rectangle vs. Rectangle (AABB Collision)

**AABB** (*Axis-Aligned Bounding Box*) tests whether two unrotated rectangles overlap on both axes.

#### The 4 Edges:
- **Left**: `rect.x`
- **Right**: `rect.x + rect.width`
- **Top**: `rect.y`
- **Bottom**: `rect.y + rect.height`

#### The 4 Conditions:
```javascript
function checkRectCollision(rect1, rect2) {
    return (
        rect1.x < rect2.x + rect2.width &&   // 1. rect1 left edge is left of rect2 right edge
        rect1.x + rect1.width > rect2.x &&   // 2. rect1 right edge is right of rect2 left edge
        rect1.y < rect2.y + rect2.height &&  // 3. rect1 top edge is above rect2 bottom edge
        rect1.y + rect1.height > rect2.y     // 4. rect1 bottom edge is below rect2 top edge
    );
}
```

```
       NO OVERLAP (Gap Exists)               COLLISION (Overlapping on both X & Y)
       
        rect1                                 rect1
       ┌──────┐                              ┌──────┬─────┐
       │      │                              │      │░░░░░│ ◄── Overlap Zone!
       └──────┘                              └──────┼─────┘
                   rect2                            │ rect2
                  ┌──────┐                          │
                  │      │                          └─────┘
                  └──────┘
```

---

### Part B: Circle vs. Circle (Pythagorean Theorem & Sum of Radii)

Two circles collide if and only if **the straight-line distance between their center points ($d$) is less than or equal to the sum of their radii ($r_1 + r_2$)**.

```
    NO COLLISION: distance > (r1 + r2)          COLLISION: distance < (r1 + r2)

         (r1)               (r2)                    (r1)       (r2)
       ╭──────╮           ╭──────╮                ╭──────╮   ╭──────╮
      │   c1   │═════════│   c2   │              │   c1  │══│   c2   │
       ╰──────╯  distance ╰──────╯                ╰──────┴───┴──────╯
                                                     Overlap Zone
```

#### The Right-Angled Triangle Construction:
- **Base**: $\Delta x = x_2 - x_1$
- **Height**: $\Delta y = y_2 - y_1$
- **Hypotenuse ($d$)**: $d = \sqrt{(\Delta x)^2 + (\Delta y)^2}$

```
        (x1, y1) [Circle 1 Center]
           ●─────────────────────────┐
           │ \                       │
           │  \                      │
   Δy =    │   \  Hypotenuse (d)     │
 (y2 - y1) │    \                    │
           │     \                   │
           └──────● (x2, y2) [Circle 2 Center]
             Δx = (x2 - x1)
```

---

### ⚡ Performance Optimization: Squaring Both Sides (Skipping `Math.sqrt`)

$$\text{distance} < r_1 + r_2 \iff (\Delta x)^2 + (\Delta y)^2 < (r_1 + r_2)^2$$

```javascript
// 🚀 High-Performance Circle Collision (Zero Square Root Calculation)
function checkCircleCollision(c1, c2) {
    const dx = c2.x - c1.x;
    const dy = c2.y - c1.y;
    const distanceSquared = dx * dx + dy * dy;
    const sumOfRadii = c1.radius + c2.radius;

    return distanceSquared < (sumOfRadii * sumOfRadii);
}
```

---

### Part C: Circle vs. Rectangle (Clamping Method)

```javascript
function checkCircleRectCollision(circle, rect) {
    const closestX = Math.max(rect.x, Math.min(circle.x, rect.x + rect.width));
    const closestY = Math.max(rect.y, Math.min(circle.y, rect.y + rect.height));
    const dx = circle.x - closestX;
    const dy = circle.y - closestY;
    return (dx * dx + dy * dy) < (circle.radius * circle.radius);
}
```

---

## 💥 4. Deep Dive: Event-Triggered Sprite Animations (`boom.png`)

### The Viewport vs. Canvas Coordinate Problem
`MouseEvent.x` and `MouseEvent.y` provide coordinates relative to the **browser window viewport**. When a canvas is centered with CSS `position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%)`, clicking at pixel $(100, 100)$ on screen does not equal $(100, 100)$ inside the canvas.

---

### The Magic of `canvas.getBoundingClientRect()`
`canvas.getBoundingClientRect()` returns the canvas's exact live bounding rectangle:
- `canvasPosition.left`: Distance from window's left edge to canvas's left edge.
- `canvasPosition.top`: Distance from window's top edge to canvas's top edge.

```javascript
let canvasPosition = canvas.getBoundingClientRect();

function createAnimation(e) {
    let positionX = e.x - canvasPosition.left;
    let positionY = e.y - canvasPosition.top;
    explosions.push(new Explosions(positionX, positionY));
}
```

---

### Frame Rate Throttling Math (`timer % 10 === 0`)

Without a timer, `this.frame++` runs every tick of `requestAnimationFrame` (60 times a second). A 5-frame explosion would finish in $5/60 \approx 0.08\text{ seconds}$ (too fast to see).

```javascript
update() {
    if (this.frame === 0) this.sound.play();
    this.timer++;
    if (this.timer % 10 === 0) {
        this.frame++;
    };
}
```
- Total animation duration = $\approx 0.8\text{ seconds}$ (Smooth, natural explosion).

---

### Sprite Sheet Cropping & 9-Argument `ctx.drawImage()`

```javascript
ctx.drawImage(
    this.image, 
    this.spriteWidth * this.frame, 0,  // Source crop X, Source crop Y
    this.spriteWidth, this.spriteHeight, // Source crop Width, Source crop Height
    0 - this.width / 2, 0 - this.height / 2, // Destination X, Y (Centered on transformed origin)
    this.width, this.height              // Destination Width, Destination Height
);
```

---

## 🎨 5. Canvas Transformation Deep Dive: True Center Rotation Math

By default, canvas transforms rotate the **entire canvas** around the global origin `(0, 0)` (the top-left corner of the canvas).

To achieve **flawless center-point rotation** directly at the mouse click location without off-center wobble:

```javascript
draw() {
    ctx.save();                             // 1. Save unrotated clean canvas state
    ctx.translate(this.x, this.y);          // 2. Shift origin (0, 0) directly to mouse click (x, y)
    ctx.rotate(this.angle);                 // 3. Rotate grid around (0, 0) by random angle
    ctx.drawImage(
        this.image, 
        this.spriteWidth * this.frame, 0, 
        this.spriteWidth, this.spriteHeight, 
        0 - this.width / 2,                 // 4. Draw starting at -half width
        0 - this.height / 2,                // 5. Draw starting at -half height
        this.width, this.height
    );
    ctx.restore();                          // 6. Restore unrotated canvas state
}
```

### Why `0 - this.width / 2` and `0 - this.height / 2` works:
1. `ctx.translate(this.x, this.y)` places $(0, 0)$ precisely at the mouse click point.
2. Drawing from `(-width/2, -height/2)` to `(+width/2, +height/2)` places the center of the sprite at $(0, 0)$.
3. Because the center is at $(0, 0)$, `ctx.rotate(this.angle)` spins the sprite **around its own center point** rather than swinging like a pendulum!

```
     (-w/2, -h/2) ┌──────────────┐
                  │              │
                  │   ● (0, 0)   │ ◄── Origin & Center of Rotation
                  │              │
                  └──────────────┘ (+w/2, +h/2)
```

---

## 🔊 6. Audio Integration in Web Games (`Audio()` & Autoplay Policies)

```javascript
this.sound = new Audio();
this.sound.src = 'boom.wav';

// Triggered on first frame:
if (this.frame === 0) this.sound.play();
```

### Web Audio Best Practices:
1. **Autoplay Policy Compliance**: Modern browsers block audio from playing automatically on page load. Because our audio is triggered inside a user interaction (`click` event listener), browser security grants permission instantly.
2. **Instant Polyphony (Concurrent Explosions)**: By assigning `this.sound = new Audio()` inside the `Explosions` constructor, every single click creates an independent audio channel. Rapidly clicking 5 times plays 5 overlapping explosion sounds without cutting each other off!

---

## 🗑️ 7. Array Mutation & Garbage Collection Math: Why `i--` After `splice` is Essential

```javascript
for (let i = 0; i < explosions.length; i++) {
    explosions[i].update();
    explosions[i].draw();
    if (explosions[i].frame > 5) {
        explosions.splice(i, 1);
        i--; // 👈 CRITICAL STEP!
    }
}
```

### Why is `i--` required?
When `explosions.splice(i, 1)` deletes an item:
1. All subsequent elements in the array **shift left by one index** (item at index `2` moves to `1`, item at index `3` moves to `2`).
2. The `for` loop executes `i++`.
3. If you do **not** decrement `i--`, the loop will advance to the next index and **completely skip updating/drawing the element that just shifted into index `i`**!
4. Adding `i--` neutralizes the loop's `i++`, ensuring the newly shifted element is processed correctly.

---

## 💡 8. Why `console.log` is a Superpower in Web & Game Development

1. **Debugging Viewport Offsets**:
   ```javascript
   console.log(`Mouse: (${e.x}, ${e.y}) | Canvas Offset: (${canvasPosition.left}, ${canvasPosition.top}) | Canvas Relative: (${positionX}, ${positionY})`);
   ```
2. **Monitoring Active Particles & Catching Memory Leaks**:
   ```javascript
   console.log('Active explosions count:', explosions.length);
   ```
3. **Validating Audio Asset Loading**:
   ```javascript
   this.sound.oncanplaythrough = () => console.log('boom.wav loaded successfully!');
   this.sound.onerror = () => console.error('Error loading boom.wav');
   ```

---

## 🔍 9. Line-by-Line Breakdown of the Current Code

```javascript
// 1. Reference canvas and 2D context
const canvas = document.getElementById('canvas1');
const ctx = canvas.getContext('2d');

// 2. Set internal drawing buffer resolution (500x700px)
canvas.width = 500;
canvas.height = 700;

// 3. Obtain initial screen bounding box of canvas
let canvasPosition = canvas.getBoundingClientRect();

// 4. Array to store active explosion instances
const explosions = [];

// 5. Explosion blueprint class
class Explosions {
    constructor(x, y) {
        this.spriteWidth = 200;
        this.spriteHeight = 179;
        this.width = this.spriteWidth * 0.7;    // 140px scaled width
        this.height = this.spriteHeight * 0.7;  // 125.3px scaled height
        this.x = x;                             // Click origin X
        this.y = y;                             // Click origin Y
        this.image = new Image;
        this.image.src = 'boom.png';
        this.frame = 0;                         // Frame tracker (0 to 5)
        this.timer = 0;                         // Frame throttling counter
        this.angle = Math.random() * 6.2;       // Random rotation (0 to ~2*PI radians)
        this.sound = new Audio();               // Audio object instance
        this.sound.src = 'boom.wav';            // Sound file path
    }
    update() {
        // Play sound effect exactly once when animation starts
        if (this.frame === 0) this.sound.play();
        this.timer++;
        // Advance frame once every 10 ticks (6 FPS sprite rate)
        if (this.timer % 10 === 0) {
            this.frame++;
        };
    }
    draw() {
        ctx.save();                             // Save clean canvas state
        ctx.translate(this.x, this.y);          // Move origin to click position
        ctx.rotate(this.angle);                 // Rotate canvas context
        ctx.drawImage(
            this.image, 
            this.spriteWidth * this.frame, 0, 
            this.spriteWidth, this.spriteHeight, 
            0 - this.width / 2,                 // Center image on X origin
            0 - this.height / 2,                // Center image on Y origin
            this.width, this.height
        );
        ctx.restore();                          // Restore clean canvas state
    }
}

// 6. Click event listener delegating to createAnimation helper
window.addEventListener('click', function (e) {
    createAnimation(e);
});

// 7. Normalizes mouse coordinates and spawns new explosion instance
function createAnimation(e) {
    let positionX = e.x - canvasPosition.left;
    let positionY = e.y - canvasPosition.top;
    explosions.push(new Explosions(positionX, positionY));
}

// 8. Continuous 60 FPS animation loop with array cleanup
function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    for (let i = 0; i < explosions.length; i++) {
        explosions[i].update();
        explosions[i].draw();
        // Remove dead explosion after frame 5 to prevent memory leaks
        if (explosions[i].frame > 5) {
            explosions.splice(i, 1);
            i--;                                // Adjust index to prevent skipping items
        }
    }
    requestAnimationFrame(animate);
}
animate();
```

---

## 🛠️ 10. Chronological Implementation Stages (All Stages Preserved)

### 🔹 Stage 1: Collision Detection Mathematics
- Derived AABB 4-condition bounding box logic.
- Derived Circle-Circle Pythagorean formula and squared-distance performance optimization.

### 🔹 Stage 2: Event Listener Prototyping
- Implemented `window.addEventListener('click', ...)` with raw `ctx.fillRect(e.x, e.y, 50, 50)`.

### 🔹 Stage 3: Coordinate Normalization & CSS Integration
- Linked `style.css` in `index.html` and corrected `#canvas` to `#canvas1`.
- Implemented `e.x - canvasPosition.left` and `e.y - canvasPosition.top` via `getBoundingClientRect()`.

### 🔹 Stage 4: Sprite Class Integration & Frame Throttling
- Created `Explosions` class using `boom.png` (`200x179px`).
- Scaled sprite to `70%` (`width * 0.7`).
- Implemented `this.timer % 10 === 0` to slow animation down to natural playback speed.

### 🔹 Stage 5: Coordinate Centering, True Center Rotation & Audio Feedback (Current Stage)
- Centered rotation origin using `ctx.translate(this.x, this.y)` and `drawImage(..., 0 - width/2, 0 - height/2)`.
- Added random rotation angle (`this.angle = Math.random() * 6.2`).
- Integrated `boom.wav` audio playback on frame 0.
- Modularized click creation into `createAnimation(e)`.
- Added garbage collection `splice(i, 1)` with index decrement `i--` when `frame > 5`.

---

## 📄 11. Complete Current Code Listing

```javascript
const canvas = document.getElementById('canvas1');
const ctx = canvas.getContext('2d');
canvas.width = 500;
canvas.height = 700;
let canvasPosition = canvas.getBoundingClientRect();

const explosions = [];

class Explosions {
    constructor(x, y) {
        this.spriteWidth = 200;
        this.spriteHeight = 179;
        this.width = this.spriteWidth * 0.7;
        this.height = this.spriteHeight * 0.7;
        this.x = x;
        this.y = y;
        this.image = new Image;
        this.image.src = 'boom.png';
        this.frame = 0;
        this.timer = 0;
        this.angle = Math.random() * 6.2;
        this.sound = new Audio();
        this.sound.src = 'boom.wav';
    }
    update() {
        if (this.frame === 0) this.sound.play();
        this.timer++;
        if (this.timer % 10 === 0) {
            this.frame++;
        };
    }
    draw() {
        ctx.save();
        ctx.translate(this.x, this.y);
        ctx.rotate(this.angle);
        ctx.drawImage(this.image, this.spriteWidth * this.frame, 0, this.spriteWidth, this.spriteHeight, 0 - this.width / 2, 0 - this.height / 2, this.width, this.height);
        ctx.restore();
    }
}

window.addEventListener('click', function (e) {
    createAnimation(e);
});

function createAnimation(e) {
    let positionX = e.x - canvasPosition.left;
    let positionY = e.y - canvasPosition.top;
    explosions.push(new Explosions(positionX, positionY));
}


function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    for (let i = 0; i < explosions.length; i++) {
        explosions[i].update();
        explosions[i].draw();
        if (explosions[i].frame > 5) {
            explosions.splice(i, 1);
            i--;
        }
    }
    requestAnimationFrame(animate);
    ;
}
animate();
```

---

## 🏗️ 12. Complete Production-Grade Reference Blueprint

```javascript
/** @type {HTMLCanvasElement} */
const canvas = document.getElementById('canvas1');
const ctx = canvas.getContext('2d');
canvas.width = 500;
canvas.height = 700;

// Dynamic boundary tracker for window resize/scroll
let canvasPosition = canvas.getBoundingClientRect();
window.addEventListener('resize', () => {
    canvasPosition = canvas.getBoundingClientRect();
});

const explosions = [];

class Explosion {
    constructor(x, y) {
        this.spriteWidth = 200;
        this.spriteHeight = 179;
        this.width = this.spriteWidth * 0.7;
        this.height = this.spriteHeight * 0.7;
        this.x = x;
        this.y = y;
        this.image = new Image();
        this.image.src = 'boom.png';
        this.frame = 0;
        this.timer = 0;
        this.angle = Math.random() * 6.28; // Full 360 degree random rotation
        this.sound = new Audio();
        this.sound.src = 'boom.wav';
    }
    update() {
        if (this.frame === 0 && this.timer === 0) {
            this.sound.play();
        }
        this.timer++;
        if (this.timer % 10 === 0) {
            this.frame++;
        }
    }
    draw() {
        ctx.save();
        // Translate directly to center of sprite for true center rotation
        ctx.translate(this.x, this.y);
        ctx.rotate(this.angle);
        ctx.drawImage(
            this.image,
            this.spriteWidth * this.frame, 0,
            this.spriteWidth, this.spriteHeight,
            -this.width / 2, -this.height / 2,
            this.width, this.height
        );
        ctx.restore();
    }
}

window.addEventListener('click', function (e) {
    createAnimation(e);
});

function createAnimation(e) {
    const positionX = e.x - canvasPosition.left;
    const positionY = e.y - canvasPosition.top;
    
    if (
        positionX >= 0 && positionX <= canvas.width &&
        positionY >= 0 && positionY <= canvas.height
    ) {
        explosions.push(new Explosion(positionX, positionY));
    }
}

function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    for (let i = 0; i < explosions.length; i++) {
        explosions[i].update();
        explosions[i].draw();
        
        // Memory cleanup: remove finished explosions
        if (explosions[i].frame > 5) {
            explosions.splice(i, 1);
            i--;
        }
    }
    requestAnimationFrame(animate);
}
animate();
```
