#### 一、JS的运行机制

js为**单线程**机制，即一个任务结束才会进行下一个任务，即**先进先出**的数据结构。

**同步：**在主线程上排队执行任务，前一个任务技术后，才会（方可）进行下一个任务。如果函数A返回，调用者就可以得到预期效果，那么函数就是同步的

**异步：**不进入主线程，进入**任务队列**（task queue）。只有任务队列通知了主进程才会执行异步任务。如果函数A返回值，调用者需要将来通过其他手段得到结果，则认为这个函数是异步的。

**Event Loop（事件循环）:**一个回调函数队列，当异步函数执行时，回调函数就会被压入这个队列。

宏任务(macro-task)：整体代码script、setTimeOut、setInterval

微任务(mincro-task)：promise.then、promise.nextTick(node)

**JS的运行机制 == Event Loop：**

1、首先判断JS是同步还是异步，同步就进入主线程，异步则进入异步队列等待。

2、异步任务在异步队列中注册函数，当满足触发条件后，会被推入主进程

3、同步任务进入主线程后一直执行，直到主线程空闲时，才会去异步队列中查看是否有可执行的异步任务，如果有就会被推入主进程。

**JS中的异步操作：**

1、setTimeout

2、setInterval

3、ajax

4、promise

5、I\O

**Event Loop事件循环图：**

![img](https://upload-images.jianshu.io/upload_images/4820992-82913323252fde95.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

具体详解：<https://www.jianshu.com/p/e06e86ef2595>

#### 二、JS的数据类型

- 简单数据类型

  string, number, boolean

- 引用类型

  object, function

- undefined    ` undefined === undefinded  // true`

- null                `null === null   // true`

`typeof`**操作符**

返回值：字符串-类型

```js
typeof 1	// 'number'
typeof 'abc'	// 'string'
typeof false	// 'boolean'
typeof object	// 'function'
typeof {a:1}	// 'object'
typeof Function 	// 'function'

typeof undefined 	// 'undefined'
typeof null		// 'object'
```

`isNaN`是window下的一个方法，平时写可以省略windows

作用：用来判断某个数据是否是NaN

```js
typeof isNaN	// 'function'
NaN === NaN	// false
typeof NaN	// 'number'
```

#### 三、原型与原型链

**原型：**在js中，每定义一个函数类型数据（普通函数、类）时，都会自带一个`prototype`属性，这个属性指向函数的原型对象。并且所有函数的**默认**原型都是Object实例。原型对象就相当于一个公共的区域，所有同一类的实例都可以访问这个原型对象，我们可以将对象的共有内容，统一设置到原型对象中。

**原型链：**对象与对象间的关系，对象与继承之间的关系，在js中是通过prototype对象指向父类对象，直到指向Object位置，这项形成的一个原型指向的链条，被称之为原型链。

1、`_proto_`  和`contructor`

`_proto_`: 每一个引用类型（普通对象、实例、prototype...）都会带一个属性`_proto_`,其属性值代表当前实例所属类的原型（prototype)。

`contructor`：是原型对象中的一个属性，指向函数对象。

`实例对象._proto_ === 构造函数.ptototype`

**构造函数.prototype.contructor === 构造函数**

