



# Anchor Component

> antd 组件 Anchor 

注意：

- Vue会缓存组件，确保传入的组件不重复

  

#  Anchor Component - Properties


- [ x ]支持定义高度

\- 支持是否溢出隐藏

\- 动态






| 属性            | 说明                    | 类型    | 默认值 |
| --------------- | ----------------------- | ------- | ------ |
| scroll-view     | 组件wrapper是否溢出隐藏  | Boolean | true   |
| label-name | 自定义导航列表名称，顺序排列，不可重复 | Array |  |
| lazy | 支持懒加载，三种参数类型详细说明见例子 | Boolean / String / Array | false |
| lazy-include | 预期加入，传入子组件对应的index | index : Array<number> | / |
| lazy-exclude | 预期加入，传入子组件对应的index | index : Array<number> | / |
|  |  |  |  |



# Anchor Component - Methods









# 开发过程中的问题



- 如何进行传入的组件侵入，并在上面运用 模板语法 `v-if` 
  - 询问师傅后，得知 可以用 render

> render 函数我不太熟悉，也没认认真真看完文档，就去问了师傅，师傅听到我的想法后，带我一起翻了下文档，然后看到了 `v-if` 的用法 直接就是 在 `render` 函数中以 `if` `else`  的方式呈现 ，(条件判断决定是否 `return createElement()`)  参考官方文档  [使用 JavaScript 代替模板功能](https://cn.vuejs.org/v2/guide/render-function.html#使用-JavaScript-代替模板功能)



- 在第一步成功后，如何唤醒未加载 (懒加载) 的子组件？
  1. 建立一个缓存，存储每次变化的VNode （开销会不会太大，浅拷贝应该不会）
  2. 去翻 render 函数文档，看看如何加 @click.once 事件
- 经历第二步的唤醒函数思考，如何将render函数中的函数重新调用一次，
  1. 这令我想到了 computed属性 实质上也是一个函数，但却具有响应式依赖
  2. 改写 `_childComponentLazyLoadMarkInitFactory` 函数为 computed 属性