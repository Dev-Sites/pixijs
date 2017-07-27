### I. Including PixiJS

Include the PixiJS library in your project by linking to the PixiJS CDN. [More info here.](https://github.com/pixijs/pixi.js/wiki/FAQs#where-can-i-get-a-build)

```html
<script src="https://pixijs.download/release/pixi.min.js"></script>
```

Create a script element in the `<body>` element that will contain the PixiJS code:

```html
<body>
  <script>
     // code here
  </script>
</body>
```

### II. Creating An Application

Define a PixiJS Application object which will handle the rendering and updating as well as create your root stage object. Optionally, you can specify `width` and `height` options (defaults are 800 x 600). List of options can be found [here](http://pixijs.download/release/docs/PIXI.Application.html).

```js
var app = new PIXI.Application({ width: 640, height: 360 });
```

Add the `<canvas>` element created by PixiJS to your document (called `app.view`).

```js
document.body.appendChild(app.view);
```

### III. Adding Display Objects

Next, we'll draw a simple circle which we will use to render to our canvas.

```js
var circle = new PIXI.Graphics();
circle.beginFill(0x5cafe2);
circle.drawCircle(0, 0, 80);
circle.x = 320;
circle.y = 180;
```

This element is added to the `app.stage` element, the root-level instance of `PIXI.Container` which contains all elements to be rendered.

```js
app.stage.addChild(circle);
```

### IV. Now You're Cookin'

And that's it! To see the JSFiddle version of this example, [click here](https://jsfiddle.net/khm3L7qf/).

![download](https://user-images.githubusercontent.com/864393/28676996-3b0b8fd2-72ba-11e7-92f7-9bb71bec2cb1.png)


## Complete Code

```html
<html>
  <head>
    <script src="https://pixijs.download/release/pixi.min.js"></script>
  </head>
  <body>
    <script>
      var app = new PIXI.Application(640, 360);
      document.body.appendChild(app.view);
      var circle = new PIXI.Graphics();
      circle.beginFill(0x5cafe2);
      circle.drawCircle(0, 0, 80);
      circle.x = 320;
      circle.y = 180;
      app.stage.addChild(circle);
    </script>
  </body>
</html>
```

## More Examples

PixiJS has a bunch of [examples](http://pixijs.github.io/examples) worth checking out, here are a few:

* [Basic](http://pixijs.github.io/examples/#/basics/basic.js) Simple example like the one above
* [Container](http://pixijs.github.io/examples/#/basics/container.js) Use Container elements to next objects
* [Click](http://pixijs.github.io/examples/#/basics/click.js) Use interactivity to handle mouse/touch events
* [Text](http://pixijs.github.io/examples/#/basics/text.js) Render and style text
* [Graphics](http://pixijs.github.io/examples/#/basics/graphics.js) Drawing different shapes using Graphics object

## Documentation

PixiJS API Documentation can be found here: http://pixijs.download/release/docs/index.html