![img](https://upload-images.jianshu.io/upload_images/3174701-9a3de0b501161c07?imageMogr2/auto-orient/strip|imageView2/2/w/530/format/webp)

![img](https://upload-images.jianshu.io/upload_images/3174701-18a76d28c0a9ea1b?imageMogr2/auto-orient/strip|imageView2/2/w/521/format/webp)

知识扩充

`instanceof`：用来判断一个实例是否属于某一个构造函数。

- 常规用法

```js
function GrandParent()
function Parent() {}
Parent.prototype = new GrandParent()
let child = new Parent()

child instanceof GrandParent  // true
child instanceof Parent  // true
```

- 复杂用法

```js
Object instanceof Function	// true
Function instanceof Object	// true
Object instanceof Object	// true
Function instanceof Function	// true
String instanceof Stirng	// false

String instanceof Object	// true
Number instanceof Object	// true
Boolean instanceof Object	// true
undefinded instanceof Object	// false
null instanceof Object	// false


```

#### 四、闭包

**定义：**有权访问另一个，函数作用域的，变量的，函数。

**闭包的使用场景：**

1、setTimeout，原生的setTimeout传递的第一个函数不能带参数，通过闭包可以实现传参效果

```js
function f1(a) {
    function f2() {
        console.log(a);
    }
    return f2;
}
let fun = f1(1)
setTimeout(fun,1000) // 1秒后打印出1
```

2、回调，定义行为，然后把它关联到某个用户事件上（点击或者按键）。代码通常会作为一个回调（事件触发时调用的函数）绑定到事件。

```html
// 当点击数字时，字体会变成相应大小
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
</head>
<body>
    <a href="#" id="size-12">12</a>
    <a href="#" id="size-20">20</a>
    <a href="#" id="size-30">30</a>

    <script type="text/javascript">
        function changeSize(size){
            return function(){
                document.body.style.fontSize = size + 'px';
            };
        }

        var size12 = changeSize(12);
        var size14 = changeSize(20);
        var size16 = changeSize(30);

        document.getElementById('size-12').onclick = size12;
        document.getElementById('size-20').onclick = size14;
        document.getElementById('size-30').onclick = size16;

    </script>
</body>
</html>

```

3、函数防抖，防止多次点击时。

```js
// 在时间被触发n秒后再执行回调，如果再者n秒内又被触发，则重新计时。
/*
* fn [function] 需要防抖的函数
* delay [number] 毫秒，防抖期限值
*/
function debounce(fn,delay) {
    let timer = null
    return function() {
        if(timer) {
            clearTimeout(timer)
            timer = setTimout(fn,delay)
        }else {
            timer = setTimeOut(fn,delay)
        }
    }
}
```

#### 五、函数防抖（debounce）

**理解：**

指触发事件后n秒内函数只执行一次，如果n秒内又触发了事件，则会重新计算函数时间。即：**当一个动作连续触发，只执行最后一次**

**应用场景：**连续的时间，只需触发一次回调的场景

- 搜索框搜索输入，只需用户最后一次输入完，再发送请求
- 手机号、邮箱..验证输入检测
- 窗口大小Resize,只需窗口调整完成后，计算窗口大小。防止重复渲染。

**简单实现：**

```js
const debounce = (fn,wait) => {
    let timer;
    return () => {
        clearTimeout(timer);
        timer = setTimeout(fn,wait);
    };
};
```

#### 五、函数节流(throttle)

**理解：**

限制一个函数在一定时间内只执行一次。

**应用场景：**间隔一段时间执行一次回调

- 滚动加载，加载更多或滚到底部监听
- 谷歌搜索框、搜索联想功能
- 高频点击提交、表单重复提交

**简单实现：**

```js
// setTimeout方式
const throttle = (fn,wait) => {
    let timer;
    return () => {
        if(timer) return;
        timer = setTimeout(() => {
            fn();
            timer = null;
        },wait);
    };
}
// 时间戳比较方式
const throttle = (fn,wait) => {
    let last = 0;
    return () => {
        const current_time = +new Date();
        if(current_time - last > wait) {
            fn.apply(this.arguments);
            last = +new Date();
        }
    };
};
```

**函数防抖 diff 函数节流**

相同点：

- 都可以通过setTimeout实现。
- 目的都是，降低回调执行频率，节省计算资源。

不同点：

- 函数防抖：在一段连续操作后，处理回调，利用clearTimeout 和setTimeout实现。其关注**一定时间连续触发**
- 函数接口：在一段连续操作中，每一段时间只执行一次，频率较高的事件中使用来提交性能。其关注**一段时间只执行一次**

#### 六、http和https的区别

![img](https://pics1.baidu.com/feed/3ac79f3df8dcd10089c3c49abb52f914b8122f0f.jpeg?token=0496ea7ba2b881d6df9e2e3dd8efeec7&s=28B6C812C572CE215CE39946020060F3)

​	网站的URL分为两部分：通信协议+域名

​	通信协议：http、https

- http

  http使用的是一种明文数据传输的网络协议。

  **弊端：**会被第三者截获未加密的传输数据，导致信息泄露

- https

  https在数据传输前会对数据进行加密，及时被拦截也是加密后的数据。

- 区别

  1、数据安全方面，https比http更安全

  2、在chrome浏览器上，http地址的链接会被标注为红色不安全链接。但https就是绿色的，用户体验更好一些

  3、百度和谷歌两大搜索引擎也表示，https的网站搜索排名要高于http.