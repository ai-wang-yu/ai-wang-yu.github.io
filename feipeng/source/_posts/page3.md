---
title: 强大的moment.js
date: 2020-07-01 23:25:28
tags:
    - js
    - moment
categories: js
# no_toc: true
---

玩转moment
<!--more-->

使用安装就不多赘述了，直接进入正题

### 常用的几个方法（持续更新）

1. 获取当前时间  

   ```js
   console.log(moment().format("YYYY-MM-DD")) // 2020-07-01
   console.log(moment().format("YYYY年MM月DD日")) // 2020年07月01日
   ```

   

2.  获取当前日期前一天与后一天的日期   

   ```js
   console.log(moment('2020-07-01').subtract(1, 'days').format("YYYY-MM-DD")) // 2020-06-30
   console.log(moment('2020年07月01日').add(1, 'days').format("YYYY年MM月DD日")) // Invalid date
   console.log(moment('20200701').add(1, 'days').format("YYYY年MM月DD日")) // 2020年07月02日
   ```

   

3.  获取两个时间的差 值  

   ```js
   let end_date = moment().format("YYYY-MM-DD") // 2020-07-01
   let start_date = moment(end_date).subtract(1, 'month').format("YYYY-MM-DD") // 2020-06-01
   moment(end_date).diff(start_date, 'months') // 1
   moment('2020-07-31').diff(start_date, 'months') // 1
   moment('2020-06-30').diff(start_date, 'months') // 0
   moment(end_date).diff(start_date, 'weeks') // 4
   moment(end_date).diff(start_date, 'days') // 30
   ```

   

### 发现问题

diff()方法2020-07-01减去2020-06-01值为1，2020-07-31减去2020-06-01值也为1，2020-06-30减去2020-06-01值也为0

### 项目实践

有两个输入框，开始时间跟结束时间，要求：

1. 只能输入今天以及之前一年的时间
2. 开始时间与结束时间最多相差一个月
3. 隐藏要求，结束时间不能小于且不能等于开始时间

   ```js
   let end_date = moment().format("YYYY-MM-DD") // 2020-07-01
   let start_date = moment(end_date).subtract(1, 'year').format("YYYY-MM-DD") // 2019-07-01
   ```

这样可以控制第一个条件，第二个条件会出现**发现问题**那样的情况，首先我们可以通过diff()函数取得两个时间月份month的差，然后在去判断两个时间天数days的差，月份差为1，天数差小于等于31就是符合条件的，但是好像有点麻烦了，下面看代码

   ```js
   let start_date = '2020-06-01' // 输入开始时间
   let end_date1 = '2020-07-01' // 输入结束时间
   let end_date2 = '2020-07-02' // 输入结束时间
   let end_date3 = '2020-05-31' // 输入结束时间
   let month1 = moment(end_date1).subtract(1, 'days').diff(start_date, 'months') // 输出0
   let month2 = moment(end_date2).subtract(1, 'days').diff(start_date, 'months') // 输出1
   当month大于0的时候，就是不符合条件
   let day1 = moment(end_date3).diff(start_date, 'days') // 输出-1
   当day小于1的时候，就是不符合条件
   ```

更多内容可以查看[moment.js中文网](http://momentjs.cn/)

------

欢迎留言评论交流
转载本博客的文章请注明原始出处和作者