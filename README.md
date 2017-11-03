### HTML CSS | 清除浮动的几种方式

> 第一次遇到「清除浮动有几种方式」这个问题是在生平的第一次面试中。那次面试让自己意识到前端学习中的一个重要问题，那就是不注重知识的归纳总结。
>
> 我记得，当时除了回答上来一个「使用 `clear:both;` 」之外，还说了一个特别傻的答案──对元素应用 `position: relative;` 。因为我想当然的理由是：「应用 `position: absolute;` 的元素会浮动，脱离文档流；因此恢复其布局方式，便能清除浮动」。面试官反问一句「那不就回去了吗？」。这说明还是从本质上不理解**为什么要清除浮动**。
>
> 后来，在网上搜到一些总结，但要么是只有代码和例子，不讲原理；要么是只有文字性的描述，没有实例。我觉着这两种方法都不够好，所以就有了这篇文章。

#### 为什么要清除浮动？

要搞明白清除浮动的方法，首先要搞清为什么要清除浮动。通常情况下，应用浮动是想要让元素在水平方向上排列，但由于对元素应用浮动之后脱离了文档流，因此可能会打破其后面的布局。接下来就以下方布局为例，来说明清除浮动的几种方法。

根据文档流的原理，我们想让第一行水平排列三个 `div` 元素（为了明显，为三个 `div` 的父级容器加上了红色的边框），第二行只放一个 `div` 元素。也就是这样：

<img src="01-perfect.png" width="100%" alt="正确布局"/>

```html
<!DOCTYPE html>
<html>
<head>
	<title>Float</title>
	<style type="text/css">
		/* 全局文字、背景色 */
		body{
			color: #445252;
			font-family: sans-serif;
			font-size: 16px;
			background: #F7E4C5;
		}

		/* 第一行div 和 第二行div 的大小等 */
		.first div,
		.second{
			width: 100px;
			height: 100px;
			line-height: 100px;
			text-align: center;
			border-radius: 8px;
		}

		/* 第一行三个 div 的颜色及外边距 */
		.first div{
			margin: 20px;
			background: #F8937E;
			float: left;
		}

		/* 第一行三个 div 父级容器边框 */
		.first{
			border: 1px solid red;
		}

		/* 第二行 div 的颜色及外边框*/
		.second{
			margin: 40px;
			background: #E73A38;
		}
	</style>
</head>


<body>
	<div class="first">
		<div>First-A</div>
		<div>First-B</div>
		<div>First-C</div>
	</div>

	<div class="second">Second</div>
</body>
</html>
```

#### 问题的出现

之前是我们预想的效果。但当对第一行的三个 `div` 元素应用 `float: left;` 使其水平并列排列的时，真实效果如上图：

<img src="02-problem.png" width="100%" alt="排版错位"/>

其中有几个问题：

- 第一行三个 `div` 并没有将父容器撑开，父容器 `div.first` 高度为 0

- 由于对第一行的三个 `div` 应用了 `float: left;` 因此它们脱离了文档流，在纵深上向上移动了，因此没有应用浮动的 `div.second` 会紧接着没有应用浮动的元素排列──也就是顶着文档的开始排列，且在纵深关系上位于第一行三个 `div` 的底部（错位是因为 `div.second` 和第一行三个 `div` 元素的 `margin` 值不一样，否则就会被挡住）

- 这一条其实这不能算一个问题，可以看做一个特性：浮动元素（`.first > div`）不会影响未浮动元素中（`div.second`）的文字，而是保留在原来位置。开发者会利用这一特性绕排文字，就像这样（自己动手改变一下样式试试。当然，你可以查看我写的 <a href="text-surrounding.html" target="_blank_">text-surrounding.html</a> ）：

  ![文字绕排](03-text-surrounding.png)

#### 解决方法

根据上边的问题，我们首先从原理上来分析如何实现正确的效果。其实我们想要的效果就是，**`div.second` 元素不要从文档顶端开始排列，而是紧接着宽度为 100% 的 `div.first` 容器排列**。 

**1. 给浮动元素的父级元素设置高度**

要跟在 `div.first` 后面，首先 `div.first` 得在布局中占据空间。之前它高度为 0 ，就相当于没有占据空间。因此我们来手动给它设置一个，如 400px ：

