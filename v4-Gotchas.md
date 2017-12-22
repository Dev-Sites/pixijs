## Overview 

There are a handful of use-cases in PixiJS which you might, some-day encounter. In most cases, these were created because they fall outside of the majority of use-cases, because a lack of developer time, or because of some other necessity related to performance.

## PIXI.Container

### There's no `anchor` on non-Sprite objects.

Anchor has a meaning only for Sprite-based objects. There are [hacks](https://github.com/pixijs/pixi.js/issues/3272#issuecomment-349359529) that can help achieve an anchor for other DisplayObjects.

### Change parent of object without changing coords.

DisplayObject absolute transform is the result of multiplication of all local transforms of its parents. If we change object parent, it will change absolute position. This code prevents saves the position:

```js
newParent.toLocal(new PIXI.Point(0,0), bunny, bunny.position);
newParent.addChild(bunny);
```

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

@ivanpopelyshev: Sometimes this workaround works in other corner cases: cacheAsBitmap, ParticleContainer.

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
* PIXI.mesh.Mesh  †
* PIXI.mesh.Plane †
* PIXI.mesh.Rope †
* PIXI.mesh.NineSlicePlane †
* PIXI.extras.TilingSprite ‡

> † Supports texture regions since 4.5.0

> ‡ Supports texture regions since 4.1.0

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

## PIXI.VideoBaseTexture

Formats for video, and whether the video can be transparent, depend on browser. Sometimes there are bugs. See the [thread](http://www.html5gamedevs.com/topic/34518-basic-demo-video-for-hls-source/).

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

## Blend Modes

### Limited WebGL Blend Mode Support

WebGL supports only `NORMAL`, `ADD`, `MULTIPLY` and `SCREEN` blend modes. Advanced blend modes like `OVERLAY` are only supported by canvas renderer (at least until WebGL updates up to OpenGL ES 4.0).

**Workaround:** Check out the PixiJS plugin [pixi-picture](https://github.com/pixijs/pixi-picture) to add advanced blend mode support for WebGL.

### Custom canvas composition mode

Numbers from 20 are free, you can use composition modes that absent in [CanvasRenderer](https://github.com/pixijs/pixi.js/blob/dev/src/core/renderers/canvas/utils/mapCanvasBlendModesToPixi.js)

```
renderer.blendModes[20] = 'destination-out';
sprite.blendMode = 20;
```

### No Container Blend Mode Support

You cannot specify blend mode for a normal Container. 

**Workaround:** Using a VoidFilter, PixiJS will render everything inside container into separate framebuffer and then render whole layer with a blending mode of your choice. See [example](http://pixijs.github.io/examples/#/layers/lighting.js).

```js
var myBlend = new PIXI.filters.VoidFilter();
myBlend.blendMode = PIXI.BLEND_MODES.ADD;
container.filters = [myBlend];
```

### No Custom WebGL Blend Modes

You cannot use custom WebGL blend modes in vanilla PixiJS.

**Workaround:** [Example](http://www.html5gamedevs.com/topic/31803-scratch-card-effect/?tab=comments#comment-182673) of 'clearing' the surface, but there is no guarantee that it works on mobile devices.

## Texture preparation

Sometimes there's a lag when your app shows a sprite of particular base texture first time.

Manually, you can upload a texture to webgl memory using this:

```js
renderer.textureManager.updateTexture(myTexture);
```

You may pass any texture of particular atlas to upload whole atlas.

### Garbage Collector

When you change the stage, most of the time you dont remember which resources you need for next stage and which ones can be destroyed. There's also huge problem with generated resources.

GC solves that easily, it removes all textures that weren't used in last two minutes from videomemory. But it can be a headache if you want to show particular animation every three minutes - first frame will have lag. In that case, if you are ready to take responsibility, you can switch it off

```js
renderer.textureGC.GC_MODE = PIXI.GC_MODES.MANUAL;
```

Or just switch it off globally for all renderers to be created later

```js
PIXI.settings.GC_MODE = PIXI.GC_MODES.MANUAL;
```

### prepare

Use [prepare](http://pixijs.download/dev/docs/PIXI.prepare.html) API to make sure that all textures of particular stage are uploaded to videomemory. Beware, the method is asynchronous!

### createImageBitmap

Videomemory upload is not the only operation that causes lag. Browser's PNG->RGBA conversion also does it, its synchronous pain. After some time, browser removes RGBA buffer from regular memory, that helps in low-memory devices. We have no control over it, except new function [createImageBitmap](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/createImageBitmap) that solves this issue.

createImageBitmap method will be used in pixi-v5. 

Fortunately, for v4, there is the [snippet](https://gist.github.com/anonymous/d01963a44c523dc04b21bfd7037081b6) from [forum thread](http://www.html5gamedevs.com/topic/31359-upload-images-to-gpu-in-a-web-worker/) that enables createImageBitmap in case you load resources with `fromImage`.

## Interaction

[InteractionManager](https://github.com/pixijs/pixi.js/blob/dev/src/interaction/InteractionManager.js) instance is created for every renderer instance automagically. See `renderer.plugins.interaction`

### CSS transforms on canvas

If you do any transformations on canvas, there's no guarantee that InteractionManager calculates correct coordinates, see https://jsfiddle.net/r3fktwkk/ - pixi thinks that sprite is in top-left corner.

Please override [mapPositionToPoint](https://github.com/pixijs/pixi.js/blob/dev/src/interaction/InteractionManager.js#L954)

Do it either in a class

```js
PIXI.interaction.InteractionManager.prototype.mapPositionToPoint= function(point, x, y) { ... }
```

either in a particular instance

```js
renderer.plugins.interaction.mapPositionToPoint= function(point, x, y) { ... }
```
