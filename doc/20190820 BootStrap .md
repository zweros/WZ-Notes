---
title: 20190820 Bootstrap
date: 2019-08-20
---

## Bootstrap

> 经典且流行，**UI 框架**

BootStrap 网站： https://getbootstrap.com/

BootStrap 中文网站：https://www.bootcss.com/

BooStrap 中文手册：https://v3.bootcss.com/css/

## Boostrap 开始 ##

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <!-- 视口设置：用于移动端大小适配 -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- BootStrap css -->
    <link href="lib/bootstrap/css/bootstrap.css" rel="stylesheet" />
    <!-- jquery 必须加载在 bootstrap.js 之前 -->
    <script src="lib/jquery/jquery.js"></script>
    <script src="lib/bootstrap/js/bootstrap.js"></script>
</head>
<body>
</body>
</html>
```



## 视口 ##

> 在移动设备浏览器上，通过为视口（viewport）设置 meta 属性为 `user-scalable=no` 可以禁用其缩放（zooming）功能。这样禁用缩放功能后，用户只能滚动屏幕，就能让你的网站看上去更像原生应用的感觉。注意，这种方式我们并不推荐所有网站使用，还是要看你自己的情况而定！

```html
<!-- 视口设置：用于移动端大小适配 -->
<meta name="viewport" content="width=device-width, initial-scale=1">

<meta 
     name="viewport" 
     content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```

## 栅格系统 ##

> 栅格系统（grid system） 是bootstrap 创建的一个用于页面构建的 html 布局系统。该系统通过提供的类名来将容器等分为12个结构。通过对容器的宽度进行页面的快速构建

### 说明： ###

1. 响应式布局（一套代码，能够在不同的页面中有不同的加载效果
2. 提供了很预定义的类名，通过使用这些类名能够快速获得 css 样式）

### 内容： ###

（1）container : 通过使用 .container 类名，来定义一个栅格容器

（2）row：通过使用  .row 类名, 来定义一个栅格中的行容器

（3）col—[screenStyle]-[percent]: 通过使用复合类名，来定义栅格行中的一个栅格大小

​		1. screenStyle 设备类名(样式能够生效的设备类型)

| 类名       | 说明                                         |
| ---------- | -------------------------------------------- |
| col-xs-... | 超小型设备，width <= 768px                   |
| col-sm-... | 小型设备（width > 768px && width < =992px ） |
| col-md-... | 中型设备 (width > 992px && width < 1200px)   |
| col-lg-... | 大型设备（width > 1200px）                   |

   2. percent ： 栅格在一行中占有多少列，可选值为 1~12

### 补充： ###

   栅格 container 能够赋予页面元素一个默认水平居中的效果

​	其实就是通过设置了 margin 和 padding



​         

### 栅格系统实现原理 ###

- **使用 Js 控制 div 宽度**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>container 学习</title>
    <style>
        * {
            padding: 0px;
            margin: 0px;
            list-style: none;
        }
        .container{
            height: 40px;
            margin: 0 auto;
            background-color: red;
        }
    </style>
</head>
<body>
    <div class="container">容器学习</div>

    <script type="application/javascript">
        // 当页面加载时触发
        window.addEventListener("load", function () {
            // 1. 获取常量
            let container = document.querySelector(".container");
            let clientW = 0;
            // 当第一次进入页面，调用 resize 方法
            resize();

            // 2.1 监听窗口变化
            window.addEventListener("resize",resize);

            function resize() {
                clientW = window.innerWidth;
                //console.log(clientW)
                // 判断
                if(clientW >= 1200){//超大屏幕
                    container.style.width = "1178px";
                }else if(clientW >= 992){// 中等屏幕
                    container.style.width = "970px";
                }else if(clientW >= 768){// 小屏幕
                    container.style.width = "750px";
                }else{//超小屏幕
                    container.style.width = "100%";
                }
            };
        });

    </script>
</body>
</html>
```

