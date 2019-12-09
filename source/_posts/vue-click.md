---
title: Vue click expends
tags: vue
---

>#### 自定义点击指令,包括:
>
>- 鼠标时间 – pc端
>- touch – 移动端

---



```javascript
Vue.directive('touch', {
  bind: function (el, binding, vNode) {
    // Make sure expression provided is a function
    if (typeof binding.value !== 'function') {
      const compName = vNode.context.name
      let warn = `[v-touch:] provided expression '${binding.expression}' is not a function, but has to be`
      if (compName) { warn += `Found in component '${compName}' ` }
      console.warn(warn)
      return
    }
    const start = e => {
      e.preventDefault()
      console.log(e.type)
    }
    const click = e => {
      e.preventDefault()
      console.log(e.type)
    }
    const end = e => {
      e.preventDefault()
      console.log(e.type)
      // binding.value(e)
      // binding.value.func(binding.value.param)
    }
    const cancel = e => {
      e.preventDefault()
      console.log(e.type)
    }
    // Add Event listeners
    el.addEventListener('touchstart', start)
    el.addEventListener('mousedown', start)
    el.addEventListener('click', click)
    el.addEventListener('touchend', end)
    el.addEventListener('mouseup', end)
    el.addEventListener('touchcancel', cancel)
  }
})
```

