This document is useful for developers who are attempting to **upgrading from v4 to v5**. This includes gotchas and important context for understanding why your v4 code made need some subtle changes. In general, we've try to be as backward-compatible in v5 with the use of deprecation warnings in the `console`. There are, however, sometimes when changes are too substantial and require some additional help.

## API Changes

### Making WebGL First-Class

PixiJS v5 has made WebGL the first-class renderer and made CanvasRenderer to be second-class. Functionally, there's not much that changed from v4, but there are a bunch of subtle internal naming changes which could trip-up some developers upgrading to v5. For instance:

* `WebGLRenderer` becomes `Renderer`
* `renderWebGL` becomes `render` (in DisplayObject, Sprite, Container, etc)
* `_renderWebGL` becomes `_render` (in DisplayObject, Container, etc)

If you created a plugin or project that previously used `render` on a Container (see [#5510](https://github.com/pixijs/pixi.js/issues/5510)), this will probably cause your project to not render correctly. Please consider renaming your user-defined `render` to something else. In most other cases, you'll get a deprecation warning trying to invoke WebGL-related classes or methods, e.g., `new PIXI.WebGLRenderer()`.

### Graphics Holes

Drawing holes in Graphics was very limited in v4. This only supported non-Shape drawing, like using `lineTo`, `bezierCurveTo`, etc. In v5, we improved the hole API by supporting shapes. Unfortunately, there's no deprecation strategy to support the v4 API. For instance, in v4:

```js
const graphic = new PIXI.Graphics()
  .beginFill(0xff0000)
  .moveTo(0, 0)
  .lineTo(100, 0)
  .lineTo(100, 100)
  .lineTo(0, 100)
  .moveTo(10, 10)
  .lineTo(90, 10)
  .lineTo(90, 90)
  .lineTo(10, 90)
  .addHole();
```
[Live example in v4.8.6](https://pixiplayground.com/#/edit/i8LAwMn4s1oFkx2_fPt3F)

In v5, Graphics has simplified and the API changed from `addHole` to `beginHole` and `endHole`. 

```js
const graphic = new PIXI.Graphics()
  .beginFill(0xff0000)
  .drawRect(0, 0, 100, 100)
  .beginHole()
  .drawCircle(50, 50, 30)
  .endHole();
```
[Live example in v5.0.0-rc.2](https://pixiplayground.com/#/edit/TOj~yOoDPXJ2IuULzdnq8)

### Filter Padding

In v4 filters had a default padding of `4` and in v5 this has been changed to a default of `0`. This can cause some filters to look broken when used. To fix this issue simply add some padding to the filters you create. 

```js
// Glow filter from https://github.com/pixijs/pixi-filters
const filter = new PIXI.filters.GlowFilter(); 
filter.padding = 4;
```

Some filters, like `BlurFilter`, automatically calculate the padding so changes may not be necessary.

## Publishing Changes

### Canvas Becomes Legacy

Since WebGL and WebGL2 are now first-class, we have removed the canvas-based fallback from the default **pixi.js** package. If you need `CanvasRenderer`, you should switch to use **pixi.js-legacy** instead.

```js
import * as PIXI from "pixi.js";
// Will NOT return CanvasRenderer because canvas-based
// functionality was removed from "pixi.js"
const renderer = PIXI.autoDetectRenderer(); // return PIXI.Renderer or throws error
```

Instead, use the legacy bundle to have access to the canvas rendering.

```js
import * as PIXI from "pixi.js-legacy";
const renderer = PIXI.autoDetectRenderer(); // returns PIXI.Renderer or PIXI.CanvasRenderer
```

### Bundling Changes

If you're using [Webpack](https://webpack.js.org/), [Rollup](https://rollupjs.org/), [Parcel](https://parceljs.org) or another bundler to add PixiJS into your project there are a few subtle changes when moving to v5. Namely, the global `PIXI` object is no longer created automatically. This was removed from bundling for two purpose: 1) to improve tree-shaking for bundlers, and 2) for security purpose by protecting `PIXI`.

This is no longer a valid way to import:

```js
import "pixi.js"; // INVALID! No more global.PIXI!
const renderer = PIXI.autoDetectRenderer();
```

Instead, you should import only what's needed:

```js
import * as PIXI from "pixi.js";
const renderer = PIXI.autoDetectRenderer();

// or even better:
import { autoDetectRenderer } from "pixi.js";
const renderer = autoDetectRenderer();
```

Lastly, some 3rd-party plugins maybe expecting `window.PIXI`, so you might have to explicitly expose the global like this, however this is _not recommended_.

```js
import * as PIXI from 'pixi.js';
window.PIXI = PIXI; // some bundler might prefer "global" instead of "window"
```