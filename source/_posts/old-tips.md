---
title: H5 IOS适配小结
tags: JS
---

---

#### IOS 有几个常见的问题，踩了大坑所以在此记录下下

**坑**：

- 有些IOS收起键盘时会卡住
- 微信浏览器默认没绑定全局（document）`click`事件（深坑）
- 弹起键盘页面被顶上去的问题
- IOS端 Date.parse()异常的问题

#### 解决方案

> **IOS收起键盘时页面卡住了：**
>
> 可以做一个失去焦点的判断，然后回归页面本位

```javascript
$('input').blur(()=>{    
	window.scroll(0, 0);//让页面归位
})
```

> **微信浏览器默认没绑定全局（document）click事件**
>
> 这个问题绑定下事件就好了，可惜当时不知道，困扰了我很久，百思不得起解，深坑~~~
>
> 为啥用click？因为VUE默认没 tap 事件，【其实可用`touchstart.prevent` 代替】

```javascript
// IOS 微信浏览器开启点击事件
$("body>*").bind("click",function(){});
// other click
```

> **弹起键盘页面被顶上去的问题**
>
> 这个问题很常见了，贴一个常见的解决方案

```javascript
let oh = $(window).height()
$(window).resize(function () {
    if ($(window).height() < oh) {
        $('body').css('height', `${oh}px`)
        $('select').css('height', `${oh}px`)
        // other logic
    }
});
```

> #### IOS端 Date.parse()异常的问题
>
> 原因是 ‘2019-11-11’ 这种格式是不可以地 换成 ‘2019/11/11’即可
>
> ```
> '2019-11-11'.replace(/-/g, '/')
> ```

> 最后记一个建议：
>
> 在IOS端，click 事件通常会有奇怪的问题，可以的话用touch类的事件代替
>
> The End | 2019-02-28