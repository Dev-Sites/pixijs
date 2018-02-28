V4 filters differ from V3. You can't just add in the shader and assume that texture coordinates are in the [0,1] range.

# Filter Area

Thanks to @bQvle and @radixzz

First, lets work with the AREA. When you apply a filter to a container, PIXI calculates the bounding box for it. We are working with that bounding box.

Note: `vTextureCoord` multiplied by `filterArea.xy` is the real size of bounding box.

Don't try to think about it: it's like that because of performance reasons, it's not logical in a user-experience sense. Neither `vTextureCoord` dimensions or `filterArea.xy` are predictable, but their multiplication is what we need. 

Area can have padding, so please don't use it to get "displacement texture" coordinates or any second-hand textures you are adding to the shader, use "mappedMatrix" for it (see below)

If you want to get the pixel coordinates, use `uniform filterArea`, it will be passed to the filter automatically.

```glsl
uniform vec4 filterArea;
...
vec2 pixelCoord = vTextureCoord * filterArea.xy;
```

They are in pixels. That won't work if we want something like "fill the ellipse into a bounding box". So, lets pass dimensions too! PIXI doesnt do it automatically, we need a manual fix:

```js
filter.apply = function(filterManager, input, output, clear)
{
  this.uniforms.dimensions[0] = input.sourceFrame.width
  this.uniforms.dimensions[1] = input.sourceFrame.height

  // draw the filter...
  filterManager.applyFilter(this, input, output, clear);
}
```

Lets combine it in shader!

```glsl
uniform vec4 filterArea;
uniform vec2 dimensions;
...
vec2 pixelCoord = vTextureCoord * filterArea.xy;
vec2 normalizedCoord = pixelCoord / dimensions;
```

Here's the fiddle: https://jsfiddle.net/parsab1h/ . You can see that the shader uses "map" and "unmap" to get to that pixel.

Now let's assume that you somehow need real coordinates on screen for your filter. You can use another component of `filterArea`, `zw`:

```glsl
uniform vec4 filterArea;
...
vec2 screenCoord = (vTextureCoord * filterArea.xy + filterArea.zw);
```

# Fitting problem

Thanks to @adam13531 at github.

One small problem: these values can become wrong when PIXI tries to fit the filter bounding box to the renderer. Here's an example: http://jsfiddle.net/xbmhh207/1/

Please use this line to fix it:

```js
filter.autoFit = false;
```

# Bleeding problem

Thanks to @bQvle

The temporary textures that are used by FilterManager can have some bad pixels. It can bleed. For example, displacementSprite can look through the edge, try to move mouse at the bottom or left edge of http://pixijs.github.io/examples/#/filters/displacement-map.js. You see that transparent (black) zone, but it could be ANYTHING if it wasn't clamped. To make sure it doesn't happen in your case, please clamp your coordinates after you map them:

```glsl
uniform vec4 filterClamp;

vec2 pixelCoord = WE_CALCULATED_IT_SOMEHOW_SEE_ABOVE
vec2 unmappedCoord = pixelCoord / filterArea.xy;
vec2 clampedCoord = clamp(unmappedCoord, filterClamp.xy, filterClamp.zw);
vec4 rgba = texture2D(uSampler, clampedCoord);
```

Both FilterClamp and FilterArea are provided by FilterManager, you don't have to pass it in "filter.apply". For reference:

```js
//From FilterManager.syncUniforms
if (shader.uniforms.filterArea)
        {
            currentState = this.filterData.stack[this.filterData.index];

            const filterArea = shader.uniforms.filterArea;

            filterArea[0] = currentState.renderTarget.size.width;
            filterArea[1] = currentState.renderTarget.size.height;
            filterArea[2] = currentState.sourceFrame.x;
            filterArea[3] = currentState.sourceFrame.y;

            shader.uniforms.filterArea = filterArea;
        }

        // use this to clamp displaced texture coords so they belong to filterArea
        // see displacementFilter fragment shader for an example
        if (shader.uniforms.filterClamp)
        {
            currentState = currentState || this.filterData.stack[this.filterData.index];

            const filterClamp = shader.uniforms.filterClamp;

            filterClamp[0] = 0;
            filterClamp[1] = 0;
            filterClamp[2] = (currentState.sourceFrame.width - 1) / currentState.renderTarget.size.width;
            filterClamp[3] = (currentState.sourceFrame.height - 1) / currentState.renderTarget.size.height;

            shader.uniforms.filterClamp = filterClamp;
}
```

OK, now we have a "transparent" zone instead of random pixels. But what if we want it to be fit completely?

```js
displacementFilter.filterArea = app.screen; // not necessary, but I prefer to do it.
displacementFilter.padding = 0;
```

That'll do it. Why did I modify `filterArea` there, PIXI will "fit" it anyway, right? I don't want PIXI to spend time calculating the bounds of the container.

No extra transparent space, and if you put it into http://pixijs.github.io/examples/#/filters/displacement-map.js , and you move mouse to bottom edge, you'll see the grass.

# Extra texture, mapped matrix

When you want to use an extra texture to put in the filter, you need to position it as a sprite somewhere. We are working with a sprite that is not renderable but exists in the stage. Its transformation matrix will be used to position your texture in the filter. Please use https://github.com/pixijs/pixi.js/blob/dev/src/filters/displacement/DisplacementFilter.js and http://pixijs.github.io/examples/#/filters/displacement-map.js as an example.

Look for a mapped matrix: 

```js
this.uniforms.filterMatrix = filterManager.calculateSpriteMatrix(this.maskMatrix, this.maskSprite);
```

`maskMatrix` is a temporary transformation that you have to create for the filter, you don't need to fill it. Sprite has to be added into the stage tree and positioned properly.

You only can use textures that are not trimmed or cropped, as that would throw off the transformation. If you want the texture to be repeated, like a fog, make sure it has power-of-two dimensions, and specify it in `baseTexture` before it's uploaded on GPU/rendered for first time!

```js
texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;
```
If you want to use an atlas texture as a secondary input for a filter, please wait for pixi-v5 or do it yourself. Add clamping uniforms, use them in the shader and make a better mapping in `filterMatrix`.

# Renderer plugin

For some cases its easier to perform operations that are easily handled through renderer plugin.

If you want to use extra textures for a filter, please look here:

Full example: https://github.com/pixijs/pixi-plugin-example

Shortcut: https://github.com/TazOen/createShaderPlugin , http://www.html5gamedevs.com/topic/31704-createshaderplugin-helper-function/

# Cannot read property 'location' of undefined

That appears when some of properties that are required by FilterManager are not used in the shader. Even if you have them in your code, glsl compiler may remove unused attributes, its platform-dependant.

Please dont do that. If you really dont need textureCoord - consider switch to renderer plugin.

```js
const filterCode = `void main(){
   gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0);
}`;
const filter = new PIXI.Filter(null, filterCode);
someSprite.filters = [filter];
```