This document is useful for developers who are attempting to **upgrading from v4 to v5**. This includes gotchas and important context for understanding why your v4 code made need some subtle changes. In general, we've try to be as backward-compatible in v5 with the use of deprecation warnings in the `console`. There are, however, sometimes when changes are too substantial and require some additional help.

## Making WebGL First-Class

PixiJS v5 has made WebGL the first-class renderer and made CanvasRenderer to be second-class. Functionally, there's not much that changed from v4, but there are a bunch of subtle internal naming changes which could trip-up some developers upgrading to v5. For instance:

* `WebGLRenderer` becomes `Renderer`
* `renderWebGL` becomes `render` (in DisplayObject, Sprite, Container, etc)
* `_renderWebGL` becomes `_render` (in DisplayObject, Container, etc)

If you created a plugin or project that previously used `render` on a Container (see [#5510](https://github.com/pixijs/pixi.js/issues/5510)), this will probably cause your project to not render correctly. Please consider renaming your user-defined `render` to something else. In most other cases, you'll get a deprecation warning trying to invoke WebGL-related classes or methods, e.g., `new PIXI.WebGLRenderer()`.