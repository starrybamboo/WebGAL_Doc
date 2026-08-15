# 动画效果

## 为背景或立绘设置动画

使用语句 `setAnimation:动画名 -target=作用目标;`

**示例：**

``` ws
; // 为中间立绘设置一个从下方进入的动画，并转到下一句
setAnimation:enter-from-bottom -target=fig-center -next;
```

目前，预制的动画有：

| 动画效果 | 动画名 | 持续时间（毫秒） |
| :--- | :--- | :--- |
| 渐入 | enter | 300 |
| 渐出 | exit | 300 |
| 左右摇晃一次 | shake | 1000 |
| 从下侧进入 | enter-from-bottom | 500 |
| 从左侧进入 | enter-from-left | 500 |
| 从右侧进入 | enter-from-right | 500 |
| 前后移动一次 | move-front-and-back | 1000 |
| 模糊进入 | blur | 300 |
| 老电影滤镜 | oldFilm | 0 |
| 点状滤镜 | dotFilm | 0 |
| 反射滤镜 | reflectionFilm | 0 |
| 故障滤镜 | glitchFilm | 0 |
| RGB 分离滤镜 | rgbFilm | 0 |
| 光辉滤镜 | godrayFilm | 0 |
| 移除电影类滤镜 | removeFilm | 0 |
| 冲击波入场 | shockwaveIn | 2000 |
| 冲击波退场 | shockwaveOut | 2000 |

这些名称来自 `game/animation/animationTable.json`。持续时间为 0 的预设通常用于立刻设置或清除滤镜状态。

目前，动画的作用目标有：

| target     | 实际目标       |
| :--------- | :------------ |
| fig-left   | 左立绘         |
| fig-center | 中间立绘       |
| fig-right  | 右侧立绘       |
| bg-main    | 背景           |
| id | 某个有id的立绘 |

## 自定义动画

### 创建动画

动画文件在 `game/animation`，你可以创建自己的自定义动画。

动画文件使用 JSON 描述，你可以在 [参考文档](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/JSON) 参考JSON语法。

每一个动画文件都代表一个**动画序列**，使用一个 JSON 数组来描述。下面是一个示例，描述了一个从左侧进入的动画：

**示例: `enter-from-left.json`**

``` json
[
  {
    "alpha": 0,
    "scale": {
      "x": 1,
      "y": 1
    },
    "position": {
      "x": -50,
      "y": 0
    },
    "rotation": 0,
    "blur": 5,
    "duration": 0
  },
  {
    "alpha": 1,
    "scale": {
      "x": 1,
      "y": 1
    },
    "position": {
      "x": 0,
      "y": 0
    },
    "rotation": 0,
    "blur": 0,
    "duration": 500
  }
]

```

其中各个属性代表的释义为：

| 属性名   | 释义                                |
| :------- | :--------------------------------- |
| alpha    | 透明度，范围0-1                     |
| scale    | 缩放                               |
| position | 位置偏移                           |
| rotation | 旋转角度，单位为弧度                |
| blur     | 高斯模糊半径                        |
| brightness  | 调节亮度                        |
| contrast    | 调节对比度                       |
| saturation  | 调节饱和度                      |
| gamma       | 调节伽马值                      |
| colorRed    | 颜色分量:红色, 范围0-255          |
| colorGreen  | 颜色分量:绿色, 范围0-255          |
| colorBlue   | 颜色分量:蓝色, 范围0-255          |
| duration | 这个时间片的持续时间，单位为毫秒(ms) |
| oldFilm         | 老电影效果，0代表关闭，1代表开启       |
| dotFilm         | 点状电影效果，0代表关闭，1代表开启     |
| reflectionFilm  | 反射电影效果，0代表关闭，1代表开启     |
| glitchFilm      | 故障电影效果，0代表关闭，1代表开启     |
| rgbFilm         | RGB电影效果，0代表关闭，1代表开启      |
| godrayFilm      | 光辉效果，0代表关闭，1代表开启         |

然后，你需要在 `animationTable`中加上你的自定义动画的文件名（不需要后缀名）

在文件 `animationTable.json`：

``` json
["enter-from-left","enter-from-bottom","enter-from-right"]
```

然后，你就可以在脚本中调用：

``` ws
setAnimation:enter-from-left -target=fig-left -next;
```

### 省略部分属性

如果你的动画只需要操作部分属性，你可以将其他不参与动画的属性留空，使它们保持默认。

**示例：`enter.json`**

``` json
[
  {
    "alpha": 0,
    "duration": 0
  },
  {
    "alpha": 1,
    "duration": 300
  }
]

```

### 使用变换

一个持续时间为0毫秒，且只有一个时间片的动画就是变换。

**示例：**

``` json
[
  {
    "alpha": 0,
    "duration": 0
  }
]
```

## 设置进出场效果

你还可以覆盖 WebGAL 默认的渐变进出场效果，替换为自己的动画效果，例如：

```
setTransition: -target=fig-center -enter=enter-from-bottom -exit=exit;
```

注意：只有当立绘或背景被设置后，你才能为其设置进出场效果。设置进出场效果的代码写在立绘或背景的设置代码**后**。并且，设置进出场效果的语句必须紧随设置立绘或背景的语句**连续执行**，否则无法被正确应用。

::: tip
为什么要这样做？

在设置立绘或背景时，会为立绘或背景默认设置一个进出场动画。在设置了立绘或背景后，立绘和背景不会立即展示在舞台上，而是会等待进出场效果的设置。

如果在立绘或背景设置后，立即执行设置进出场效果的语句，就可以覆盖掉默认的进出场动画，进而实现进出场效果的自定义。如果没有设置，则在进出场时执行默认动画。

如果不在立绘或背景设置后立即执行进出场效果的设置，等到图像已经进场了，再覆盖进场动画就没有意义了。但如果此时图像还没有出场，设置的出场动画仍有意义。其会在立绘或背景出场时正确地被应用。
:::

## 图片角色的面部动画

图片角色的眨眼与嘴型应在角色目录的 `figure.json` 中声明 `facialRig`，再通过 `character` 命令上场。底图使用睁眼、闭嘴状态，眼睛和嘴巴只需提供小型替换片：

```json
{
  "Version": 1,
  "canvas": { "width": 1024, "height": 1536 },
  "components": {
    "base": { "src": "base.png", "x": 0, "y": 0 }
  },
  "presets": {
    "default": {
      "items": ["base"],
      "facialRig": {
        "eyes": {
          "x": 485, "y": 228, "width": 168, "height": 113,
          "half": "eyes-half.png", "closed": "eyes-closed.png"
        },
        "mouth": {
          "x": 531, "y": 317, "width": 82, "height": 73,
          "halfOpen": "mouth-half.png", "open": "mouth-open.png"
        }
      }
    }
  }
}
```

```webgal
character:hero/default -id=charA;
角色A:你好，世界！ -vocal=charA_hello.wav -figureId=charA;
```

统一面部运行时会组合独立的眨眼和嘴型轨道；有语音时按音频时间线驱动嘴型，无语音时按文本节奏驱动。旧的 `changeFigure` 整图嘴型、眼睛差分参数已移除，请将相关素材迁移到 `facialRig`。该运行时只适用于通过 `character` 上场的位图 `facialRig`，不接管 Live2D 或 Spine。
