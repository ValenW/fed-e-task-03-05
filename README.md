# fed-e-task-03-05

## 简答题

### Vue 3.0 性能提升主要是通过哪几方面体现的？

1. 响应式的实现优化
   - 从DefineProperty升级到Proxy对象, 性能提升
2. 编译优化
   - 提升静态组件, 现在diff不会再对比编译非根节点的静态节点
   - patch flag标记动态节点会发生变动的类别, 如prop, text等, 在diff中更有针对性地对比和编译动态节点
   - 缓存事件处理函数, 在实际运行时再调用绑定函数, 绑定函数的变动不会导致组件更新
3. 源码体积优化
   - 移除了不常用api, 如`inline-template`, `filter`
   - 引入tree-shaking, 编译时去掉未被引用的模块

### Vue 3.0 所采用的 Composition Api 与 Vue 2.x使用的Options Api 有什么区别？

options API将描述组件的选项按数据性质分类放置到不同对象中, 如`data, prop, methods`, 这样在开发一个复杂组件时, 不同业务逻辑的代码被分开放置到不同选项中, 不好进行封装和逻辑复用

Composition API允许开发者通过逻辑功能来组织代码, 将数据和代码组织为处理特定功能的函数, 这样就能更方便地进行封装和逻辑复用

另外Options API依赖于this上下文暴露property, 这对类型推导并不友好. 而Composition API使用的是普通变量和函数, 天然对类型推断友好, 更方便集成typescript.

### Proxy 相对于 Object.defineProperty 有哪些优点？

1. 性能更好, Proxy直接监听对象而非属性, Object.defineProperty需要遍历属性进行绑定
2. 支持监听动态添加的属性, 删除属性操作, 以及数组的索引访问和length变化
3. Proxy支持更多拦截方法, 如`apply, ownKeys, has`等
4. Proxy返回新对象, 可以只操作新对象, 而defineProperty只能遍历对象属性修改

### Vue 3.0 在编译方面有哪些优化？

- 添加`fragments`, 允许根节点下多个平行子节点
- 提升静态组件, 现在diff不会再对比和编译非根节点的静态节点
- patch flag标记动态节点会发生变动的类别, 如prop, text等, 在diff中更有针对性地对比和编译动态节点
- 缓存事件处理函数, 在实际运行时再调用绑定函数, 绑定函数的变动不会导致组件更新

### Vue.js 3.0 响应式系统的实现原理？

核心原理和2一样, 使用发布订阅模式, 总体分为3个阶段:

#### 组件初始化

1. `setupComponent`创建组件, 使用compositionAPI处理options得到一个响应式Observer对象
2. `setupRenderEffect`包装真正渲染方法, 构建render effect
3. 将当前render effect赋值activeEffect, 并立即执行effect

#### 依赖收集

0. 将render effect赋值activeEffect并执行, 触发render函数
1. render访问data属性, 触发get
2. get方法中触发track进行依赖收集
3. track方法通过当前proxy对象和属性名key, 从targetMap中得到目标对象对应属性的dep Set
4. 将activeEffect加入dep中

#### 派发更新

1. 执行`target[key] = value`改变属性时, 会调用proxy中的set, 触发trigger
2. trigger函数通过`proxy`对象和`key`属性名从targetMap找到deps
3. 遍历deps, 区分computed回调和renderEffect回调, 分开调用对应的回调,  触发对应的视图渲染和计算属性更新

