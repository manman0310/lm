---
layout: blog_default
title:  "经常使用的css样式"
---

# 经常使用的css样式

在此记录总是忘了的css样式

### 1. 去除th或td之间的空隙

``` css
<table style="border-collapse:collapse;"></table>
```

### 2. 设置input的placeholder的字体样式

``` css
input::-webkit-input-placeholder {    /* Chrome/Opera/Safari */
    color: red;}input::-moz-placeholder { /* Firefox 19+ */
    color: red;}input:-ms-input-placeholder { /* IE 10+ */
    color: red;}input:-moz-placeholder { /* Firefox 18- */
    color: red;
}
```

### 3. 隐藏滚动条或更改滚动条样式

``` css
*//*定义滚动条宽高及背景，宽高分别对应横竖滚动条的尺寸*/
::-webkit-scrollbar {
    width: 10px; /*对垂直流动条有效*/
    height: 10px; /*对水平流动条有效*/
}/*定义滚动条的轨道颜色、内阴影及圆角*/
::-webkit-scrollbar-track{
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,.3);
    background-color: rosybrown;
    border-radius: 3px;
}/*定义滑块颜色、内阴影及圆角*/
::-webkit-scrollbar-thumb{
    border-radius: 7px;
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,.3);
    background-color: #E8E8E8;
}/*定义两端按钮的样式*/
::-webkit-scrollbar-button {
    background-color:#fbcdf0;
}/*定义右下角汇合处的样式*/
::-webkit-scrollbar-corner {
    background:khaki;
}
```

### 4. 文字超出隐藏并显示省略号

**单行（要有宽度）**

``` css
width:200px;
white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
```

**多行**

``` css
word-break: break-all;
display: -webkit-box;
-webkit-line-clamp: 2;
-webkit-box-orient: vertical;
overflow: hidden;
```

### 5. 绝对定位元素居中（水平和垂直方向）

``` css
width: 200px;
height: 200px;
position: absolute;
left: 50%;
top: 50%;
transform: translate(-50%,-50%);
background-color: green;
```
