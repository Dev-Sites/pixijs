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

## PIXI.Texture

### Things That Don't Support Spritesheet Textures

The following objects do not support Texture objects that are part of a Spritesheet shared with other Texture objects or where the Texture's frame is not the entire size of the BaseTexture. Using a Spritesheet with one of these classes will cause the transforms to render incorrectly. 

* Sprite-based masks
* Filters with sprite inputs (e.g., DisplacementFilter)

Mesh objects support texture regions since 4.5.0

* `PIXI.mesh.Mesh`
* `PIXI.mesh.Plane`
* `PIXI.mesh.Rope`
* `PIXI.mesh.NineSlicePlane`

TilingSprite supports texture regions since 4.1.0

* `PIXI.extras.TilingSprite`

```js
PIXI.loader.add('atlas', 'atlas.json')
  .load((loader, resources) => {
    const sprite = new PIXI.Sprite(
      resources.atlas.textures.myTexture
    );
    someContainer.addChild(sprite);
    someContainer.mask = sprite; // <-- won't work
  });
```

**Workaround:** If you intend to use any of these advanced DisplayObjects, make sure they are loaded as standalone images and are not part of a bundled Spritesheet.

```js
PIXI.loader.add('atlas', 'atlas.json')
  .add('myMask', 'myMask.png')
  .load((loader, resources) => {
    const mesh = new PIXI.Sprite(
      resources.myMask.texture
    );
    someContainer.addChild(sprite);
    someContainer.mask = sprite;
  });
```

## PIXI.Filter

### Filters Don't Render on CanvasRenderer

Filters are very expensive when implemented on a `CanvasRenderer` and PixiJS does not support the use-case here. Performance would simply be too poor. Therefore, Filters only work with WebGLRenderer and will be ignored on platforms that do not support WebGL (e.g., Internet Explorer, Android 4, etc).

```js
const stage = new PIXI.Container();
const renderer = new PIXI.CanvasRenderer();
stage.filters = [new PIXI.filters.BlurFilter()]; // <-- won't render
renderer.render(stage);
```

**Tip:** While there's no workaround, you should make sure that you test your PixiJS content on non-WebGL platforms if you intend to use filters.

## BlendModes

WebGL supports only NORMAL, ADD, MULTIPLY and SCREEN blendmodes. Advanced blendmodes are supported by canvas renderer, but there is a workaround through (pixi-picture)[https://github.com/pixijs/pixi-picture]

You cannot use blendmode on a simple container, but there's workaround through filters, that way PIXI will render everything inside container into separate framebuffer and then render whole layer with a blendmode of your choice (example)[http://pixijs.github.io/examples/#/layers/lighting.js]

```js
var myBlend = new PIXI.filters.VoidFilter();
myBlend.blendMode = PIXI.BLEND_MODES.ADD;
container.filters = [myBlend];
```