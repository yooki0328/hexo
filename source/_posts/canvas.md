---
title: canvas学习之旅
date: 2016-09-15 04:16:30
tags: [canvas]
---
canvas之前有过一点点了解，最近看到了一些很酷的demo，好佩服～所以从今天开始好好看看html5的canvas.Yeah~<!--more-->
***canvas***

### 基本知识
* context: var context = canvas.getContext(‘2d’)
* context.fill() 填充 context.stroke() 绘制边框
* context.fillStyle()填充样式 context.strokeStyle()边框样式
* context.lineWidth()图形边框宽度
* 绘制矩形 context.fillRect(x,y,width,height) strokeRect(x,y,width,height)
* 清除矩形区域 context.clearRect(x,y,width,height)
* 圆弧context.arc(x,y,radius,starAngle,endAngle,anticlockwise)
* 路径context.beginPath() context.closePath() 如果画完前面的路径没有指定beginPath，那么画其他路径时候会将前面指定的beginPath重新绘制
* 绘制线段 context.moveTo(x,y) context.lineTo(x,y)每次画线都从moveTo的点到lineTo的点
* 线性渐变 var lg=context.createLinearGradient(xStart,yStart,xEnd,yEnd)
* 线性渐变颜色 lg.addColorStop(offset,color)
* 径向渐变(发散) var rg=context.createRadialGradient(xStart,yStart,radiusStart,xEnd,yEnd,radiusEnd)
* 径向渐变颜色:rg.addColorStop(offset,color)
* 图形变形
1. 平移context.translate(x,y) x:坐标原点向x轴方向平移X y：坐标原点向y轴方向平移y
2. 缩放context.scale(x,y) x:x坐标轴按x比例缩放，y:y坐标轴按y比例缩放
3. 旋转context.rotate(angle) angle:坐标轴旋转x角度
4. 矩阵变换context.transform(a,b,c,d,e,f);
a:水平缩放绘图
b:水平倾斜绘图
c:垂直倾斜绘图
d:垂直缩放绘图
e:水平移动绘图
f:垂直移动绘图
* 图形组合 cotext.globalCompositeOperation=type
1. source-over: 在原有图形上绘制新图形
2. destination-over:在原有图形下绘制新图形
3. source-in:显示原图与新图的交集，新图在上
4. destination-in:显示交集，原图在上
5. source-out：显示新图形非交集部分
6. destination-out:显示原图形非交集部分
7. source-atop:显示原有图形和交集部分，新图形在上
8. destination-atop:显示新图形和交集部分，原图在上
9. lighter:原有图形和新图形都显示，交集部分做颜色叠加
10. xor：显示非交集部分
11. copy:只显示新图形
* 绘制阴影
1. context.shadowOffsetX:阴影的横向位移量
2. context.shadowOffsetY:阴影的纵向位移量
3. context.shadowColor:阴影的颜色
4. context.shadowBlur:阴影的模糊范围
* 绘制文字
1. 填充文字:context.fillText(text,x,y)
2. 绘制文字轮廓:context.strokeText(text,x,y)
  text:要绘制的文字
　　x:文字起点的x坐标轴
　　y:文字起点的y坐标轴
　　context.font:设置字体样式
　　context.textAlign:水平对齐方式: start、end、right、center
　　context.textBaseline:垂直对齐方式:top、hanging、middle、bottom
　　var length=context.measureText(text):计算字体长度(px)

* 保存和恢复状态
　　保存:context.save() 调用该方法,会保存当前context的状态、属性
　　恢复:context.restore()　调用该方法就能恢复到savｅ时候context的状态、属性
　   保存文件　canvas.toDataURL()
总结就是今天暂时把基础知识看完了，跟着做了一些demo
接下来做几个有意思的动画巩固一下，代码顺便传到github上去
