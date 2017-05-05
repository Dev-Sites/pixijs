## Global
- Only optimize when you need to! PIXI can handle a fair amount of content off the bat.
- Be mindful of the complexity of your scene. The more objects you add the slower things will end up.
- Order can help, for example sprite / graphic / sprite / graphic is slower than sprite / sprite / graphic / graphic
- Some older mobile devices run things a little slower. passing in the option 'legacy:true' to the renderer can help with performance
- Culling, PIXI does not cull anything, we have left this to you and you application. 

## Sprites
- Use spritesheets where possible to minimize total textures
- Sprites can be batched with up to 16 different textures (dependent on hardware)
- This is the fastest way to render content
- On older devices use smaller low res textures
- Add the extension `@0.5x.png` to the 50% scale-down so PIXI will double them automatically
- Draw order can be important

## Graphics
- Graphics fastest when they are not modified constantly (not including the transform, alpha or tint!)
- Separate Graphic instances are not batched
- Using 100s of graphics objects can be slow, in this instance use sprites (you can create a texture)

## Textures
- Textures are automatically managed by a Texture Garbage Collector
- You can also manage them yourself by using `texture.destroy()`
- If you plan to destroyed more than one at once add a random delay to their destruction to remove freezing.
- delayed texture destroy if you plan to delete a lot of textures yourself.

## Text
- Avoid changing it on every frame as this can be expensive (each time it draws to a canvas and then uploads to GPU)
- Bitmap Text gives much better performance for dynamically changing text. 
- Text resolution matches the renderer resolution, to increase resolution yourself you can textDouble the font size & scale it down to 50%.

## Masks
- Masks can be expensive if too many are used. 100s of masks will really slow things down
- Axis aligned Rectangle Masks are the fastest (as the use scissor rect)
- Graphics masks are second fastest (as they use the stencil buffer)
- Sprite masks are the third fastest (they uses filters) - really expensive do not use too many in your scene!

## Filters
- Release memory : `displayObject.filters = null`
- If you know the size of them: `displayObject.filterArea = new PIXI.Rectangle(x,y,w,h)`. This can speeds things up as it means the object does not need to be measured. 
- Filters are expensive. Using too many will start to slow things down!

## CacheAsBitmap
- Setting `cacheAsBitmap` to `true` turns an object into a sprite by caching it as a texture.
- It has a one time cost when it is activated as it draws the object to a texture.
- Avoid changing this on elements frequently.
- If you have a complicated item that has lots of sprites / filters AND do not move then this will speed up rendering!
- Do not need apply to sprites as they are already textures
- Do not use if the object where its children are constantly changing as this will slow things down

## Interaction
- If an object has no interactive children use `interactiveChildren = false` the interaction manager will then be able to avoid crawling through the object.
- Set `hitArea = new PIXI.Rectangle(x,y,w,h)`. As above should stop the interaction manager from crawling through the object.