---
title: ES6 Proxy 实现基本的数据绑定
date: 2017-08-4 13:43:24
tags:
- ES6
- JavaScript
---

> 最近看到ES6的Proxy，想到可以用他实现一下数据绑定

## 构造模板
我们可以构造这样一个模板
``` JavaScript
const app = new Von({
  el: '#app',
  data() {
    return{
      a: 1,
      b: 2
    }
  },
  template(){
    const sum = this.a + this.b
    return `<div>Hello ${sum}</div>`
  }
})
```
是不是感觉跟Vue很像，没错，接下来我们要做的就是将这个模板跟我们的视图进行绑定！

## Proxy 语法
先来看看 Proxy 的语法 :
 ``` JavaScript
let prox = new Proxy(target, handler);
// target: 目标对象，可以是任意类型的对象，比如数组，函数，甚至是另外一个代理对象。
// handler: 处理器对象，包含了一组代理方法，分别控制所生成代理对象的各种行为。
```

这样看可能还不太清楚，我们可以这样理解，在初始化时我们构造了 target 对象的一个代理 prox ，当 prox 中的数据被修改时，我们遭到了拦截 , 调用了 handler ，在 handler 中，我们可以对模板进行渲染，达到数据与视图绑定的目的。

接下来我们构造一个类 Von ( 取了博主名字一部分）

``` JavaScript
class Von{
  constructor(config){
    this.$el = config.el
    this.$data = config.data()
    this.$tpl = config.template.bind(this.$data)
    this.dom = document.querySelector(self.$el)
    const self = this
    // 用this.$binding作为代理
    this.$binding = new Proxy(this.$data,{
      set(target,prop,value){  // target: 拦截的目标对象，prop: 拦截的属性 , value: 具体修改的值
        target[prop] = value  // 代理触发拦截时，对目标属性进行修改
        return true
      }
    })
  }
}

```

## 模板渲染
然后我们需要对模板进行渲染，构造一个内部函数 _render()

``` JavaScript
_render(){
    //这里我们简单粗暴的直接对 DOM 进行了覆盖，当然比较优雅的做法是使用 VirtualDOM 进行渲染
    this.dom.innerHTML = this.$tpl()
}
```
然后运行完整的代码
> https://github.com/lihaof/DataBinding-Proxy-.git

我们尝试一下 ：


![未修改时](http://upload-images.jianshu.io/upload_images/5304624-3398b79fa2fbac83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![修改代理后视图自动改变，app 的数据也会随之改变](http://upload-images.jianshu.io/upload_images/5304624-4ee2ddbf5f1508cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样我们的一个基本的数据绑定就算实现了，也可以按自己的要求修改 template 进行不同的渲染

_博主目前还是前端小白一枚，如果对文章有什么意见，欢迎在评论区中指出~_
