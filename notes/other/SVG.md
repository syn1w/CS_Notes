# 一、SVG

SVG(Scalable Vector Graphics)，是一种 XML 的应用。用于二维矢量图形，三维矢量图形可以使用 X3D。可以在 Web 浏览器中显示 SVG

SVG 也可以嵌入到 HTML 中，`iframe`，`img`，`svg`，`embed`，还有可以作为背景图片

```html
<iframe src="svgImage.svg" width="200" height="200" />
<img src="svgImage.svg"/>

<svg>
    <circle cx="40" cy="40" r="24" style="stroke:#006600; fill:#00cc00"/>
</svg>

<embed src="svgImage.svg"
       width="300" height="220"
       type="image/svg+xml"
       pluginspage="http://www.adobe.com/svg/viewer/install/" />
```

```css
div {
    background-image: url('my-svg-image.svg');
    background-size : 100px 100px;
}
```



## 1. 图形系统

栅格图像(*raster graphics*) 和 矢量图像(*vector graphics*)

栅格图像表现为图片元素或像素的长方形数组，这一系列像素也称为位图(*bitmap*)。

矢量图像，图像被描述为一系列的几何形状，矢量图形阅读器接受在指定坐标集上绘制图形的指令，而不是接受一系列已经计算好的像素。

矢量图形被用于：

- 计算机辅助绘图(CAD)
- 高分辨率的打印图形
- 打印和成像语言
- 基于矢量图形的 Flash 系统

矢量图形缩放不损失图像质量。



## 2. SVG 图像

**基本形状**

可以绘制直线(`line`)，矩形(`rect`)，圆(`circle`)，椭圆(`ellipse`)，折线(`polyline`)，多边形(`polygen`)，路径(`path`，可以绘制高级图像)、文本(`text`)。

```xml
<svg width="140" height="170" xmlns="http://www.w3.org/2000/svg">
	<circle cx="70" cy="95" r="50" stroke="black" fill="none" />
</svg>
```


还可以进行分层、设置透明度、转换(`transform`)，渐变、链接、动画、统计图



## 3. SVG 坐标系

SVG 坐标系 (0, 0) 在**左上角**。

SVG 元素坐标值的长度单位

| Unit |                        Description                         |
| :--: | :--------------------------------------------------------: |
|  em  | The default font size - usually the height of a character. |
|  ex  |               The height of the character x                |
|  px  |                           Pixels                           |
|  pt  |             Points(1 / 72 of an inch = 0.03cm)             |
|  pc  |             Picas (1 / 6 of an inch = 0.42 cm)             |
|  cm  |                        Centimeters                         |
|  mm  |                        Millimeters                         |
|  in  |                      Inches(2.54 cm)                       |

在 `svg` 元素上的 `width` 和 `height` 只会影响 `<svg>` 元素，默认单位是像素(px)。



## 4. viewBox