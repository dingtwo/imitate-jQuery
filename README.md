# 原生仿jQuery的方法
### 版本兼容的一些问题
#### addEventListener和attachEvent
给元素简单的添加事件,通过

	obj.onclick = function () {};

这个方法是可以兼容主流浏览器的,但是如果同时添加多个事件的时候之前的事件会被后添加的事件覆盖.在**现代浏览器中**,我们可以通过`addEventListener()`这个方法来给元素添加事件监听,实现事件绑定的效果

	/**
	 * event: 要绑定的事件的字符串
	 * function: 事件触发时对应的函数
	 * useCapturn: (可选)是否事件捕获, bool值
	 */
	obj.addEventListener(event, function, useCapture)

关于事件捕获, 看[这里](#事件捕获与事件冒泡).

而对于陈旧的**IE**来说,有一个专属的`attachEvent()`方法来实现同样的效果

	/**
	 * event: 要绑定的事件的字符串, 注意要加on, 如"onclick"
	 * function: 事件触发时对应的函数
	 */
	obj.attachEvent(event, function)

这个方法是IE浏览器独有的, 所以我们在封装的时候根据两种情况分别处理:

	/**
 	 * 添加事件
 	 * @param {eventName: 事件名}	
 	 * @param {obj: 要添加事件的对象}
 	 * @param {事件的回调}
	 */
	function addEvent(eventName, obj, fn) {
		if (obj.addEventListener) {
			obj.addEventListener(eventName, fn, false);
		}else {
			obj.attachEvent("on"+eventName, fn);
		}
	}

#### 事件捕获与事件冒泡
<span id="jump"></span>
假如有如下的页面结构, 即绿色元素嵌套了红色元素, 如果给两个元素都绑定点击事件, 那么当用户点击红色元素时, 两个元素的事件应该以什么顺序执行, 而且点击红色时如何阻止绿色的事件.

![div嵌套](http://7xr09w.com1.z0.glb.clouddn.com/capture.png)

最早网景和微软采取的是两套策略:

* 网景主张绿色元素的事件先触发, 称为捕获型(capturing). 即点击红色元素时, 父级视图(绿色)会先响应事件, 红色元素的事件最后被触发.
* 微软主张红色元素的事件先触发, 称为冒泡型(bubbling). 即点击红色元素时, 红色元素的事件会先被触发, 最后触发父级视图的事件.
W3C采取了折衷的办法, 任何发生在w3c事件模型中的事件，首是进入捕获阶段，到达目标元素，再进入冒泡阶段.

我们可以通过addEventListener方法来确定是在捕获时触发事件, 还是在冒泡时绑定函数, 如果最后一个参数为true, 则在捕获阶段绑定事件, 如果为false, 在冒泡阶段绑定事件, 比如:

	smallDiv.addEventListener("click", function  () {
		alert("小");
	}, false);
	bigDiv.addEventListener("click", function () {
		alert("大");
	}, true); 

显示的顺序为大->小, 分析一下事件的响应过程

1. 当点击红色元素(smallDiv)时, 事件进入捕获阶段, 首先找到父级视图(bigDiv), 父级视图设置为true, 函数在捕获阶段绑定, 所以先执行alert("大"), 然后向上找, 找到自己, 为false, 所以没有函数执行.
2. 进入冒泡阶段, 首先从自身开始, 执行alert("小"), 然后向下找, 父级视图上没有在冒泡阶段绑定的事件, 所以没有函数继续执行

如果:

	smallDiv.addEventListener("click", function  () {
		alert("小");
	}, true);
		bigDiv.addEventListener("click", function () {
		alert("大");
	}, false);

显示顺序为相反, 小->大, 分析:

1. 点击红色时, 进入事件捕获阶段, 先从父视图上开始查找, 发现父级视图事件捕获设置为false, 是在冒泡阶段绑定的, 所以不执行, 开始向上找, 找到红色视图自己, true, 弹出"小", 绑定阶段结束
2. 冒泡阶段开始, 首先自身的事件没有在冒泡阶段绑定, 所以向下找, 发现父级视图的事件是在这个阶段绑定的, 所以弹出"大".











