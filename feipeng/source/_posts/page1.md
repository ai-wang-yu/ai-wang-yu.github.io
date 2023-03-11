---
title: Vue项目中后端返回数字（code），前端显示中文（name）
date: 2020-06-21 23:28:58
tags:
    - Array
categories: Array
# no_toc: true
---

介绍了数组循环的几种方法，三元表达式优化
<!--more-->

比如：现在有一个数组

``` json
arr: [
    {"code": "0101", "name": "苹果"},
    {"code": "0102", "name": "荔枝"},
    {"code": "0103", "name": "甘蔗"},
    {"code": "0104", "name": "香蕉"},
    {"code": "0105", "name": "龙眼"},
]
```
后端返回一个值fruit: "0103"，页面上的showChinese这个值需要显示对应的中文

### 解决方法

直接上代码

``` js
this.arr.forEach(e=>{
    if (e.code===this.fruit) {
        this.showChinese = e.name
    }
})
```

这样就可以显示对应的中文了，但是呢写法比较low，所以改进一下

``` js
this.arr.forEach(e=>{
    e.code===this.fruit?this.showChinese = e.name:''
})
```
### 发现问题

``` js
this.arr.forEach(e=>{
    e.code===this.fruit?this.showChinese = e.name:''
    console.log(e.name, 'e.name')
})
```

### 打印结果

``` json
苹果 e.name
荔枝 e.name
甘蔗 e.name
香蕉 e.name
龙眼 e.name
```

我们可以看到，循环执行了5遍，当循环的数组比较大时，就会有问题了，那可不可以跳出循环呢

``` js
this.arr.forEach(e=>{
    if (e.code===this.fruit) {
        this.showChinese = e.name
        break
    }
})
```
这是写法是错误的，项目编译的时候就会报错，foreach中无法用break跳出循环，for循环，for in循环，for of循环是可行的
``` js
for (let index = 0; index < this.arr.length; index++) {
    if (this.arr[index].code===this.fruit) {
        this.name = this.arr[index].name
        break;
    }
}
for (let key in this.arr) {
    if (this.arr[key].code===this.fruit) {
        this.showChinese = this.arr[key].name
        break
    }
}
for (let key of this.arr) {
    if (key.code===this.fruit) {
        this.showChinese = key.name
        break
    }
}
```

具体区别，可以查看[for/for in/for of/forEach/map/filter/some/every区别对比](https://juejin.im/post/5bc98082e51d450e4369bace)

### 推荐的写法

try，catch写法个人感觉比较麻烦，下面是比较推荐的写法

``` js
this.arr.some(e=>{
    return e.code===this.fruit?this.showChinese = e.name:''
})
```

### 思考

一般业务中这种情况都是通过select双向绑定来实现，但是作者遇到一种业务场景，后端传值code，前端需要显示，我这里用的自定义组件selsect来回显的，这个时候又要给这个元素的父类加一个点击事件，我发现呢，点击其他区域是可以触发click事件，但是自定义的select组件无法触发父类元素绑定的click事件，所以我用了some函数。后面我度娘发现可以写一个[filter](https://cn.vuejs.org/v2/api/#Vue-filter)函数，适合项目比较大有多个地方需要回显中文的情况。
有别的更好的方法，欢迎留言评论。

------

欢迎留言评论交流
转载本博客的文章请注明原始出处和作者