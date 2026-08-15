# 特效

目前，WebGAL 的特效系统由 PixiJS 实现。

## 使用特效

### 初始化 Pixi

使用 `pixiInit` 初始化 Pixi。

``` ws
pixiInit;
```

::: warning
如果你要使用特效，那么你必须先运行这个命令来初始化 Pixi。

如果你想要消除已经作用的效果，你可以使用这个语法来清空效果。
:::

### 添加特效

使用 `pixiPerform` 添加特效。

``` ws
pixiPerform:rain; // 添加一个下雨的特效
```

注意：特效作用后，如果没有重新初始化，特效会一直运行。

### 默认模板特效列表

| 效果 | 指令                        |
| :--- | :-------------------------- |
| 雨 | pixiPerform:rain;             |
| 雪 | pixiPerform:snow;             |
| 大雪 | pixiPerform:heavySnow;      |
| 樱花 | pixiPerform:cherryBlossoms; |

这四个特效是默认游戏目录中的运行时脚本，位于 `game/pixi-performs/`。可以直接打开对应 `.js` 文件修改速度、数量、缩放和角度等配置，刷新预览即可生效，不需要重新编译引擎。

### 叠加特效

如果你想要叠加两种及以上效果，你可以在不使用 `pixiInit` 指令的情况下叠加不同的效果。

``` ws
pixiPerform:rain;
pixiPerform:snow;
```

### 清除已叠加的特效

使用 `pixiInit` 来初始化，这样可以消除所有已经应用的效果。

## 添加无需重新编译的自定义特效

在支持运行时特效的 WebGAL 版本中，可以直接在游戏目录下新建 JavaScript 文件，无需修改引擎源码、维护 `index.js` 或运行 `yarn build`。

特效名会直接对应文件路径：

``` text
pixiPerform:myPerform;
    -> game/pixi-performs/myPerform.js

pixiPerform:weather/rain;
    -> game/pixi-performs/weather/rain.js
```

例如，在 `game/pixi-performs/myPerform.js` 中写入：

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

纹理可以放在 `game/tex` 目录下。`fg` 使用前景层；如果需要背景特效，使用 `bg` 并将容器加入 `stage.backgroundEffectsContainer`。

每次调用都必须同步返回 `{ container, tickerKey }`，动画实例的 `tickerKey` 应保持唯一。WebGAL 在特效被清除时会使用这两个值销毁容器并移除动画。

先初始化 Pixi，再在剧本中调用与文件名对应的特效：

``` ws
pixiInit;
pixiPerform:myPerform;
```

修改特效文件后刷新预览页面即可生效，不需要重新编译引擎。特效文件是会在游戏页面中执行的可信 JavaScript，不要使用来源不明的特效文件。
