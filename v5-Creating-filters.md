## Overview

V5 filters are awesome.

### Coordinate systems

There are 3 unit measures of coordinates:
1. _Normalized_ (0,0) is top-left corner, (1,1) is bottom-right.
2. _Virtual units_ (CSS), its units that pixi use for containers, displayObjects and so on. (0,0) is left-top corner, (w,h) is right-bottom.
3. _Physical pixels_ (pixels), basically same a screen units. Difference appears on retina screens. As an example, it helps in case you want to know physical pixel on the right/left/top/down of current one.

Here and in all docs we use `normalized` word for first type, `pixels` for third type. By default we are talking about virtual units (CSS).

There are 5 coordinates systems in pixi filters. Each can be used with any of three types.

1. _Input coords_ - temporary pow2 texture that FilterSystem took from the pool. Used for `texture2D` sampling.
2. _Screen coords_ - do not depend whether output is temporary texture or screen, whether there we are 10 filters away from screen, it is the screen coord.
3. _Filter coords_ - (0,0) is mapped to top-left corner of part of screen that is covered by filter. For CSS or physical units it has the same scale but different offset than screen.
4. _Sprite texture coords_ - sometimes there's extra sprite input in filter. Sprite is positioned in pixi stage tree and its area does not equal filter area. This is what we pass in `texture2D()` for extra sampler. Example is [DisplacementFilter](https://github.com/pixijs/pixi.js/tree/dev/packages/filters/filter-displacement/src)
5. _Sprite atlas coords_ - sprite can use texture from an atlas, and just _sprite texture coords_ aren't enough to get the correct value from `texture2D`. Example: [SpriteMaskFilter](https://github.com/pixijs/pixi.js/tree/dev/packages/core/src/filters/spriteMask)

## Default filter code

Filter constructor params include vertex and fragment shaders. What happens if we specify `null` or `undefined` in them? 

```js
new Filter(undefined, fragShader, myUniforms); // default vertex shader
new Filter(vertShader, undefined, myUniforms); // default fragment shader
new Filter(undefined, undefined, myUniforms);  // both default
```

### Default vertex shader:

```glsl
attribute vec2 aVertexPosition;

uniform mat3 projectionMatrix;

varying vec2 vTextureCoord;

uniform vec4 inputSize;
uniform vec4 outputFrame;

vec4 filterVertexPosition( void )
{
    vec2 position = aVertexPosition * max(outputFrame.zw, vec2(0.)) + outputFrame.xy;

    return vec4((projectionMatrix * vec3(position, 1.0)).xy, 0.0, 1.0);
}

vec2 filterTextureCoord( void )
{
    return aVertexPosition * (outputFrame.zw * inputSize.zw);
}

void main(void)
{
    gl_Position = filterVertexPosition();
    vTextureCoord = filterTextureCoord();
}
```

* `aVertexPosition` is _normalized filter coord_

* `vTextureCoord` is _normalized input coord_

* `aVertexPosition * outputFrame.zw` is _filter coord_

*  `vec2 position = aVertexPosition * max(outputFrame.zw, vec2(0.)) + outputFrame.xy` is screen coord

@ivanpopelyshev : 

1. I do not know why is there `max` 
2. I recommend to put it in `vFilterCoord` varying

### Default fragment code

```glsl
varying vec2 vTextureCoord;

uniform sampler2D uSampler;

void main(void){
   gl_FragColor = texture2D(uSampler, vTextureCoord);
}
```

As mentioned above, `vTextureCoord` is _normalized input coord_ that we pass to default sampler.

## Default shaders in v4

### v4 vertex shader

Default shaders in PixiJS v4 differ from the ones in v5. If you are porting filter from v4 please use the following code for vertex shader:

```glsl
attribute vec2 aVertexPosition;
attribute vec2 aTextureCoord;

uniform mat3 projectionMatrix;

varying vec2 vTextureCoord;

void main(void) {
    gl_Position = vec4((projectionMatrix * vec3(aVertexPosition, 1.0)).xy, 0.0, 1.0);
    vTextureCoord = aTextureCoord;
}
```

`aTextureCoord` is normalized input coord that is passed through. If PixiJS v5 detects that attribute, it switches to v4-compatibility mode that adds old filter uniforms.


### v4 default fragment shader

In case you are porting stuff from v4 and you used default fragment shader, you may notice that your filter look different in v5. That's because default v4 fragment code had a serious problem regarding double multiplication by alpha.

### v4 `dimensions` uniform

There is very popular workaround with `dimensions` uniform in v4, it looks like that:

```js
apply (filterManager, input, output, clear)
{
    this.uniforms.dimensions[0] = input.sourceFrame.width;
    this.uniforms.dimensions[1] = input.sourceFrame.height;
    filterManager.applyFilter(this, input, output, clear);
}
```

Unfortunately, in v5 it crashes because `input` is not RenderTarget but RenderTexture, it has no `sourceFrame` field. To fix the crash you have to:

1. replace `input.sourceFrame` to `input.filterFrame`.
2. add `dimensions` uniform in filter constructor params or body: `this.uniforms.dimensions = new Float32Array(2);`

However, I advice you to remove it completely in favor of built-in `inputSize` uniform, `inputSize.xy` is the size of filter area in pixels, exactly the same as `dimensions`. In that case you can remove extra code from constructor and `apply` function.

## Fullscreen filters

This line forces pixi to use temporary renderTexture of the same size as screen:

```js
filter.filterArea = renderer.screen; //same as app.screen
```

Also known as `pixi-v3 emulation` mode.

Input, output and screen coords are the same in that case, and you don't have to use conversion functions.

## Conversion functions
