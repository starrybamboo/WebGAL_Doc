# エフェクト

現在、WebGAL のエフェクトシステムは PixiJS で実装されています。

## エフェクトを使用する

### Pixi を初期化する

`pixiInit` を使用して Pixi を初期化します。

``` ws
pixiInit;
```

::: warning
エフェクトを使用する場合は、このコマンドを最初に実行して Pixi を初期化する必要があります。

すでに適用されているエフェクトを消去したい場合は、この構文を使用してエフェクトをクリアできます。
:::

### エフェクトを追加する

`pixiPerform` を使用してエフェクトを追加します。

``` ws
pixiPerform:rain; // 雨のエフェクトを追加する
```

注意：エフェクトを適用した後、再度初期化しないと、エフェクトは常に実行されます。

### デフォルトテンプレートのエフェクト

| エフェクト | コマンド                        |
| :--- | :-------------------------- |
| 雨 | pixiPerform:rain;           |
| 雪 | pixiPerform:snow;           |
| 大雪 | pixiPerform:heavySnow;    |
| 桜 | pixiPerform:cherryBlossoms; |

これら 4 つのエフェクトは、デフォルトゲームの `game/pixi-performs/` にあるランタイムスクリプトです。対応する `.js` ファイルで速度、パーティクル数、スケール、角度などを変更し、プレビューを更新すれば反映されます。エンジンの再ビルドは不要です。

### エフェクトを重ねる

2 つ以上エフェクトを重ねたい場合は、`pixiInit` コマンドを使用せずに異なるエフェクトを重ねることができます。

``` ws
pixiPerform:rain;
pixiPerform:snow;
```

### 重ねたエフェクトをクリアする

`pixiInit` を使用して初期化します。これにより、適用されているすべてエフェクトを消去できます。

## 再ビルド不要のカスタムエフェクト

ランタイムエフェクトに対応した WebGAL では、ゲームディレクトリに JavaScript ファイルを追加するだけで使用できます。エンジンソースの変更、`index.js` の管理、`yarn build` は必要ありません。

エフェクト名は次のようにファイルへ対応します。

``` text
pixiPerform:myPerform;
    -> game/pixi-performs/myPerform.js

pixiPerform:weather/rain;
    -> game/pixi-performs/weather/rain.js
```

例として、`game/pixi-performs/myPerform.js` に次を記述します。

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

テクスチャは `game/tex` に配置できます。`fg` は前景レイヤーを使用します。背景エフェクトには `bg` を使い、コンテナを `stage.backgroundEffectsContainer` に追加します。

各呼び出しは `{ container, tickerKey }` を同期的に返す必要があり、アニメーションの各インスタンスには一意の `tickerKey` を使用します。WebGAL はエフェクトの解除時にこれらの値を使ってコンテナとアニメーションを削除します。

Pixi を初期化してから、ファイル名に対応するエフェクトを呼び出します。

``` ws
pixiInit;
pixiPerform:myPerform;
```

エフェクトファイルを編集した後は、プレビューページを更新すれば反映されます。エンジンの再ビルドは不要です。エフェクトファイルはゲームページで実行される信頼済み JavaScript です。信頼できないソースのファイルは使用しないでください。
