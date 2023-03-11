---
title: Flex:1是什么
date: 2020-06-23 21:28:58
tags:
    - Css
    - 前端面试
categories: Css
# no_toc: true
---

Flex:1是一个简写
<!--more-->

### 横向用法

日常项目中要实现下面这样的效果是不好实现的，左侧内容是跟随屏幕变化的无法给固定的宽度，用百分比或者rem也不行，右侧需要完全占据剩余区域，这样点击右侧任何一个地方都可以触发下拉或者输入

![横向效果图]( https://pic.downk.cc/item/5ef4c8df14195aa5945e6572.png )

```html
<div style="display:flex;justify-content: flex-start;">
    <div>资料寄送地址</div>
    <div style="flex:1">下拉框</div>
</div>
```

效果图内部样式就不写了，垂直居中可以使用flex的另一个属性

```css
align-items: center;
```

### 竖向用法

日常项目中很多按钮要求居于页面底部，但是现在屏幕尺寸很多，给出固定的高度是不行的，使用媒体查询又太麻烦了

![竖向效果图]( https://pic.downk.cc/item/5ef4c8f114195aa5945e6c0a.png )

```html
<div style="display: flex;flex-direction: column;height: 100%;">
    <div style="flex:1">上方</div>
    <div style="height: 40px;margin-bottom: 60px;">下方button</div>
</div>
```

需要注意的是下方button块需要一个固定的高度，父容器需要设置高度100%。

### 总结

flex: 1 等同于 flex: 1 1 0

flex: 1浏览器中解析为flex-grow:1 flex-shrink:1 flex-basis:0%

flex: 1 1 0浏览器中解析为flex-grow:1 flex-shrink:1 flex-basis:0px

flex: 1 实际上是三个属性的缩写：flex-grow:1 flex-shrink:1 flex-basis:auto

 flex-grow:：当父控件还有剩余空间的时候，是否进行放大(grow)其中数值代表的是放大比例，值为0的时候表示不放大

flex-shrink：当父控件空间不够的时候，是否进行缩小(shrink)其中数值代表的是与控件大小有关的缩小比例

flex-basis：当子空间含有这个属性的时候，代表了子空间占主轴的大小，主轴就是flex的主方向row是横向，column是竖向，(这里一般是指width，如果flex方向是column也可以是height)，会覆盖原有的width(或height)属性

### 其他属性

flex-direction属性 决定主轴(一般是x轴)的方向，即项目排列的方向 

+  **row**:主轴为水平方向，项目沿主轴从**左至右**排列 
+  **column**：主轴为竖直方向，项目沿主轴**从上至下**排列 
+  **row-reverse**：主轴水平，项目从**右至左**排列，与row反向 
+  **column-reverse**：主轴竖直，项目从**下至**上排列，与column反向 

align-item与align-conter区别

- align-items： 适用于单行情况下 只有 上对齐 下对齐 居中和拉伸 属性值
- align-content ：适用于换行（多行）的情况下（单行无效） 可以设置上对齐 下对齐 居中 拉伸和平均分配剩余空间等属性值，需要配合flex-wrap：wrap使用



更多Flex布局请看[总结flex布局，一眼看懂]( https://www.jianshu.com/p/300575490dcf )

------

欢迎留言评论交流
转载本博客的文章请注明原始出处和作者