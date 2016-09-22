For those upgrading from v3 to v4 of PIXI.

### Texture

Textures are [clamped by default](http://www.html5gamedevs.com/topic/24778-displacementfilter-not-repeating-when-moving-displacement-sprite/), in v3 they were repeated. 

```js
texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;
```

### Filters

Many of the filters have been externalized to [**pixi-filters**](https://github.com/pixijs/pixi-filters) with the exception of ColorMatrixFilter, BlurFilter, DisplacementFilter, FXAAFilter, NoiseFilter. All other filters, use **pixi-filters**.

You have to use `mappedMatrix` to convert `textureCoord` to `mapCoord`, all because of paddings and pow2. 

Use `filterClamp` uniform to prevent bad things on edges (`inputCoord.xy`, `filterClamp.xy`, `filterClamp.zw`) , displacement and blur are good examples for that. See [forum post](http://www.html5gamedevs.com/topic/24347-weird-filterareas-size-on-v4/) for more information.

### DisplayObject

Setting the position of a DisplayObject no longer maintain the reference to the Point. The values of the point are assigned to the transform of the DisplayObject. For instance:

```js
s.position = myPoint; // COPYING ASSIGNMENT! in v3 that was reference
s2.position = myPoint; //shadow sprite , same coord
myPoint.x = 10; // won't work

s.anchor = myPoint ; // DON'T do that please! we will fix it later
``` 

### Text

`font` field is deprecated on the Text's style. Use `fontFamily`, `fontWeight`, `fontSize` etc. See corresponding [pull-request](https://github.com/pixijs/pixi.js/issues/2859). 

### TilingSprite

TilingSprite works bad with mipmap, and pow2 textures are mipmapped by default. We will fix that issue later, for now just

```js
texture.baseTexture.mipmap = false;
```