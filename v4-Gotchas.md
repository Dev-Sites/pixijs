## Overview 

There are a handful of use-cases in PixiJS which you might, some-day encounter. In most cases, these were created because they fall outside of the majority of use-cases, because a lack of developer time, or because of some other necessity related to performance.

## PIXI.Container

### Filters on root PIXI.Container don't render

For example, in the snippet below, the blur does not render:

```js
const app = new PIXI.Application();
app.stage.filters = [new PIXI.filters.BlurFilter()]; // <-- does not show up
```

**Workaround:** Nest a container within the stage:

```js
const app = new PIXI.Application();
const root = new PIXI.Container();
app.stage.addChild(root);
root.filters = [new PIXI.filters.BlurFilter()];
```

## PIXI.Graphics

### The `addHole` API doesn't work with shapes

The `addHole` API is designed to draw holes, or "windows", within Graphics objects. Yet, it only works with primitive drawing commands and does not work with other convenience shapes (e.g., `drawCircle`, `drawRect`). The hole and the main Graphic cannot be shapes.

```js
const graphic = new PIXI.Graphics()
  .beginFill(0xFF0000)
  .drawRect(0, 0, 110, 110)
  .drawRect(50, 50, 10, 10)
  .addHole(); // <-- hole does not show-up
```

**Workaround:** Use `moveTo`, `lineTo`, `arcTo`, `bezierCurveTo` and `quadraticCurveTo` to render the hole _and_ the main polygon.

```js
const graphic = new PIXI.Graphics()
  .beginFill(0xFF0000)
  .moveTo(0, 0)
  .lineTo(110, 0)
  .lineTo(110, 110)
  .lineTo(0, 110)
  .moveTo(50, 50)
  .lineTo(60, 50)
  .lineTo(60, 60)
  .lineTo(50, 60)
  .addHole();
```
