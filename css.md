#### 一、盒模型

标准盒模型：内容(content)+填充(padding)+边框(border)+边距(margin)

IE(怪异盒模型)：内容（content+padding+border）+边距

box-sizing:  content-box(标准盒模型)

​					  border-box(怪异盒模型)

​					  inherit(跟从父元素继承)

#### 二、flex布局

flex-direction: row(默认横向顺排)、row-reverse(横向反排)、column(纵向顺排)、column(纵向反排)

flex-wrap:  nowrap(默认不换行)、wrap(换行)、wrap(换行反排 )

flex-flow:  是flex-direction | flex-wrap 的简写模式，例如：flex-flow: row nowrap;

justify-content: flex-start（左对齐）

​							flex-end（右对齐）

​							center（居中对齐）

​							space-around（两端对齐）

​							space-between（四周围绕）

align-items:   flex-start（交叉轴上对齐）

​						flex-end（交叉轴下对齐）

​						center（交叉轴居中对齐）

​						baseline（项目的第一行文字基线对齐）

​						stretch（默认值-当项目未设定高度或高度为auto时默认填充整个容器）

align-content:	flex-start、flex-end、space-between、space-around、stetch（默认值-占满整个交叉轴）

项目的属性：

order: 数字越小越靠前

flex-grow: <number>定义占比,默认0

flex-shrink:<number>项目的缩小比例，默认1

flex-base:分配多余空间之前，项目占据的主轴空间，默认auto

flex: flex-grow | flex-shrink | flex-base 的缩写建议优先写这个

#### 三、盒子水平垂直居中

1、

```css
.box {
	position:absolute;
    top:0;
    left:0;
    right:0;
    bottom:0;
    margin:auto
}
```

2、

```css
.container{
    display:flex;
    justify-content:center;
    align-items:center;
}
```

3、

```
.box {
	position:absolute;
	top:50%;
	left:50%;
	transform:translate(-50%,-50%);
}
```

4、已知宽高

```css
.box{
    width:200px;
    height:200px;
    position:absolute;
    top:50%;
    left:50%;
    margin-top:-100px;
    margin-right:-100px;
}
```

#### 四、BFC的理解，日常遇到的BFC

定义：块儿级格式化上下文

目的：完全形成一个独立的空间，让空间内的子元素不会受到外界因素的影响

**BFC能够解决那些问题：**

1、因浮动造成的父元素塌陷问题

2、div浮动造成的遮盖问题(浮动的两栏布局)【overflow:hidden;触发bfc来解决遮挡问题】

3、垂直方向上下布局，margin塌陷问题【overflow:hidden;产生bfc来解决】

**如何使用BFC解决上述问题：**

1、float的值不为none；

2、position的值不是static或者relative;

3、display的值是inline-block、table-cell、flex、tab-caption、inline-flex;

4、overflow的是不是visible;

相关内容简述：<https://blog.csdn.net/sinat_36422236/article/details/88763187>

#### 五、清除浮动的方法

1、clear：both；（不推荐）

2、over：hidden; （不推荐）

3、使用after伪元素清除（推荐）

```css
.clearfix:after {
    content:'';
    height:0;
    display:block;
    clear:both;
    visiblity:hidden;
}
.clearfix {
    *zoom:1; // 兼容ie6-ie7,此代码在其他浏览器不识别
}
```

4、使用before、after双伪元素清除

```css
.clearfix:after,.clearfix:before {
	content:'';
	display:table;
}
.clearfix:after {
	clear:both;
}
.clearfix {
	*zoom:1; // 兼容ie6-ie7,此代码在其他浏览器不识别
}
```

**何为clear：both** -> 本质就是闭合浮动，父盒子将子盒子闭合起来，不让子盒子出去

#### 六、什么是文档流？什么是脱离文档流？如何脱离文档流？

文档流：浏览器默认的排版模式就是文档流，即块级元素垂直排版、行内元素和行内块元素水平排版的模式。

脱离文档流：使元素不参与标准文档流的排版模式（处于标准文档流之上）

脱离文档流的方法：

```css
1、使用float浮动
2、使用position: absolute | fixed;
```

更多css面试题：

<https://blog.csdn.net/u014697639/article/details/80311559>

<https://juejin.im/post/5cc59e41e51d456e62545b66>

#### 七、CSS3的新属性

<https://www.cnblogs.com/star91/p/5659134.html>

#### 八、position有哪些属性，分别相对于谁定位

- **position: static;**	默认布局

- **position: relative;**    相对定位

  相对于：本身定位、不脱离文档流、可以使用z-index属性

- **position: absolute;**   绝对定位

  相对于：最近一个有定位的父元素定位（向上找，若没有则相对于document）、脱离文档流、可以使用z-index属性

- **position: fixed;**   固定定位

  相对于：浏览器窗口定位、脱离文档流、可以使用z-index属性

- **position:sticky;**   粘贴定位（吸顶）

  相对于：用户滚动的位置

  它的行为就像 position:relative; 而当页面滚动超出目标区域时，它的表现就像 position:fixed;，它会固定在目标位置。

  **注意:** Internet Explorer, Edge 15 及更早 IE 版本不支持 sticky 定位。 Safari 需要使用 -webkit- prefix 

- **position:inherit;**  规定继承父元素定位方式

  相对本身定位、不脱离文档流、可以使用z-index属性  设置恢复默认值

#### **九、超出文本显示省略号**

- 一行超出省略...

```css
{
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    word-break: break-all;
}
```

- 两行超出省略...

```css
{
    display:-webkit-box;
    overflow: hidden;
    text-overflow: ellipsis;
    line-clap:2;	// 多行修改此数
    text-overflow:-o-ellipsis-lastline;
    -webkit-line-clap: 2;	// 多行修改此数
    -webkit-box-orient:vertical;
}
```

#### 十、css3实现多列布局

column-count：属性规定元素应该被分隔的列数

column-width：属性规定列的宽度

column-gap：属性规定列之间的间隔

column-span：属性规定元素应该横跨的列数

column-fill：属性规定如何填充列

columns：column-width | column-count 合写的属性

column-rule: 所有column-rule-*的简写

​	column-rule-width:规定列之间**规则**的宽度

​	column-rule-style:规定列之间的**规则**样式

​	column-rule-color:规定列之间的**规则**颜色



