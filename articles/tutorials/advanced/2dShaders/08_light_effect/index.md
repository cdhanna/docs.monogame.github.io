In the previous chapter, you learned how to use custom vertex shaders with MonoGame `SpriteBatch`. In this chapter, we will focus on adding lighting to the _Dungeon Slime_ game. Our goal is to add more visual depth to the game.

## Deferred Rendering

So far, the game's rendering has been fairly straightforward. The game consists of a bunch of sprites, and all those sprites are drawn straight to the screen using a custom shader effect. Adding lights is going to complicate the rendering, because now each sprite must consider _N_ number of lights before being drawn to the screen. 

There are two broad categories of strategies for rendering lights in a game, 
1. _Forward_ rendering, and
2. _Deferred_ rendering. 

In the earlier days of computer graphics, forward renderers were ubiquitous. Imagine a simple 2d where there is a single sprite with 3 lights nearby. The sprite would be rendered 3 times, once for each light. Each individual pass would layer any existing passes with the next light. This technique is forward rendering, and there are many optimizations that make it fast and efficient. However, in a scene with lots of objects and lots of lights, each object needs to be rendered for each light, and the amount of rendering can scale poorly. The amount of work the renderer needs to do is roughly proportional to the number of sprites (`S`) multiped by the number of lights (`L`), or `S * L`. 

In the 2000's, the deferred rendering strategy was [introduced](https://sites.google.com/site/richgel99/the-early-history-of-deferred-shading-and-lighting) and popularized by games like [S.T.A.L.K.E.R](https://developer.nvidia.com/gpugems/gpugems2/part-ii-shading-lighting-and-shadows/chapter-9-deferred-shading-stalker). In deferred rendering, each object is drawn _once_ without _any_ lights to an off-screen texture. Then, each light is drawn on top of the off-screen texture. To make that possible, the initial rendering pass draws extra data about the scene into additional off-screen textures. Theoretically, a deferred renderer can handle more lights and objects because the work is roughly approximate to the sprites (`S`) _added_ to the lights (`L`), or `S + L`. 

Deferred rendering was popular for several years. MonoGame is an adaptation of XNA, which came out in the era of deferred rendering. However, deferred renderers are not a silver bullet for performance and graphics programming. The crux of a deferred renderer is to bake data into off-screen textures, and as monitor resolutions have gotten larger and larger, the 4k resolutions are starting to add too much overhead. Also, deferred renderers cannot handle transparent materials. Many big game projects use deferred rendering for _most_ of the scene, and a forward renderer for the final transparent components of the scene. As with all things, which type of rendering to use is a nuanced decision. There are new types of forward rendering strategies (see, [clustered rendering](https://github.com/DaveH355/clustered-shading)) that can out perform deferred renderers. However, for our use cases, the deferred rendering technique is sufficient. 

## Modifying the Game

Writing a simple deferred renderer can be worked out in a few steps, 
1. take the scene as we are drawing it currently, and store it in an off-screen texture. This texture is often called the diffuse texture, or color texture.
2. render the scene again, but instead of drawing the sprites normally, draw their _Normal_ maps to an off-screen texture, called the normal texture.
3. create a new off-screen texture, called the light texture, where each light is layered on-top of each other,
4. finally, create a rendering to the screen based on the lighting texture and the color texture.

The second stage references a new term, called the _Normal_ Map. We will come back to this later in the chapter. For now, we will focus on the other steps. 

### Drawing to an off-screen texture

To get started, we need to draw the main game sprites to an off-screen texture instead of directly to the screen. MonoGame has a `RenderTarget` class