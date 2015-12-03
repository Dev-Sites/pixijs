### Destroying Objects

All PIXI objects have a `.destroy()` method that cleans up any references it holds (to allow for garbage collection), as well as potentially cleaning up GPU memory. Once you are finished with an object and want to get rid of it, you should call the `.destroy()` method.

When destroying textures, make sure to consider if you need to also destroy the BaseTexture that it uses to represent the source image. If so, you need to pass `true` as the first parameter to ensure that WebGL and CPU memory for the base image is also cleaned up.

References: [`Texture.prototype.destroy`][Texture.destroy], [`BaseTexture.prototype.destroy`][BaseTexture.destroy]

### Masking 

There are two forms of masking supported: "vector" masking which uses a `Graphics` object to draw a masking shape, and "alpha" masking which uses a `Sprite` that represents the masking area. When using a Sprite it may be useful to sometimes draw the shape and colors that are needed at runtime. You can do this in a few ways:

1. Draw the shape with a Graphics object, call `graphics.generateTexture()` and then use that texture to create a new `Sprite`.
2. Draw the desired mask to a `<canvas/>` element, then create a texture from that canvas with `Texture.fromCanvas(canvas);`

References: [`Texture.fromCanvas`][`Texture.fromCanvas`]





[Texture.destroy]: http://pixijs.github.io/docs/PIXI.Texture.html#destroy
[BaseTexture.destroy]: http://pixijs.github.io/docs/PIXI.BaseTexture.html#destroy
[Texture.fromCanvas]: http://pixijs.github.io/docs/PIXI.Texture.html#.fromCanvas