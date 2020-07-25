



# Anchor Component

> antd 组件 Anchor , 蚂蚁那边是用的 a 链接跳转方式，会有 # ，本组件使用另外的跳转方式实现跳转。
>
> 不依赖任何第三方组件，使用简单



## 已实现版本

1. label + input 跳转，注入input
2. scrollIntoView 跳转方式，不注入input

## 已实现功能

- 支持定义高度
- 支持自定义锚点文案

- 容器支持是否溢出隐藏，（即是否允许在容器中鼠标进行滚动）

- 传入组件动态懒加载
- 支持点击事件，能得知点击加载了哪个组件
- 支持容器滚动事件 
  - 正在滚动
  - 滚动完成
- 可监听到所有懒加载组件均加载完成，并提供回调

#  Anchor Component - Properties




| 属性            | 说明                    | 类型    | 默认值 |
| --------------- | ----------------------- | ------- | ------ |
| scroll-view     | 组件wrapper是否溢出隐藏  | Boolean | true   |
| label-name | `必须`自定义导航列表名称，顺序排列，不可重复 | Array |  |
| lazy | 支持懒加载，三种参数类型详细说明见例子 | Boolean / String / Array | false |
| lazy-include | 预期加入，传入子组件对应的index | index : Array<number> | / |
| lazy-exclude | 预期加入，传入子组件对应的index | index : Array<number> | / |
|  |  |  |  |



# Anchor Component - Methods



| 方法        | 说明                                | 类型                 | 默认值                    |
| ----------- | ----------------------------------- | -------------------- | ------------------------- |
| onClick     | 点击labelName 时触发的事件          | function(node,index) | 子组件，子组件所在的index |
| onScrolling | 容器正在执行滚动事件                | function()           | /                         |
| onScrolled  | 容器已执行完滚动事件，节流延迟100ms | function()           | /                         |
| onAllDone   | 预期加入，传入子组件对应的index     | function()           | /                         |
|             |                                     |                      |                           |
|             |                                     |                      |                           |



# 注意：常见问题

- Vue会缓存组件，确保传入的组件不重复

  

## 下个版本预计实现

- lazy-include
- lazy-exclude

- 锚点（label）样式
- 锚点（label）可自定义吸附位置 top | right | bottom | left
- 性能优化



# 开发过程中的问题

本着好奇的心情想通过比较高级的写法来写一个组件，以下是我遇到的问题，有师傅点拨真的太好了



- 如何进行传入的组件侵入，并在上面运用 模板语法 `v-if` 
  - 询问师傅后，得知 可以用 render

> render 函数我不太熟悉，也没认认真真看完文档，就去问了师傅，师傅听到我的想法后，带我一起翻了下文档，然后看到了 `v-if` 的用法 直接就是 在 `render` 函数中以 `if` `else`  的方式呈现 ，(条件判断决定是否 `return createElement()`)  参考官方文档  [使用 JavaScript 代替模板功能](https://cn.vuejs.org/v2/guide/render-function.html#使用-JavaScript-代替模板功能)



- 在第一步成功后，如何唤醒未加载 (懒加载) 的子组件？
  1. 建立一个缓存，存储每次变化的VNode （开销会不会太大，浅拷贝应该不会）
  2. 去翻 render 函数文档，看看如何加 @click.once 事件
- 经历第二步的唤醒函数思考，如何将render函数中的函数重新调用一次，
  1. 这令我想到了 computed属性 ，但却具有响应式特性
  2. 改写 `_childComponentLazyLoadMarkInitFactory` 函数为 computed 属性
  3. computed属性非常的不能理解，为什么他能收到依赖的改动而响应，需要好好研究









# 开发总结

- 对生命周期的理解程度

在render 中能 调用到  data ，computed ，methods 等等属性

Vue的生命周期最新开始的是 实例属性，可去官网看看那些东西属于实例属性，

总结出来的生命周期流程：

beforeCreated > 实例属性 > created > render函数 > before mounted >  mounted  > beforeUpdate > update > beforeDestroy > destroyed

- prop 的自定义验证
- render函数的配置，render函数的提取美化

- 合理安排watch
- 合理借助中间变量
- 恰到好处的执行 nextTick
- `||` 表达式的重新认知
- `，` 表达式配合 computed属性的妙用
- 不爱写`if else` 的替换方案
- 更好的去判断 数据类型 

- 在合理的时机运用好生命周期的钩子函数
- 永远不要相信console，要相信debugger
- 要仔细看debugger所在的上下文，当前想看的变量都是些什么值，特别是在写render函数的时候，有时候闭包过多就要注意去调用栈里查看`closure` 里面都保存着什么