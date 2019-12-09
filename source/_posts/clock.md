---
title: 计时器
tags: JS

---


```js
requestAnimationFrame(function fn() {
    // 60帧 每秒 (根据显示器刷新率而定)
    if(count == 60) {
        count = 0
        _this.now = _this.$moment().format('YYYY-MM-DD HH:mm:ss')
    }
    count ++
    // 渲染下一帧
    requestAnimationFrame(fn)
})
```

