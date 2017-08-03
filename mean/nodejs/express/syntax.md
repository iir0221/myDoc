# 在Express中使用layout

默认情况下，Express在views目录下查找views,在views/layout目录下查找layout

* 指定默认layout
```js
app.engine('handlebars',
	express_handlebars({
		defaultLayout:'main',
		helpers:{
			section:express_handlebars_sections()
		}
	})
);
```
* 渲染view:views/tours/oregon-coast,使用默认layout
```js
app.get('/tours/oregon-coast', function(req, res){
	res.render('tours/oregon-coast');
});
```
* 渲染view,不使用layout
```js
app.get('/tours/oregon-coast', function(req, res){
	res.render('tours/oregon-coast',{layout:null});
});
```
* 渲染view，使用指定layout: views/layouts/microsite.handlebars 
```js
app.get('/tours/oregon-coast', function(req, res){
	res.render('tours/oregon-coast',{layout:'microsite'});
});
```

# partials

# section
从微软的优秀模板引擎 Razor 中借鉴了段落(section)的概念。
如果所有的视图在你的布局中都正好放在一个单独的元素里,布局会正常运转,但是当你的视图本身需要添加到布局的不同部分时会发生什么?

一个常见的例子是,视图需要向 \<head> 元素中添加一些东 西,或是插入一段使用 jQuery 的 \<script>脚本(这意味着必须引入 jQuery,由于性能原 因,有时在布局中这是最后才做的事)。

Handlebars 和 express3-handlebars 都没有针对于此的内置方法。幸运的是,Handlebars 的 辅助方法让整件事情变得简单起来。当我们实例化 Handlebars 对象时,会添加一个叫作 section 的辅助方法: