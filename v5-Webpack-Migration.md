This document is useful for developers who are attempting to upgrading from v4 to v5 using webpack. 
However if you are looking for more general advice, you can find them [here](https://github.com/pixijs/pixi.js/wiki/v5-Migration-Guide).

## PIXI Core

### PIXI.Loader

In v5, the Resources Loader has been modified to allow multiple instances. The `PIXI.loader` v4 does not exist anymore. However, there is a premade instance exposed in case you need to replace it with something equivalent : 
`PIXI.Loader.shared`

v4: 
```js
	import * as PIXI from 'pixi.js';
	PIXI.loader.add(assets).load(...);
        PIXI.loader.resources;
```	
v5:
```js
	import * as PIXI from 'pixi.js';
	PIXI.Loader.shared.add(assets).load(...);
        PIXI.Loader.shared.resources;
```
Or :  
```js
	import { Loader } from 'pixi.js';
	Loader.shared.add(assets).load(...);
        Loader.shared.resources;
```	

You can find more information in the Loader PIXI documentation : [http://pixijs.download/dev/docs/PIXI.Loader.html](http://pixijs.download/dev/docs/PIXI.Loader.html)

### PIXI.Texture

Regarding Texture, the function `PIXI.Texture.fromImage` has been renamed to `PIXI.Texture.from`

v4 : 
```js
	import * as PIXI from 'pixi.js';
	PIXI.Texture.fromImage('./img/test.png');
```	
v5:
```js
	import { Texture } from 'pixi.js';
	Texture.from('./img/test.png')
```

## PIXI-Spine (and other 3rd-party plugins that need global PIXI)

First, you need to upgrade it to a 2.0.0+ version to be compatible with v5.

When [Webpack](https://webpack.js.org/) and 3rd-party plugins, like **pixi-spine**, you might have difficulties building the global `PIXI` object resulting in a runtime error `ReferenceError: PIXI is not defined`. Usually this can be resolved by using [Webpack shimming globals](https://webpack.js.org/guides/shimming/#shimming-globals).

For instance, here's your import code:
 
```js
import * as PIXI from 'pixi.js';
import 'pixi-spine'; // or other plugins that need global 'PIXI' to be defined first
```

Add a `plugins` section to your **webpack.config.js** to let know Webpack that the global `PIXI` variable make reference to `pixi.js` module. For instance:

```
const webpack = require('webpack');

module.exports = {
    entry: '...',
    output: {
        ...
    },
    plugins: [
     new webpack.ProvidePlugin({
       PIXI: 'pixi.js'
     })
   ]
}
```

## PIXI-Sound

First, you need to upgrade it to a 3.0.0+ version to be compatible with v5.
In v5, import of pixi-sound has been changed to get rid of the use of PIXI global variable.

v4: 
```js
	import 'pixi-sound.js';
	PIXI.sound.stopAll();
```
v5:
```js
	import Sound from 'pixi-sound';
	Sound.stopAll();
```

You might have difficulties using pixi-sound with the PIXI.Loader. You need to import pixi-sound with PIXI.Loader to make the PIXI.Loader construct correctly the sound object or you will face some `undefined` errors when calling `PIXI.Loader.shared.resources["bgMusic"].sound`.

v5 without import : 
```js
        import * as PIXI from 'pixi.js';
        // Loading resources
        ...
	const bgMusic = PIXI.Loader.shared.resources["bgMusic"].sound; // bgMusic is undefined
```
v5:
```js
	import { Loader } from 'pixi.js';
	import 'pixi-sound'; // Needed where you load resources
        // Loading resources
        ...
        const bgMusic = Loader.shared.resources["bgMusic"].sound;
```

You can find a usefull webpack example here : [https://github.com/bigtimebuddy/pixi-sound-webpack-example](https://github.com/bigtimebuddy/pixi-sound-webpack-example)

## PIXI-Particles

First, you need to upgrade it to a 4.0.0+ version to be compatible with v5.

The use of `PIXI.particles` namespace is not required anymore.

v4: 
```js
       import 'pixi-particles';	
       const emitter = new PIXI.particles.Emitter(this, images, json);
```
v5:
```js
	import { Emitter } from 'pixi-particles';
	const emitter = new Emitter(this, images, json);
```