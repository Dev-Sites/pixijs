## Where can I get a build?

PixiJS automatically builds using http://travis-ci.org and uploads those builds to our S3 servers. The latest build for each branch (and tag) is kept there. You can download the latest build for any branch (or tag) with the following URLs:

Unminified:

```
https://pixijs.download/<branch-or-tag-name>/pixi.js
https://pixijs.download/<branch-or-tag-name>/pixi.js.map
```

Minified:

```
https://pixijs.download/<branch-or-tag-name>/pixi.min.js
https://pixijs.download/<branch-or-tag-name>/pixi.min.js.map
```

For example, to download the latest unstable `dev` build:

- https://pixijs.download/dev/pixi.js
- https://pixijs.download/dev/pixi.js.map

Or the latest `release` build:

- https://pixijs.download/release/pixi.min.js
- https://pixijs.download/release/pixi.min.js.map

## How do I run the PixiJS locally?

If you are using the canvas renderer things will run fine :) However due to the nature of webGL's security restrictions, the webGL renderer will not work locally without a little help. Here's some tips that will help you out: https://github.com/mrdoob/three.js/wiki/How-to-run-things-locally

## What browsers are supported?

Browsers supported by the CanvasRenderer:
- IE 9+,
- FF 10+,
- Chrome 11+,
- Safari 2.0+,
- Opera 12+

Note that IE 8 can work with the CanvasRenderer when using a [canvas polyfill][0] and modern Object method polyfills, but it is not officially supported by the PixiJS team.

Browsers supported by the WebGLRenderer:
- IE 11+,
- FF 15+,
- Chrome 11+,
- Safari 5.1+,
- Opera 19+

We also try very hard to include support for:
- Ejecta
- CocoonJS

[0]: https://code.google.com/p/explorercanvas/