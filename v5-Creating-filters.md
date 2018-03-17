V5 filters are awesome.

# Fullscreen filters

This line forces pixi to use temporary renderTexture of the same size as screen:

```js
filter.filterArea = renderer.screen; //same as app.screen
```

In PixiJS version 3 that was the only mode available.
PixiJS version 4 calculates the size of filtered objects and runs shader only on part of texture, that adds some problems.

Lets talk about uniforms that help to map the filter are to the texture.
