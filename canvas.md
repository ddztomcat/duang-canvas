## canvas知识集锦
### 基础知识
+ 默认canvas背景色与父元素一致，canvas内容部分在不支持canvas才会显示
```javascript
<canvas>不支持canvas</canvas>
```
+ canvas元素大小与绘图坐标系统都是默认300*150，使用css修改只会更改canvas元素大小；而通过修改width和height两者都会修改，当使用css修改时，浏览器可能会自动缩放canvas。设置width，height不建议带`px`
```css
#canvas {
  width: 600px;
  height: 300px;
}
```
```javascript
<canvas width="600" height="300"></canvas> // 同时修改元素大小 绘制元素坐标系统
<canvas id="canvas"></canvas> // 只修改元素大小 坐标系统仍然300*150
```
+ stroke 绘制已定义的路径
+ fill 填充当前绘制图形。如果路径未关闭，会先关闭该路径，在填充该路径
+ canvas 是不可获取焦点元素，keydown => 如果该字符可以打印触发keypress，持续按则持续触发 => keyup
+ 立即模式绘图系统
  canvas将你的内容绘制到canvas上之后，就会立刻忘记刚才绘制的内容，这表示canvas之中不会包含将要绘制的图形对象列表。而SVG则会维护一份所绘图形对象的列表，该模式为“保留模式”绘图系统
### 绘制
+ 绘制阴影效果可能会很耗时，这与canvas的绘制模型有关
+ strokeRect strokeText fillRect fillText 都是立即绘制，对于复杂的形状大都使用基于路径（path）的
>与路径相关的方法
---
|方法|描述|
|---|---|
|arc()|绘制弧形|
|beginPath()|重置当前路径|
|closePath()|封闭路径|
|fill()|对路径内部填充|
|rect()|绘制矩形路径|
|stroke()|绘制轮廓|
+ 路径与子路径，在某一时刻，canvas中只能由有一条路径存在，canvas将其称为“当前路径”，但是这条路径可以包含很多子路径。
```javascript
// 1
context.beginPath()
context.rect(10,10,100,100)
context.stroke()
context.beginPath()
context.rect(10,10,100,100)
context.stroke()

// 2
context.beginPath()
context.rect(10,10,100,100)
context.stroke()
context.rect(10,10,100,100)
context.stroke() // 没调用beginPath 此时描边会将第一个矩形描边重绘
```
+ fill 填充路径时使用“非零环绕规则”