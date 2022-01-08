+++
title = "UGUI源码分析(一): Mask 面具🎭之下"
date = 2022-01-05T22:12:45+08:00
lastmod = 2022-01-08T09:24:29+08:00
tags = ["Unity", "UGUI"]
categories = ["UGUI源码分析"]
draft = true
+++

`Mask 遮罩`  可以按自身的形状限制子元素的显示范围. 这句话有点抽象, 实际上自己动手做一下就很好理解.
例如, 我们常见的显示圆形头像的效果, 就是拿一个圆形的图像来限制方形头像的显示范围, 就能得到圆形头像.
下面我们就来看看Mask的背后的原理和其源码.


## 模板缓冲区和模板测试 {#模板缓冲区和模板测试}

`Mask` 的原理是模板缓冲区和模板测试. 模板缓冲区给每个像素分配一个8位的模板值, 渲染 `Mask` 时会写入指定的值.
子元素在渲染时, 会做模板测试, 如果不通过则舍弃子元素的片元, 通过则会正常渲染, 以此达到控制子元素显示范围的目的.


## Mask相关的类 {#mask相关的类}

`Mask` 相关的类和属性如下:


{{< figure src="/ox-hugo/2021-12-UGUI-Source-Reading-012.Mask-Hierarchy.png" >}}


## 一个Demo {#一个demo}

我们使用下面两张图来演示 `Mask` 的效果.

![](/ox-hugo/2021-12-UGUI-Source-Reading-014.Mask-Red-Hole.png)
![](/ox-hugo/2021-12-UGUI-Source-Reading-013.Mask-Blue.png)

我们新建两个 Image 控件, 分别使用这两张图做Sprite, 蓝色控件作为子对象, 并且给红色控件添加 `Mask` 组件.

我们得到如下图效果, 我们看到了蓝色图片, 但按照红色图片的形状显示.

{{< figure src="/ox-hugo/2021-12-UGUI-Source-Reading-015.Mask-Scene-Blue-Hole.png" >}}


## Mask的面具之下 {#mask的面具之下}

我们可以在Inspector里观察下, 禁用和启用 `Mask` 组件, 对 Image 控件使用材质的影响.

![](/ox-hugo/2021-12-UGUI-Source-Reading-016.Mask-Red-Disable.png)
![](/ox-hugo/2021-12-UGUI-Source-Reading-017.Mask-Red-Enable.png)

可以看到禁用 `Mask` 组件后, 使用的是 `Default UI Material`, 启用之后使用的材质被修改. 同时, 也能发现
子对象的材质也被修改.

我们打开 `Runtime/UI/Core/Mask.cs`, 并找到 `OnEnable` 方法代码如下:

```csharp
protected override void OnEnable()
{
    base.OnEnable();
    if (graphic != null)
    {
        graphic.canvasRenderer.hasPopInstruction = true;
        graphic.SetMaterialDirty();

        // Default the graphic to being the maskable graphic if its found.
        if (graphic is MaskableGraphic)
            (graphic as MaskableGraphic).isMaskingGraphic = true;
    }

    MaskUtilities.NotifyStencilStateChanged(this);
}
```


## 参考 {#参考}

-   <https://www.cnblogs.com/iwiniwin/p/15131528.html>
