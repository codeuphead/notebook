# 盒模型

- [盒模型](#盒模型)
  - [盒子类型](#盒子类型)
  - [盒子的组成部分](#盒子的组成部分)
    - [content](#content)
    - [padding](#padding)
    - [border](#border)
    - [margin](#margin)
  - [盒模型应用](#盒模型应用)
    - [改变宽高范围](#改变宽高范围)
    - [改变背景覆盖范围](#改变背景覆盖范围)
    - [溢出处理](#溢出处理)
    - [断词规则](#断词规则)
    - [空白处理](#空白处理)
  - [行盒的盒模型](#行盒的盒模型)
    - [显著特点](#显著特点)
    - [行块盒](#行块盒)
    - [空白折叠](#空白折叠)
    - [可替换元素 非可替换元素](#可替换元素-非可替换元素)
    - [`object-fit` 图片的适应方式](#object-fit-图片的适应方式)

box：盒子，每个元素在页面中都会生成一个矩形区域，称为盒子

## 盒子类型

1. 行盒，display 属性是 inline 的元素
2. 块盒，display 属性是 block 的元素

display 默认值为 inline

浏览器默认样式表设置的块盒：容器元素，h1~h6，p

常见行盒：span a img video audio

## 盒子的组成部分

从内到外：

1. 内容 content
2. 填充 padding
3. 边框 border
4. 外边距 margin

### content

width、height 设置的是盒子内容的宽高

内容部分通常叫做整个黑子的**内容盒 content-box**

### padding

盒子边框到盒子内容的距离

padding-left、padding-right、padding-top、padding-bottom

padding：简写属性，最终由计算机进行转换

`padding: 上 右 下 左;`

`padding: 上下 左右;`

`padding: 一样;`

填充区 + 内容区 = **填充盒 padding-box**

### border

边框 = 样式 + 宽度 + 颜色

```css
border-styleborder-widthborder-colorborder: width style color;
```

这些属性都是简写属性，可以分别设置 4 个方向的属性。浏览器中，属性右边有个箭头的都是。

边框 + 填充区 + 内容去 = **边框盒 border-box**

### margin

边框到其他盒子的距离

## 盒模型应用

### 改变宽高范围

默认情况下，width 和 height 设置的是内容盒的宽高。

> 页面重构师：将设计稿制作为静态页面

但是设计稿通常测量的是边框盒，但是设置的是内容盒，就会出现一些问题

1. 精确计算
2. CSS3：box-sizing 可以设置盒子宽高的范围

`box-sizing` 默认是 `content-box` 可以设置为 `border-box` 改变 width height 影响的范围为边框盒

### 改变背景覆盖范围

默认情况下，背景覆盖边框盒。

可以通过 `background-clip` 设置背景的覆盖范围

### 溢出处理

设置了高度后，内容可能会溢出，默认是可以看见的

通过 `overflow` 控制内容溢出边框盒后的处理方式，默认是 visible

### 断词规则

`word-break` 影响断词位置，默认是 normal（CJK 字符，在字符位置截断，非 CJK 字符，在单词位置截断）

`break-all` 所有字符都在文字处截断

### 空白处理

`white-space` 空白处理规则，`nowrap` 不换行

设置 `overflow:hidden; text-overflow:eliipsis;` 显示溢出的地方，会...显示

单行文本通过这样处理，如果是多行文本，需要通过 JS 处理

## 行盒的盒模型

### 显著特点

- 盒子沿着内容延伸
- 行盒不能设置宽高，设置了无效，取决于内容，应该调整字体大小、行高等间接调整
- 填充区 padding，水平方向有效，垂直方向只影响背景，不会占用实际空间
- 边框，与 padding 一样
- 外边距，与 padding 一样

### 行块盒

`display: inline-block` 的盒子

- 不独占一行
- 盒模型中，所有尺寸都有效

### 空白折叠

空白折叠，发生在行盒内部 或 行盒之间 （包含行块盒），不换行就能消除

### 可替换元素 非可替换元素

大部分元素，页面上显示的结果，取决于元素内容，称为**非可替换元素**

少部分元素，页面上显示的结果，取决于元素属性，成为**可替换元素**

可替换元素：img，video，audio

绝大部分可替换元素均为行盒。

可替换元素，类似于行块盒，盒模型中所有尺寸都有效

### `object-fit` 图片的适应方式

- contain:保持比例，不裁剪
- fill：默认填充
- cover：保持比例填充满，裁剪
