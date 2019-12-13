---
title: Vue tips
tags: vue
categories:  前端
---

##  Vue tips

<!--less-->



#### 初始化 ‘data’

```javascript
Object.assign(this.$data, this.$options.data.call(this))
```

