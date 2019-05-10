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
+ 绘制1像素宽的线条，需要定位到某个像素的中间即整数+0.5
+ 在默认情况下，剪辑区域的大小与canvas一致，除非你手动调用clip()方法设置当前路径与当前剪辑区域的交集
### 文本
|与文本相关的属性方法|
|---|
|strokeText(text, x, y, width)|
|fillText(text, x, y, width)|
|measureText(text)|
|font|
|textAlign|
|textBaseline|
+ font属性不支持inherit initial，另外在canvas设置line-height时，浏览器会忽略其值，规范要求浏览器必须将该值设置为normal
+ textAlign属性默认值是start，当canvas的dir属性是ltr时left=start，反之，同理
+ 在使用measureText()方法时先设置好字型，它总是基于当前字型计算宽度
+ 如果要渲染或移除文本，必须将剪辑区域所覆盖的整个canvas范围都替换掉
### 图像与视频
+ 图像的绘制效果受制于阴影、剪辑区域、图像合成等属性，但不会考虑当前路径
+ getImageData()返回的对象的data属性，包含各个设备像素数值的数组,width、height属性单位是像素
+ putImageData(image, dx, dy, dirtyX, dirtyY, width, height)后四个参数表示以设备像素为单位的脏矩形，并且该函数不受全局设置的影响
+ canvas允许绘制不属于自己域中的图像，但是不能通过api操作其他域的图像，origin-clean默认true，当是其他域下的图片时，会被置成false
>性能分析

1. putImageData()总要比drawImage()慢，而且drawImage()支持canvas
2. 将canvas绘制到自身很耗时，绘制canvas时对其缩放也很耗时
3. 遍历图像数据时，应避免直接访问对象属性，应该将其放到局部变量中
4. 应该使用循环计数器遍历完整像素，i+=4,而非i++
5. 不要频繁的调用getImageData()
### 动画
+ 制作动画不用setTimeout setInterval,因为浏览器为了节省电力消耗，可以拉长执行代码的间隔时间，也就是“强制规定时间间隔的下限”
+ 制作动画时处理背景

1. 将所有内容擦除，并重新绘制
2. 仅重绘内容发生改变的区域，即利用剪辑区域
3. 从离屏缓冲区将内容变化的背景复制到屏幕上，简称“图块复制”
>一般来说图块复制要比剪辑区域快，但是它需要一个离屏canvas，会占据更多内存

+ 双缓冲技术，解决clearRect()方法可能带来的白屏闪烁，该方案浏览器已经内置
+ 基于时间的运动，即动画中物体每秒移动的像素p恒定，每帧移动的像素v = p / fps
+ 时间轴扭曲函数，实际都是基于linear动画效果的时间映射函数，类似于在匀速直线运动的基础上衍生出各种加速，匀加速，变速运动
### 碰撞
>检测方法
1. 外接矩形检测或者外接圆
2. 光线投射法
3. 分离轴定理只适用于凸多边形
向量的数量积可以用来计算投影