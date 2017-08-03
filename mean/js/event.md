# event

## onload

### window.onload VS &lt;body onload="/&gt;

用法：

```html
window.onload = myOnloadFunc 

<body onload="myOnloadFunc();">
```

二者没有什么区别，但是推荐用第一个，可以把js和html分离。

另外onload要等页面所有资源都加载完毕才会执行，

**jquery的$\(document\).ready\(\)**方法，在页面所有的DOM加载完毕后就会触发,无疑很大的加快了网页的速度.

参考:  [window.onload vs &lt;body onload=“”/&gt;](http://stackoverflow.com/questions/191157/window-onload-vs-body-onload)

## onclick

```html
<li>
    <!--onclick事件，作用于链接，函数返回false，则不会触发链接-->
    <a href="images/fireworks.jpg" title="A fireworks display"
        onclick="showPic(this); return false;">
        Fireworks
    </a>
</li>
```



