# Effects

Currently, WebGAL's effect system is powered by PixiJS.

## Using Effects

### Initialize Pixi

Initialize Pixi using `pixiInit`.

``` ws
pixiInit;
```

::: warning
If you want to use effects, you must run this command to initialize Pixi first.

If you want to clear the effects that have already taken effect, you can use this syntax to clear the effects.
:::

### Add Effects

Add effects using `pixiPerform`.

``` ws
pixiPerform:rain; // Add a rain effect
```

Note: After the effect takes effect, if it is not re initialized, the effect will continue to run.

### Default Template Effects

| Effect | Command |
| :--- | :--- |
| Rain | pixiPerform:rain; |
| Snow | pixiPerform:snow; |
| Heavy Snow | pixiPerform:heavySnow; |
| Cherry Blossoms | pixiPerform:cherryBlossoms; |

These four effects are runtime scripts in the default game's `game/pixi-performs/` directory. Edit the matching `.js` file to change values such as speed, particle count, scale, and angle, then refresh the preview. Rebuilding the engine is not required.

### Superimpose Effects

If you want to superimpose two or more effects, you can superimpose different effects without using the `pixiInit` command.

``` ws
pixiPerform:rain;
pixiPerform:snow;
```

### Clear Superimposed Effects

Initialize using `pixiInit` to clear all effects that have been applied.

## Adding Custom Effects Without Rebuilding

In a WebGAL version that supports runtime effects, add a JavaScript file directly to the game directory. You do not need to modify the engine source, maintain an `index.js`, or run `yarn build`.

The effect name maps directly to its file:

``` text
pixiPerform:myPerform;
    -> game/pixi-performs/myPerform.js

pixiPerform:weather/rain;
    -> game/pixi-performs/weather/rain.js
```

For example, put this in `game/pixi-performs/myPerform.js`:

``` ts
(() => {
  let instanceId = 0;

  window.WebGALPixiPerform.register('myPerform', {
    fg: () => {
      const PIXI = window.PIXI;
      const stage = window.PIXIapp;
      const container = new PIXI.Container();
      const sprite = PIXI.Sprite.from('./game/tex/my-effect.png');

      sprite.anchor.set(0.5);
      sprite.position.set(stage.stageWidth / 2, stage.stageHeight / 2);
      container.addChild(sprite);
      stage.foregroundEffectsContainer.addChild(container);

      const tickerKey = `runtime-my-perform-${++instanceId}`;
      stage.registerAnimation(
        {
          setStartState: () => {},
          setEndState: () => {},
          tickerFunc: (delta) => {
            sprite.rotation += 0.01 * delta;
          },
        },
        tickerKey,
      );
      stage.requestRender();

      return { container, tickerKey };
    },
  });
})();
```

Textures can be placed in `game/tex`. `fg` uses the foreground layer. For a background effect, use `bg` and add the container to `stage.backgroundEffectsContainer`.

Every call must synchronously return `{ container, tickerKey }`, and each animated instance should use a unique `tickerKey`. WebGAL uses these values to destroy the container and remove the animation when the effect is cleared.

Initialize Pixi before calling the effect whose name matches the file:

``` ws
pixiInit;
pixiPerform:myPerform;
```

After editing the effect file, refresh the preview page. Rebuilding the engine is not required. Effect files are trusted JavaScript executed in the game page; do not install effects from an untrusted source.
