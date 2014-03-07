#### How do I run the pixi.js locally?

If you are using the canvas renderer things will run fine :) However due to the nature of webGL's security restrictions, the webGL renderer will not work locally without a little help. Here's some tips that will help you out: https://github.com/mrdoob/three.js/wiki/How-to-run-things-locally

#### What browsers are supported?

We support the following with Canvas:
IE 9+, FF 10+, Chrome 11+, Safari 5.1+, Opera 12+

We support the following with WebGL:
IE 11+, FF 15+, Chrome 11+, Safari 5.1+, Opera 19+

We also try very hard to include support for:
Ejecta, and CocoonJS

If it doesn't work in an environment you are using please let us know and we will try to include it; provided that it supports *at least* these features:

- Object.defineProperty
- Canvas API
- JS Image objects