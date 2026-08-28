# 🎮 Sprite Animation & Game Loop Progress Guide

A complete, beginner-to-advanced reference guide covering everything learned while building the 2D Sprite Animation engine in Vanilla JavaScript.

---

## 📑 Table of Contents
1. [Phase 1: Canvas Foundations & Setup](#phase-1-canvas-foundations--setup)
2. [Phase 2: The Core Game Loop & Motion](#phase-2-the-core-game-loop--motion)
3. [Phase 3: Mastering `ctx.drawImage()`](#phase-3-mastering-ctxdrawimage)
4. [Phase 4: Frame Rate Control & Animation Staggering](#phase-4-frame-rate-control--animation-staggering)
5. [Phase 5: Advanced Math vs. Conditional Branching](#phase-5-advanced-math-vs-conditional-branching)
6. [Phase 6: Data Structures & Pre-Calculated Lookup Tables](#phase-6-data-structures--pre-calculated-lookup-tables)
7. [Phase 7: UI Controls & Dynamic State Management (Dropdown Integration)](#phase-7-ui-controls--dynamic-state-management-dropdown-integration)
8. [Complete Final Code Listing](#complete-final-code-listing)
9. [Quick Architecture Blueprint for Future Games](#quick-architecture-blueprint-for-future-games)

---

## Phase 1: Canvas Foundations & Setup

### HTML + CSS + JS Triad
A canvas cannot run on JavaScript alone; it requires an HTML element to mount onto and CSS for visual styling:
* **HTML (`index.html`)**: Defines `<canvas id="canvas1"></canvas>` and loads the script.
* **CSS (`style.css`)**: Centers the canvas on the screen and draws a border around it.

### Internal Drawing Resolution vs. CSS Display Size
* **`canvas.width` / `canvas.height`**: The internal drawing resolution (the actual pixel grid).
* **CSS `width` / `height`**: The scaled box size on the page.
* *Rule*: Always set `canvas.width` and `canvas.height` in JavaScript to keep your graphics crisp and prevent pixel blurring.

```javascript
const canvas = document.getElementById('canvas1');
const ctx = canvas.getContext('2d'); // Your paintbrush/toolkit

const CANVAS_WIDTH = canvas.width = 600;
const CANVAS_HEIGHT = canvas.height = 600;
```

---

## Phase 2: The Core Game Loop & Motion

To create the illusion of animation, the browser runs a continuous loop at your screen's refresh rate (~60 FPS):

```javascript
let x = 0;

function animate() {
    // 1. Clear previous frame
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

    // 2. Draw current state
    ctx.fillRect(x, 50, 100, 100);

    // 3. Update state for next frame
    x++;

    // 4. Request the browser to call animate() again on the next paint cycle
    requestAnimationFrame(animate);
}

animate();
```

* **`ctx.clearRect(0, 0, w, h)`**: Clears the canvas. Without this, drawn shapes leave continuous trails.
* **`requestAnimationFrame(callback)`**: High-performance browser API that synchronizes the loop with the monitor refresh rate.

---

## Phase 3: Mastering `ctx.drawImage()`

`ctx.drawImage()` has 3 distinct signatures:

### 1. 3 Arguments (Basic Placement)
Draws the entire image at native resolution at coordinate `(dx, dy)`:
```javascript
ctx.drawImage(image, dx, dy);
```

### 2. 5 Arguments (Placement + Scaling)
Draws and stretches/shrinks the entire image to fit `(dWidth, dHeight)`:
```javascript
ctx.drawImage(image, dx, dy, dWidth, dHeight);
```

### 3. 9 Arguments (Cropping / Spritesheet Slicing)
Crucial for 2D games. Crops a sub-rectangle from the spritesheet and draws it on the canvas.

$$\text{Source (Crop from sheet)} \longrightarrow \text{Destination (Draw on Canvas)}$$

```javascript
ctx.drawImage(
    image,
    sx, sy, sWidth, sHeight,  // SOURCE: where & how much to cut out
    dx, dy, dWidth, dHeight   // DESTINATION: where & how big to draw on canvas
);
```

---

## Phase 4: Frame Rate Control & Animation Staggering

By default, 60 FPS is too fast for sprite character animations. We slow it down by staggering frame updates using the **Modulo (`%`)** operator.

### Approach 1: State Mutation with `if/else`
```javascript
let frameX = 0;
let gameFrame = 0;
const staggerFrames = 6;

if (gameFrame % staggerFrames === 0) {
    if (frameX < 6) frameX++;
    else frameX = 0;
}
gameFrame++;
```

---

## Phase 5: Advanced Math vs. Conditional Branching

### Approach 2: Pure Cycle Math (`Math.floor` + `%`)
Replacing manual `if/else` checks with pure mathematical mapping:

```javascript
let position = Math.floor(gameFrame / staggerFrames) % totalFrames;
let frameX = spriteWidth * position;
```

### Why this is better:
1. **Deterministic & Stateless**: Any `gameFrame` number instantly produces the exact correct frame.
2. **No nested branching**: Cleaner, faster, and bug-free wrapping.
3. **Dynamic**: Easily swap `totalFrames` depending on the current character action.

---

## Phase 6: Data Structures & Pre-Calculated Lookup Tables

Instead of computing coordinates on the fly in the render loop, you pre-calculate every frame coordinate into an indexed lookup structure during startup.

### 1. The Configuration Array (All 10 Animation States)
```javascript
const animationStates = [
    { name: 'idle',   frames: 7 },  // Row 0
    { name: 'jump',   frames: 7 },  // Row 1
    { name: 'fall',   frames: 9 },  // Row 2
    { name: 'run',    frames: 9 },  // Row 3
    { name: 'dizzy',  frames: 11 }, // Row 4
    { name: 'sit',    frames: 5 },  // Row 5
    { name: 'roll',   frames: 7 },  // Row 6
    { name: 'bite',   frames: 7 },  // Row 7
    { name: 'ko',     frames: 12 }, // Row 8
    { name: 'getHit', frames: 4 }   // Row 9
];
```

### 2. Building the Lookup Dictionary
```javascript
const spriteAnimations = [];

animationStates.forEach((state, index) => {
    let frames = { loc: [] };
    for (let j = 0; j < state.frames; j++) {
        let positionX = j * spriteWidth;
        let positionY = index * spriteHeight;
        frames.loc.push({ x: positionX, y: positionY });
    }
    spriteAnimations[state.name] = frames;
});
```

---

## Phase 7: UI Controls & Dynamic State Management (Dropdown Integration)

To let users switch between animations dynamically without modifying code, we bind an HTML `<select>` dropdown to our JavaScript state.

### 1. HTML Dropdown (`index.html`)
```html
<div class="controls">
    <label for="animations">Choose animation:</label>
    <select id="animations" name="animations">
        <option value="idle">Idle</option>
        <option value="jump">Jump</option>
        <option value="fall">Fall</option>
        <option value="run">Run</option>
        <option value="dizzy">Dizzy</option>
        <option value="sit">Sit</option>
        <option value="roll">Roll</option>
        <option value="bite">Bite</option>
        <option value="ko">KO</option>
        <option value="getHit">Get Hit</option>
    </select>
</div>
```

### 2. Event Listener in JavaScript (`script.js`)
```javascript
let playerState = 'idle';
const dropdown = document.getElementById('animations');
dropdown.addEventListener('change', function (e) {
    playerState = e.target.value;
});
```

### 3. Truly Dynamic Animation Loop
Using `.loc.length` ensures that regardless of whether the animation has 4 frames (`getHit`) or 12 frames (`ko`), it loops accurately without running out of bounds:

```javascript
function animate() {
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

    // Dynamic modulo based on the current state's frame count
    let position = Math.floor(gameFrame / staggerFrames) % spriteAnimations[playerState].loc.length;

    // Direct lookup from the coordinate table
    let frameX = spriteAnimations[playerState].loc[position].x;
    let frameY = spriteAnimations[playerState].loc[position].y;

    ctx.drawImage(playerImage, frameX, frameY, spriteWidth, spriteHeight, 0, 0, spriteWidth, spriteHeight);

    gameFrame++;
    requestAnimationFrame(animate);
}
```

---

## Complete Final Code Listing

<details>
<summary><b>Click to view full <code>script.js</code></b></summary>

```javascript
let playerState = 'idle';
const dropdown = document.getElementById('animations');
dropdown.addEventListener('change', function (e) {
    playerState = e.target.value;
});

const canvas = document.getElementById('canvas1');
const ctx = canvas.getContext('2d');

const CANVAS_WIDTH = canvas.width = 600;
const CANVAS_HEIGHT = canvas.height = 600;

const playerImage = new Image();
playerImage.src = 'shadow_dog.png';
const spriteWidth = 575;
const spriteHeight = 523;

let gameFrame = 0;
const staggerFrames = 5;
const spriteAnimations = [];
const animationStates = [
    { name: 'idle', frames: 7 },
    { name: 'jump', frames: 7 },
    { name: 'fall', frames: 9 },
    { name: 'run', frames: 9 },
    { name: 'dizzy', frames: 11 },
    { name: 'sit', frames: 5 },
    { name: 'roll', frames: 7 },
    { name: 'bite', frames: 7 },
    { name: 'ko', frames: 12 },
    { name: 'getHit', frames: 4 }
];

animationStates.forEach((state, index) => {
    let frames = {
        loc: [],
    };
    for (let j = 0; j < state.frames; j++) {
        let positionX = j * spriteWidth;
        let positionY = index * spriteHeight;
        frames.loc.push({ x: positionX, y: positionY });
    }
    spriteAnimations[state.name] = frames;
});

function animate() {
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    let position = Math.floor(gameFrame / staggerFrames) % spriteAnimations[playerState].loc.length;
    let frameX = spriteAnimations[playerState].loc[position].x;
    let frameY = spriteAnimations[playerState].loc[position].y;
    ctx.drawImage(playerImage, frameX, frameY, spriteWidth, spriteHeight, 0, 0, spriteWidth, spriteHeight);

    gameFrame++;
    requestAnimationFrame(animate);
}

animate();
```
</details>

---

## Quick Architecture Blueprint for Future Games

To build your own 2D sprite-based games, use this standard pipeline:

1. **Asset Loading**: Load images (`new Image()`) and sound effects.
2. **Metadata Setup**: Define frame sizes, speeds, and state lists (`animationStates`).
3. **Preprocessor**: Build coordinate lookup tables (`spriteAnimations`).
4. **Input Listeners**: Listen to UI events or keyboard/gamepad inputs to switch `playerState`.
5. **Main Game Loop (`animate`)**:
   * Clear screen (`clearRect`)
   * Update physics/gameplay state
   * Render active sprite frame (`drawImage`)
   * Advance global tick counters (`gameFrame++`)
   * Request next frame (`requestAnimationFrame`)
