## watch和computed区别

 ### 相似之处

- 计算机属性和监听属性都可以依赖于响应式数据
- 都可以监听数据的变化，并作出相应的处理



###不同之处

1. 计算机属性

- 基于它们的依赖进行缓存的，只有依赖发生变化，计算属性才会重新计算，可以避免不必要的重复计算
- 适用于派生出一些新数据，比如对数据的过滤、排序、格式化等
- 可以像普通属性一样使用，不需要模板中调用方法
- 函数必须要有 return 返回值



1. 监听属性

- 允许在数据变化时执行异步操作或复杂的逻辑
- 适用于对数据变化做出响应，比如数据变化时发送请求、处理副作用等
- 可以监听一个或多个数据变化，并在数据变化时执行相应操作