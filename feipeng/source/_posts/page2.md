---
title: Flex:1的两种用法
date: 2020-06-23 21:28:58
tags:
    - Flex
    - Css
categories: Css
# no_toc: true
---

介绍Flex:1的两种用法
<!--more-->

## 一、横向用法

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

## 二、竖向用法

日常项目中很多按钮要求居于页面底部，但是现在屏幕尺寸很多，给出固定的高度是不行的，使用媒体查询又太麻烦了

![竖向效果图]( https://pic.downk.cc/item/5ef4c8f114195aa5945e6c0a.png )

```html
<div style="display: flex;flex-direction: column;height: 100%;">
    <div style="flex:1">上方</div>
    <div style="height: 40px;margin-bottom: 60px;">下方button</div>
</div>
```

flex-direction属性 决定主轴(一般是x轴)的方向，即项目排列的方向 

+  **row**:主轴为水平方向，项目沿主轴从**左至右**排列 
+  **column**：主轴为竖直方向，项目沿主轴**从上至下**排列 
+  **row-reverse**：主轴水平，项目从**右至左**排列，与row反向 
+  **column-reverse**：主轴竖直，项目从**下至**上排列，与column反向 

需要注意的是下方button块需要一个固定的高度，父容器需要设置高度100%。

Flex布局请看[总结flex布局，一眼看懂]( https://www.jianshu.com/p/300575490dcf )
有别的更好的方法，欢迎留言评论。

------

欢迎留言评论交流
转载本博客的文章请注明原始出处和作者