- **使用 css 媒体查询**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>媒体查询</title>
    <style>
        * {
            padding: 0px;
            margin: 0px;
            list-style: none;
        }
        .container{
            height: 40px;
            margin: 0 auto;
            background-color: red;
        }
        /*媒体查询*/
        @media screen and (min-width: 768px) {
            .container{
                width: 100%;
            }
        }
        @media screen and (min-width: 768px) and (max-width: 992px){
            .container{
                width: 750px;
            }
        }
        @media screen and (min-width: 992px) and (max-width: 1200px){
            .container{
                width: 970px;
            }
        }
        @media screen and (min-width: 1200px){
            .container{
                width: 1170px;
            }
        }
    </style>
</head>
<body>
    <div class="container"></div>
</body>
</html>
```

## CSS 属性选择器 ##

#### 属性选择器之模糊匹配 ####

- 代码

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>属性选择器</title>
    <style>
        [class^="icon-"],[class*="icon-"]{
            width: 150px;
            height: 50px;
            background-color: #2aabd2;
        }
    </style>
</head>
<body>
    <div class="icon-111">1111111111</div>
    <div class="text icon-222">2222222222</div>
    <div>3333333333</div>
    <div class="icon-44444">4444444444</div>
    <div>5555555555</div>
</body>
</html>
```

- 效果

![](http://img.zwer.xyz/blog/20191002191810.png)

## 图片显示 ##

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div{
            width:800px;
            height:410px;
            border:1px solid #000;
            margin: 20px auto;
            background: no-repeat center center;
            -webkit-background-size: cover;
            /*background-size: 50% 50%;*/
        }
    </style>
</head>
<body>
    <div style="background-image: url('imgs/wallhaven-kw1ky1_1920x1080.png')"></div>
</body>
</html>
```

## Jquery data 传值 ##

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../college/lib/jquery/jquery.js"></script>
</head>
<body>
    <div data-name="zwz" data-age="18"></div>
    
    <hr/>
    
    <button id="getData">获取数据</button>
    <button id="setData">设置数据</button>
    
    <script type="application/javascript">
        $(function(){
            // 获取数据
            $("#getData").on("click",function () {
                    console.log($("div").data("name"));
                    console.log($("div").data("age"));
            })
            // 设置数据
            $("#setData").on("click",function(){
                $("div").data({name:'中国', age:'70'})
                alert("ok");
            })
        });
    </script>
</body>
</html>
```

## 收藏功能 ##

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <!-- BootStrap css -->
    <link href="lib/bootstrap/css/bootstrap.css" rel="stylesheet" />
</head>
<body>
    <button class="btn btn-success">
        <span class="infoSpan">收藏</span>
        <span class="icon glyphicon glyphicon-star-empty"></span>
    </button>
</body>
<script src="lib/jquery/jquery.js"></script>
<script src="lib/bootstrap/js/bootstrap.js"></script>
<script type="application/javascript">
     var btn = document.querySelector('button')
     // console.log(btn)
     var infoSpan =document.querySelector('.infoSpan');
     var iconSpan = document.querySelector('.icon');
     btn.onclick = function (){
         infoSpan.innerHTML = '已收藏'
         iconSpan.className = 'icon glyphicon glyphicon-star'
     };
</script>
</html>
```

## BootStrap 开发

### CSS 层叠样式表

选择器的使用 

复杂选择器不会使用

### BootStrap 响应式开发顺序

1. 先做好大屏幕、中等屏幕（ PC 端）的，后面再考虑小屏幕、超小屏幕（移动端）
2. 做一个模块时，同时把 PC端（台式电脑、笔记本电脑）、移动端（平板、手机）都做好

### BootStrap 响应式开发的思路

这里以先 PC 端做好，然后考虑小屏幕和超小屏幕开发顺序为例。

当遇到图片大小做响应式时，通常将图片 img 的 width 属性设置为 100% ，或者准备两种不同尺寸的图片，适配不同的设备，再或若该图片在移动端不想显示，可以设置 class 属性为 `hidden-xs`  ，还可以利用媒体查询，在不同 width 下，调整字体大小和布局方式(例如：改垂直布局为水平布局)

### 列表 ul 和 li 的使用

通过设置 li 的 css 样式 `float: left|right`，让其浮动成水平布局

应用场景：制作图片新闻图片

### 浮动问题





