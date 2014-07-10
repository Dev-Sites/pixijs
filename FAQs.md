#### How do I run the pixi.js locally?

If you are using the canvas renderer things will run fine :) However due to the nature of webGL's security restrictions, the webGL renderer will not work locally without a little help. Here's some tips that will help you out: https://github.com/mrdoob/three.js/wiki/How-to-run-things-locally

#### What browsers are supported?

Browsers supported by the CanvasRenderer:
- IE 9+,
- FF 10+,
- Chrome 11+,
- Safari 5.1.4+,
- Opera 12+

Note that IE 8 can work with the CanvasRenderer when using a [canvas polyfill][0], but it is not officially supported by the Pixi.js team.

Browsers supported by the WebGLRenderer:
- IE 11+,
- FF 15+,
- Chrome 11+,
- Safari 5.1+,
- Opera 19+

We also try very hard to include support for:
- Ejecta,
- CocoonJS

[0]: https://code.google.com/p/explorercanvas/