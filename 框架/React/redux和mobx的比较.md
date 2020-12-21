### redux  和 mobx 的比较

- 1.redux将数据保存在单一的store中，mobx将数据保存在分散的多个store中
- 2.redux使用plain object保存数据，需要手动处理变化后的操作；mobx用observable保存数据，数据变化后自动处理响应的操作
- 3.redux使用不可变状态，这意味着状态是只读的，不能直接去修改它，而是应该返回一个新的状态，同时使用纯函数；mobx中的状态是可变的，可以直接对其进行修改
- 4.mobx相对来说比较简单，在其中有很多的抽象，mobx更多的使用面向对象的编程思维；redux会比较复杂，因为其中的函数式编程思想掌握起来不是那么容易，同时需要借助一系列的中间件来处理异步和副作用
- 5.mobx中有更多的抽象和封装，调试会比较困难，同时结果也难以预测；而redux提供能够进行时间回溯的开发工具，同时其纯函数以及更少的抽象，让调试变得更加的容易
- 6.异步调用mobx 可以直接在action 里面写async/await 函数，redux 需要用react-thunk中间件实现异步action 操作


#### mobx-react和react-redux
使用Redux和React应用连接时，需要使用react-redux提供的Provider和connect：

- Provider：负责将Store注入React应用；
- connect：负责将store state注入容器组件，并选择特定状态作为容器组件props传递；

对于Mobx而言，同样需要两个步骤：

- Provider：使用mobx-react提供的Provider将所有stores注入应用；
- 使用inject将特定store注入某组件，store可以传递状态或action；然后使用observer保证组件能响应store中的可观察对象（observable）变更，即store更新，组件视图响应式更新。

#### 选择Mobx的原因
- 学习成本少：Mobx基础知识很简单，学习了半小时官方文档和示例代码就搭建了新项目实例；而Redux确较繁琐，流程较多，需要配置，创建store，编写reducer，action，如果涉及异步任务，还需要引入redux-thunk或redux-saga编写额外代码，Mobx流程相比就简单很多，并且不需要额外异步处理库；
- 面向对象编程：Mobx支持面向对象编程，我们可以使用@observable and @observer，以面向对象编程方式使得JavaScript对象具有响应式能力；而Redux最推荐遵循函数式编程，当然Mobx也支持函数式编程；
- 模版代码少：相对于Redux的各种模版代码，如，actionCreater，reducer，saga／thunk等，Mobx则不需要编写这类模板代码；

#### 不选择Mobx的可能原因
- 过于自由：Mobx提供的约定及模版代码很少，这导致开发代码编写很自由，如果不做一些约定，比较容易导致团队代码风格不统一，所以当团队成员较多时，确实需要添加一些约定；
- 可拓展，可维护性：也许你会担心Mobx能不能适应后期项目发展壮大呢？确实Mobx更适合用在中小型项目中，但这并不表示其不能支撑大型项目，关键在于大型项目通常需要特别注意可拓展性，可维护性，相比而言，规范的Redux更有优势，而Mobx更自由，需要我们自己制定一些规则来确保项目后期拓展，维护难易程度；



相较于redux，mobx中存在着一些限制：

- 它不会分析你的数据中是否存在着循环依赖
- 它不会预先假定你的数据结构是plain object，class或是任何其他的数据类型
因此，mobx不能保证用户提供的数据一定能 JSON序列化，或者能在有限的时间遍历完。所以它更应该被认为是一个数据流管理工具，能够让你以较小的代价构建自己的状态管理架构。能够快捷的再现有的项目中使用，而不需要进行大规模的重写。