# HTML & CSS Cheatsheet for React

## Table of Contents

- [HTML \& CSS Cheatsheet for React](#html--css-cheatsheet-for-react)
	- [Table of Contents](#table-of-contents)
	- [HTML Elements in JSX](#html-elements-in-jsx)
		- [1. Semantic Structure](#1-semantic-structure)
		- [2. Common HTML Elements](#2-common-html-elements)
		- [3. Form Elements](#3-form-elements)
		- [4. Tables](#4-tables)
		- [5. Media Elements](#5-media-elements)
	- [CSS Fundamentals](#css-fundamentals)
		- [6. Selectors](#6-selectors)
		- [7. Box Model](#7-box-model)
		- [8. Flexbox (Layout)](#8-flexbox-layout)
		- [9. Grid Layout](#9-grid-layout)
		- [10. Typography](#10-typography)
		- [11. Colors \& Backgrounds](#11-colors--backgrounds)
		- [12. Positioning](#12-positioning)
		- [13. Display Properties](#13-display-properties)
		- [14. Transitions \& Animations](#14-transitions--animations)
		- [15. Shadows \& Effects](#15-shadows--effects)
		- [16. Responsive Design](#16-responsive-design)
	- [CSS in React](#css-in-react)
		- [17. Styling Methods](#17-styling-methods)
		- [18. Common Layout Patterns](#18-common-layout-patterns)
		- [19. Form Styling](#19-form-styling)
		- [20. Dashboard-Specific Styles](#20-dashboard-specific-styles)
	- [Modern CSS Features](#modern-css-features)
		- [21. CSS Variables (Custom Properties)](#21-css-variables-custom-properties)
		- [22. Modern Selectors](#22-modern-selectors)
		- [23. Units](#23-units)
	- [Quick Reference](#quick-reference)
		- [24. Common Patterns](#24-common-patterns)
		- [25. Accessibility](#25-accessibility)
	- [Browser DevTools Tips](#browser-devtools-tips)
	- [Resources](#resources)

## HTML Elements in JSX

### 1. Semantic Structure

```jsx
// Page layout
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>

<main>
  <section>
    <article>
      <h1>Title</h1>
      <p>Content</p>
    </article>
  </section>

  <aside>Sidebar content</aside>
</main>

<footer>
  <p>&copy; 2025 Company</p>
</footer>
```

### 2. Common HTML Elements

```jsx
// Headings (h1 largest, h6 smallest)
<h1>Main Title</h1>
<h2>Section Title</h2>
<h3>Subsection</h3>

// Text elements
<p>Paragraph text</p>
<span>Inline text</span>
<strong>Bold text</strong>
<em>Italic text</em>
<small>Small text</small>

// Links
<a href="https://example.com">External Link</a>
<a href="/dashboard" target="_blank">New Tab</a>

// Lists
<ul>
  <li>Unordered item 1</li>
  <li>Unordered item 2</li>
</ul>

<ol>
  <li>Ordered item 1</li>
  <li>Ordered item 2</li>
</ol>

// Images
<img src="/path/to/image.jpg" alt="Description" width="300" height="200" />

// Divisions
<div className="container">Block element</div>
```

### 3. Form Elements

```jsx
// Text inputs
<input type="text" placeholder="Enter name" />
<input type="email" placeholder="email@example.com" />
<input type="password" placeholder="Password" />
<input type="number" min="0" max="100" step="0.01" />
<input type="date" />
<input type="time" />

// Textarea
<textarea rows="4" cols="50" placeholder="Enter description"></textarea>

// Checkboxes
<input type="checkbox" id="agree" />
<label htmlFor="agree">I agree</label>

// Radio buttons
<input type="radio" name="option" value="yes" id="yes" />
<label htmlFor="yes">Yes</label>
<input type="radio" name="option" value="no" id="no" />
<label htmlFor="no">No</label>

// Select dropdown
<select>
  <option value="">Select...</option>
  <option value="btc">Bitcoin</option>
  <option value="eth">Ethereum</option>
</select>

// Buttons
<button type="submit">Submit</button>
<button type="button" onClick={handleClick}>Click Me</button>
<button type="reset">Reset</button>
```

### 4. Tables

```jsx
<table>
  <thead>
    <tr>
      <th>Market</th>
      <th>Price</th>
      <th>Volume</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>BTC</td>
      <td>$67,000</td>
      <td>1.2M</td>
    </tr>
    <tr>
      <td>ETH</td>
      <td>$2,500</td>
      <td>800K</td>
    </tr>
  </tbody>
</table>
```

### 5. Media Elements

```jsx
// Video
<video controls width="640" height="360">
  <source src="video.mp4" type="video/mp4" />
</video>

// Audio
<audio controls>
  <source src="audio.mp3" type="audio/mpeg" />
</audio>

// Iframe
<iframe src="https://example.com" width="600" height="400" title="Embedded content"></iframe>
```

## CSS Fundamentals

### 6. Selectors

```css
/* Element selector */
p { color: blue; }

/* Class selector */
.container { width: 100%; }

/* ID selector (avoid in React, use classes) */
#header { background: red; }

/* Multiple classes */
.btn.primary { background: blue; }

/* Descendant selector */
.container p { margin: 10px; }

/* Child selector (direct children only) */
.container > p { margin: 10px; }

/* Pseudo-classes */
button:hover { background: #ccc; }
input:focus { border-color: blue; }
li:first-child { font-weight: bold; }
li:last-child { border: none; }
li:nth-child(2) { color: red; }

/* Pseudo-elements */
p::before { content: "→ "; }
p::after { content: " ←"; }
```

### 7. Box Model

**What is the Box Model?**
Every HTML element is a rectangular box made of layers. Understanding this is crucial for spacing and sizing.

**The layers (from inside out):**
1. **Content** - The actual text/image
2. **Padding** - Space inside the border (background color shows here)
3. **Border** - Line around the padding
4. **Margin** - Space outside the border (pushes other elements away)

**Visual:**
```
┌─────────────── Margin (transparent, pushes others away)
│ ┌───────────── Border (visible line)
│ │ ┌─────────── Padding (space inside, has background color)
│ │ │ ┌───────── Content (text/image)
│ │ │ │
│ │ │ │  Hello
│ │ │ └─────────
│ │ └───────────
│ └─────────────
└───────────────
```

```css
.box {
  /* Content size */
  width: 300px;
  height: 200px;

  /* Padding: Space INSIDE (background color fills here) */
  padding: 20px;                    /* All 4 sides */
  padding: 10px 20px;               /* Vertical | Horizontal */
  padding: 10px 20px 30px 40px;     /* Top | Right | Bottom | Left (clockwise) */
  padding-top: 10px;                /* Individual sides */

  /* Border: Line around the box */
  border: 2px solid #333;           /* width | style | color */
  border-width: 2px;
  border-style: solid;              /* solid, dashed, dotted, none */
  border-color: #333;
  border-radius: 8px;               /* Rounded corners */
  border-top-left-radius: 16px;    /* Individual corner */

  /* Margin: Space OUTSIDE (pushes other elements away) */
  margin: 20px;                     /* All 4 sides */
  margin: 0 auto;                   /* Top/bottom: 0, Left/right: auto = center */
  margin-bottom: 30px;              /* Individual sides */

  /* Box sizing: How is width calculated? */
  box-sizing: content-box;          /* width = content only (default) */
  box-sizing: border-box;           /* width = content + padding + border (RECOMMENDED) */
}
```

**Important:** Always use `box-sizing: border-box` to make sizing predictable.

**Example:**
```css
/* Without border-box: Total width = 300 + 40 + 4 = 344px */
.box-old {
  width: 300px;
  padding: 20px;  /* +20px left, +20px right */
  border: 2px;    /* +2px left, +2px right */
}

/* With border-box: Total width = 300px (padding/border included) */
.box-new {
  box-sizing: border-box;
  width: 300px;   /* This is the FINAL width */
  padding: 20px;
  border: 2px;
}
```

### 8. Flexbox (Layout)

**What is Flexbox?**
Flexbox is a one-dimensional layout system for arranging items in rows OR columns. Think of it as a smart container that automatically distributes space between items.

**When to use:**
- Navigation bars
- Button groups
- Centering content
- Form inputs in a row
- Card layouts that need equal heights

**Mental model:** Parent container controls how children are arranged (like organizing books on a shelf).

```css
/* Container (the parent) */
.flex-container {
  display: flex;              /* Turn on flexbox */

  /* Direction: Which way do items flow? */
  flex-direction: row;        /* → Horizontal (default) */
  flex-direction: column;     /* ↓ Vertical */

  /* justify-content: Align along MAIN axis (horizontal if row, vertical if column) */
  justify-content: flex-start;   /* Items at start ←[1][2][3]      */
  justify-content: flex-end;     /* Items at end        [1][2][3]→ */
  justify-content: center;       /* Items centered    [1][2][3]    */
  justify-content: space-between;/* Max space  [1]   [2]   [3]    */
  justify-content: space-around; /* Equal space  [1] [2] [3]      */

  /* align-items: Align along CROSS axis (perpendicular to main) */
  align-items: stretch;       /* Fill height (default) */
  align-items: flex-start;    /* Align to top */
  align-items: flex-end;      /* Align to bottom */
  align-items: center;        /* Vertically center */

  /* Wrapping: What happens when items don't fit? */
  flex-wrap: nowrap;          /* Squish items to fit (default) */
  flex-wrap: wrap;            /* Wrap to next row/column */

  /* Gap between items (modern way) */
  gap: 20px;                  /* Space between all items */
}

/* Items (the children) */
.flex-item {
  /* How much space should this item take? */
  flex: 1;                    /* Take equal space with siblings */
  flex: 2;                    /* Take 2x more space than flex: 1 items */

  /* Manual control (rarely needed) */
  flex-grow: 1;               /* Can grow to fill space */
  flex-shrink: 0;             /* Cannot shrink smaller than content */
  flex-basis: 200px;          /* Starting size before growing/shrinking */

  /* Override parent's align-items for this one item */
  align-self: flex-end;
}
```

**Real example:**
```css
/* Horizontal navbar */
.navbar {
  display: flex;
  justify-content: space-between; /* Logo left, links right */
  align-items: center;            /* Vertically center */
}

/* Three equal-width columns */
.columns {
  display: flex;
  gap: 20px;
}
.column {
  flex: 1; /* Each takes equal space */
}
```

### 9. Grid Layout

**What is Grid?**
Grid is a two-dimensional layout system for arranging items in rows AND columns simultaneously. Think of it as creating a table-like structure without using `<table>`.

**When to use:**
- Page layouts (header, sidebar, main, footer)
- Image galleries
- Dashboards with multiple cards
- Any complex 2D layout
- When you need precise control over rows AND columns

**Mental model:** Draw an invisible grid on the page, then place items into the cells.

```css
.grid-container {
  display: grid;              /* Turn on grid */

  /* Define columns: How many columns and their widths? */
  grid-template-columns: 200px 1fr 200px;  /* 3 cols: fixed, flexible, fixed */
  grid-template-columns: repeat(3, 1fr);    /* 3 equal columns */
  grid-template-columns: 1fr 2fr;           /* 2 cols: second is 2x wider */

  /* Auto-responsive: Creates as many columns as fit */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  /* ↑ "Make columns at least 250px, fit as many as possible" */

  /* Define rows: How tall are rows? */
  grid-template-rows: 100px auto 100px;     /* 3 rows: fixed, auto, fixed */
  grid-template-rows: repeat(3, 200px);     /* 3 rows of 200px each */

  /* Gaps between cells */
  gap: 20px;                  /* Same gap for rows and columns */
  row-gap: 20px;              /* Vertical spacing */
  column-gap: 10px;           /* Horizontal spacing */

  /* Alignment inside cells */
  justify-items: center;      /* Horizontal alignment of items */
  align-items: center;        /* Vertical alignment of items */
}

/* Grid items: Place items in specific cells */
.grid-item {
  /* Span across multiple columns */
  grid-column: 1 / 3;         /* Start at column 1, end at 3 (spans 2 cols) */
  grid-column: span 2;        /* Span 2 columns from current position */

  /* Span across multiple rows */
  grid-row: 1 / 3;            /* Start at row 1, end at 3 (spans 2 rows) */

  /* Shorthand: row-start / col-start / row-end / col-end */
  grid-area: 1 / 1 / 3 / 3;   /* Top-left to middle */
}
```

**Real example:**
```css
/* Dashboard layout */
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr;  /* Sidebar + main content */
  grid-template-rows: 60px 1fr 40px; /* Header + content + footer */
  gap: 20px;
  height: 100vh;
}

.header { grid-column: 1 / 3; }  /* Span both columns */
.sidebar { grid-row: 2; }        /* Just middle row */
.main { grid-row: 2; }
.footer { grid-column: 1 / 3; }  /* Span both columns */

/* Responsive card grid */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  /* Automatically creates 1, 2, 3, or 4 columns based on screen width */
}
```

### 10. Typography

```css
.text {
  /* Font */
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  font-weight: 400;           /* 100-900, or normal, bold */
  font-style: italic;
  line-height: 1.5;           /* Line spacing */

  /* Text properties */
  color: #333;
  text-align: center;         /* left, right, center, justify */
  text-decoration: underline; /* none, underline, line-through */
  text-transform: uppercase;  /* lowercase, uppercase, capitalize */
  letter-spacing: 2px;
  word-spacing: 5px;

  /* Overflow */
  white-space: nowrap;        /* Don't wrap */
  overflow: hidden;
  text-overflow: ellipsis;    /* Show ... when text overflows */
}
```

### 11. Colors & Backgrounds

```css
.element {
  /* Color formats */
  color: red;
  color: #ff0000;
  color: rgb(255, 0, 0);
  color: rgba(255, 0, 0, 0.5);     /* Alpha for transparency */
  color: hsl(0, 100%, 50%);
  color: hsla(0, 100%, 50%, 0.5);

  /* Background */
  background-color: #f0f0f0;
  background-image: url('/path/to/image.jpg');
  background-size: cover;          /* contain, cover, 100% auto */
  background-position: center;
  background-repeat: no-repeat;

  /* Gradient */
  background: linear-gradient(to right, #ff0000, #0000ff);
  background: linear-gradient(45deg, #ff0000 0%, #0000ff 100%);
  background: radial-gradient(circle, #ff0000, #0000ff);
}
```

### 12. Positioning

**What is positioning?**
Controls WHERE an element appears on the page. By default, elements flow naturally (top to bottom). Positioning lets you override this.

**Five position types:**

1. **`static` (default)** - Normal flow, can't use top/left/right/bottom
2. **`relative`** - Offset from its normal position (leaves gap where it was)
3. **`absolute`** - Positioned relative to nearest positioned ancestor (removed from flow)
4. **`fixed`** - Positioned relative to viewport, stays during scroll (removed from flow)
5. **`sticky`** - Acts like `relative` until scrolling threshold, then acts like `fixed`

```css
/* Static: Normal flow (default) */
.static {
  position: static;
  /* top/left/right/bottom have NO effect */
}

/* Relative: Nudge from normal position */
.relative {
  position: relative;
  top: 10px;     /* Move 10px DOWN from normal position */
  left: 20px;    /* Move 20px RIGHT from normal position */
  /* Original space is PRESERVED (ghost box remains) */
}

/* Absolute: Position relative to nearest positioned parent */
.absolute {
  position: absolute;
  top: 0;        /* 0px from top of positioned parent */
  right: 0;      /* 0px from right of positioned parent */
  /* Removed from normal flow (doesn't affect other elements) */
  /* Parent must have position: relative/absolute/fixed */
}

/* Fixed: Position relative to viewport (browser window) */
.fixed {
  position: fixed;
  bottom: 20px;  /* Always 20px from bottom of screen */
  right: 20px;   /* Always 20px from right of screen */
  /* Stays in place when scrolling */
}

/* Sticky: Relative until scroll threshold, then fixed */
.sticky {
  position: sticky;
  top: 0;        /* When scrolling past, stick to top */
  /* Useful for headers that stick when scrolling */
}

/* Z-index: Stacking order (which element is on top?) */
.overlay {
  position: absolute;  /* Must be positioned for z-index to work */
  z-index: 100;        /* Higher = more on top (default is 0) */
}
```

**Real examples:**
```css
/* Dropdown menu positioned below button */
.dropdown-container {
  position: relative;  /* Reference point for .dropdown-menu */
}
.dropdown-menu {
  position: absolute;
  top: 100%;          /* Right below the button */
  left: 0;
}

/* Fixed chat button in corner */
.chat-button {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1000;      /* Always on top */
}

/* Sticky header */
.header {
  position: sticky;
  top: 0;             /* Sticks to top when scrolling */
  background: white;
  z-index: 100;
}
```

### 13. Display Properties

```css
/* Display types */
.block { display: block; }          /* Full width, new line */
.inline { display: inline; }        /* Only content width, same line */
.inline-block { display: inline-block; } /* Inline but accepts width/height */
.flex { display: flex; }
.grid { display: grid; }
.none { display: none; }            /* Hide element */

/* Visibility */
.hidden { visibility: hidden; }     /* Hide but keep space */
.visible { visibility: visible; }

/* Overflow */
.overflow {
  overflow: hidden;                 /* auto, scroll, visible, hidden */
  overflow-x: scroll;
  overflow-y: auto;
}
```

### 14. Transitions & Animations

```css
/* Transitions */
.button {
  background-color: blue;
  transition: background-color 0.3s ease;
  transition: all 0.3s ease-in-out;
}

.button:hover {
  background-color: darkblue;
}

/* Animations */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideIn {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(0); }
}

.animated {
  animation: fadeIn 1s ease-in;
  animation: slideIn 0.5s ease-out forwards;
  animation: spin 2s linear infinite;
}

/* Transform */
.transform {
  transform: translateX(50px);      /* Move */
  transform: rotate(45deg);         /* Rotate */
  transform: scale(1.5);            /* Scale */
  transform: skew(10deg);           /* Skew */
  transform: translate(50px, 100px) rotate(45deg); /* Combine */
}
```

### 15. Shadows & Effects

```css
.effects {
  /* Box shadow */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1), 0 1px 3px rgba(0, 0, 0, 0.08);

  /* Text shadow */
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);

  /* Opacity */
  opacity: 0.8;

  /* Blur filter */
  filter: blur(5px);
  filter: grayscale(100%);
  filter: brightness(1.2);
}
```

### 16. Responsive Design

```css
/* Mobile-first approach */
.container {
  width: 100%;
  padding: 10px;
}

/* Tablet */
@media (min-width: 768px) {
  .container {
    width: 750px;
    padding: 20px;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .container {
    width: 1000px;
    padding: 30px;
  }
}

/* Common breakpoints */
@media (max-width: 480px) { /* Mobile */ }
@media (min-width: 481px) and (max-width: 768px) { /* Tablet */ }
@media (min-width: 769px) and (max-width: 1024px) { /* Desktop */ }
@media (min-width: 1025px) { /* Large Desktop */ }

/* Orientation */
@media (orientation: landscape) { }
@media (orientation: portrait) { }
```

## CSS in React

### 17. Styling Methods

```jsx
// 1. External CSS file
import './App.css';
<div className="container">Content</div>

// 2. CSS Modules (scoped)
import styles from './App.module.css';
<div className={styles.container}>Content</div>

// 3. Inline styles (object)
const style = {
  color: 'red',
  fontSize: '16px',
  backgroundColor: '#f0f0f0'
};
<div style={style}>Content</div>

// 4. Conditional classes
<div className={isActive ? 'active' : 'inactive'}>Content</div>
<div className={`base ${isActive ? 'active' : ''}`}>Content</div>

// 5. Multiple classes
<div className="container primary large">Content</div>
```

### 18. Common Layout Patterns

```css
/* Centered container */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* Card component */
.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 20px;
}

/* Flexbox centering */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

/* Two-column layout */
.two-columns {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

/* Responsive grid */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

/* Sticky header */
.header {
  position: sticky;
  top: 0;
  background: white;
  z-index: 100;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
```

### 19. Form Styling

```css
/* Input fields */
input, select, textarea {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
}

/* Buttons */
button {
  padding: 10px 20px;
  font-size: 16px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.2s;
}

button:hover {
  background: #0056b3;
}

button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* Form layout */
.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.form-group input {
  width: 100%;
}
```

### 20. Dashboard-Specific Styles

```css
/* Metrics cards */
.metric-card {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.metric-label {
  font-size: 14px;
  color: #666;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 10px;
}

.metric-value {
  font-size: 32px;
  font-weight: bold;
  color: #333;
}

/* Data table */
table {
  width: 100%;
  border-collapse: collapse;
  background: white;
}

th {
  background: #f8f9fa;
  padding: 12px;
  text-align: left;
  font-weight: 600;
  border-bottom: 2px solid #dee2e6;
}

td {
  padding: 12px;
  border-bottom: 1px solid #dee2e6;
}

tr:hover {
  background: #f8f9fa;
}

/* Loading spinner */
.spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Alert messages */
.alert {
  padding: 12px 20px;
  border-radius: 4px;
  margin-bottom: 20px;
}

.alert-success {
  background: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.alert-error {
  background: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}

.alert-warning {
  background: #fff3cd;
  color: #856404;
  border: 1px solid #ffeaa7;
}
```

## Modern CSS Features

### 21. CSS Variables (Custom Properties)

```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --spacing-unit: 8px;
  --border-radius: 4px;
  --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.button {
  background: var(--primary-color);
  padding: calc(var(--spacing-unit) * 2);
  border-radius: var(--border-radius);
  font-family: var(--font-family);
}

/* Override in component */
.dark-theme {
  --primary-color: #0056b3;
}
```

### 22. Modern Selectors

```css
/* :is() - grouping */
:is(h1, h2, h3) {
  margin-top: 0;
}

/* :where() - zero specificity */
:where(ul, ol) li {
  margin-bottom: 10px;
}

/* :not() - exclusion */
button:not(.secondary) {
  background: blue;
}

/* :has() - parent selector (newer) */
.card:has(img) {
  display: grid;
}
```

### 23. Units

```css
/* Absolute */
px      /* Pixels */
pt      /* Points */

/* Relative */
em      /* Relative to parent font-size */
rem     /* Relative to root font-size */
%       /* Percentage of parent */
vw      /* Viewport width (1vw = 1% of viewport width) */
vh      /* Viewport height */
vmin    /* Smaller of vw or vh */
vmax    /* Larger of vw or vh */

/* Examples */
.example {
  font-size: 16px;
  width: 50%;
  padding: 1rem;           /* 16px if root is 16px */
  margin: 2em;             /* 32px (2 × parent 16px) */
  height: 100vh;           /* Full viewport height */
}
```

## Quick Reference

### 24. Common Patterns

```css
/* Full-width button */
.btn-full { width: 100%; display: block; }

/* Truncate text */
.truncate {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Aspect ratio box */
.aspect-16-9 {
  aspect-ratio: 16 / 9;
}

/* Hide on mobile */
@media (max-width: 768px) {
  .hide-mobile { display: none; }
}

/* Smooth scrolling */
html { scroll-behavior: smooth; }

/* Remove default styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

### 25. Accessibility

```css
/* Focus visible (keyboard navigation) */
*:focus-visible {
  outline: 2px solid #007bff;
  outline-offset: 2px;
}

/* Screen reader only */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}

/* Skip to main content */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: white;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

## Browser DevTools Tips

1. **Inspect Element**: Right-click → Inspect
2. **Edit CSS live**: Modify styles in DevTools
3. **Toggle classes**: Check/uncheck in Elements panel
4. **Computed styles**: See final applied styles
5. **Box model visualization**: Margins, padding, borders
6. **Color picker**: Click color squares to choose colors
7. **Responsive mode**: Test different screen sizes

## Resources

- [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS-Tricks](https://css-tricks.com/)
- [Can I Use](https://caniuse.com/) - Browser compatibility
- [Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Grid Guide](https://css-tricks.com/snippets/css/complete-guide-grid/)