```css
.first{
  	/* ----- 清除浮动方法之 设置高度 ----- */
  	height: 400px;
  	border: border: 1px solid red;
}
```

效果如下图：

<img src="04-1-set-height.png" width="100%" alt="给浮动元素的父级元素设置高度"/>

缺点：需在内容尺寸确定的情况下设置固定高度；无法实现容器高度被内容自动撑开。

> *P.S. 在测试这些方法时，记得将之前的方法注释掉。每个方法是单独的。*



**2. 在浮动元素后插入空元素，并清除其浮动**

为了解决上述方法的缺点，我们转换思路：让 `div.second` 紧跟在 `div.first` 后面排列，其实可以在两者之间插入一个没有内容的 block（块级）元素（因为块级元素默认宽度是 100% ），并不允许其两侧出现浮动元素。

在这个例子中，代码如下（为了不造成干扰，省去了一些重复部分）：

```html
<style type="text/css">
  	/* ----- 清除浮动方法之 空元素 ----- */
  	.clear{
   		clear: both;
    	/* 给空元素加一个边框，易于观察 */
    	border: 1px solid green;
 	}
</style>

<body>
	<div class="first">
		<div>First-A</div>
		<div>First-B</div>
		<div>First-C</div>
	</div>
	
	<!-- 插入空元素 -->
	<div class="clear"></div>
	<div class="second">Second</div>
</body>
```

效果如下：

<img src="04-2-blank-element.png" width="100%" alt=" 通过空元素"/>



**3. 🥇（推荐）给浮动元素的父元素添加伪元素，并对齐应用清除浮动**

和方法 2 同样的思路，不借助空元素，而是在浮动元素的父级元素上创建伪元素，并对其应用 `clear: both;` 。代码如下：

```html
<style type="text/css">
  	/* ----- 清除浮动方法之 伪元素 ----- */
    .first::after{
        content: '';
        display: block;
        width: 0;
        height: 0;
        clear: both;
      	/* 给伪元素加一个边框，易于观察 */
      	border: 1px solid brown;
    }
</style>

<body>
	<div class="first">
		<div>First-A</div>
		<div>First-B</div>
		<div>First-C</div>
	</div>
  
	<div class="second">Second</div>
</body>
```

效果如下（注意观察在红色边框的左下角，有一个特别小的点，那就是伪元素代码中的棕色边框）：

<img src="04-3-pseudo-element.png" alt="通过伪元素"/>

**4. 对浮动元素的父级元素应用 `overflow: hidden;` 或 `overflow: auto;`**

在「问题的出现」部分，我们提到了父级元素 `div.first` 的高度没有被内容撑开，为 0 。事实上，我们也可以理解成 `div.first` 的内部元素溢出来了。因此，利用 `overflow` 属性，将其设置为非 `visible` 值──也就是 `hidden` 或 `auto` 。这时，`div.first` 会重新计算其内部元素位置，从而获得确切高度。这样父容器也就包含了浮动元素高度。

关于该方法的原理及底层详细讲解，涉及到 CSS 规范中的定义。如果感兴趣，可以阅读文末参考资料 4《CS001: 清理浮动的几种方法以及对应规范说明》中的第 3 个方法。

效果如下：

<img src="04-4-overflow.png" width="100%" alt="通过 oveflow"/>



#### 总结

以上只是常见的四中方法；其实方法有很多，在参考资料 4《CS001: 清理浮动的几种方法以及对应规范说明》一文中有详细的总结。

此外，本文的例子非常有局限性，主要体现原理和思路。希望帮你理清了「清除浮动」这个概念。更多的还需要自己亲手操作，应用到实战项目中去。



本文实例的 Github 项目链接：



> 参考链接：
>
> 1. 简书：[@WPeach - 清除浮动的几种方式](http://www.jianshu.com/p/1ce623f47ac2)
> 2. 简书：[@镇屌要逆袭 - CSS清除浮动的几种方法](http://www.jianshu.com/p/b381204bc34c)
> 3. CSDN 博客：[清除浮动的几种方式总结](http://blog.csdn.net/junbo_song/article/details/52194247)
> 4. W3 Help ：[CS001: 清理浮动的几种方法以及对应规范说明](http://w3help.org/zh-cn/casestudies/001)