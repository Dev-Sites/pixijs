PixiJS v5 introduces application plugins, however most people prefere to make their own application class and gameloop.

All the code from this article can be reproduced in our [examples page](http://pixijs.io/examples/?v=next-interaction#/basics/basic.js)

### Default Application

For `Hello World` apps and simple demos we have special `Application` class mashup. That way we can omit all initialization problems and get to the point:

```js
// setup application
var app = new PIXI.Application(800, 600, { backgroundColor: 0x1099bb });
document.body.appendChild(app.view);

// setup sprites
var sprite = PIXI.Sprite.fromImage('required/assets/basics/bunny.png');
sprite.anchor.set(0.5);
sprite.position.set(app.screen.width/2, app.screen.height/2);
sprite.interactive = true;
sprite.buttonMode = true;

sprite.on('pointerdown', function() {
    sprite.scale.x *= 1.25;
    sprite.scale.y *= 1.25;
});

app.stage.addChild(sprite);

app.ticker.add(function(delta) {
    sprite.rotation += 0.1 * delta;
});
```

### Custom application

To have more control, you can manually setup all the plugins you need. In this case we setup `ticker` and `interaction` plugins.

```js
// setup renderer and ticker
var renderer = new PIXI.Renderer(800, 600, { backgroundColor: 0x1099bb });
document.body.appendChild(renderer.view);
var stage = new PIXI.Container();

// setup ticker
var ticker = new PIXI.Ticker();
ticker.add(() => {
    renderer.render(stage);
}, PIXI.UPDATE_PRIORITY.LOW);
ticker.start();

// setup interaction
var interaction = new PIXI.interaction.InteractionManager({ 
    root: stage, 
    ticker: ticker, 
    view: renderer.view });

// setup sprites
var sprite = PIXI.Sprite.fromImage('required/assets/basics/bunny.png');
sprite.anchor.set(0.5);
sprite.position.set(renderer.screen.width/2, renderer.screen.height/2);
sprite.interactive = true;
sprite.buttonMode = true;

sprite.on('pointerdown', function() {
    sprite.scale.x *= 1.25;
    sprite.scale.y *= 1.25;
});

stage.addChild(sprite);

ticker.add(function(delta) {
    sprite.rotation += 0.1 * delta;
});
```

### Custom GameLoop

Some apps require even more control. Lets call requestAnimationFrame directly.

```js
// setup renderer and ticker
var renderer = new PIXI.Renderer(800, 600, { backgroundColor: 0x1099bb });
document.body.appendChild(renderer.view);
var stage = new PIXI.Container();
// setup interaction
var interaction = new PIXI.interaction.InteractionManager({ root: stage, view: renderer.view });

// setup RAF
var oldTime = Date.now();

requestAnimationFrame(animate);
function animate() {
    var newTime = Date.now();
    var deltaTime = newTime - oldTime;
    oldTime = newTime;	
    if (deltaTime < 0) deltaTime = 0;
    if (deltaTime > 1000) deltaTime = 1000;
    var deltaFrame = deltaTime * 60 / 1000; //1.0 is for single frame

    // throttled interaction updates
    interaction.update(deltaFrame);
	
    // update your game there
    sprite.rotation += 0.1 * deltaFrame;
	
    renderer.render(stage);

    requestAnimationFrame(animate);
}

// setup sprites

var sprite = PIXI.Sprite.fromImage('required/assets/basics/bunny.png');
sprite.anchor.set(0.5);
sprite.position.set(renderer.screen.width/2, renderer.screen.height/2);
sprite.interactive = true;
sprite.buttonMode = true;

sprite.on('pointerdown', function() {
    sprite.scale.x *= 1.25;
    sprite.scale.y *= 1.25;
});

stage.addChild(sprite);
```