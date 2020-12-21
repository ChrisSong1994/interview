### React Diff 算法

**解决问题** ：diff 作为 Virtual DOM 的加速器，为 diff 会帮助我们计算出 Virtual DOM 中真正变化的部分，并只针对该部分进行原生 DOM 操作，而非重新渲染整个页面，从而保证了每次操作更新后页面的高效渲染

#### diff 算法

- **传统 diff 算法**：计算一棵树形结构转换成另一棵树形结构的最少操作，是一个复杂且值得研究的问题。传统 diff 算法 ① 通过循环递归对节点进行依次对比，效率低下，算法复杂度达到 O(n3)，其中 n 是树中节点的总数。（需要先所有的节点循环比较 O(N^2),然后找到差异后最小转换方式 O(N)）

- **React Diff 算法**：
- React 将 Virtual DOM 树转换成 actual DOM 树的最少操作的过程称为调和（reconciliation）。diff 算法便是调和的具体实现。React 通过制定大胆的策略，将 O(n3) 复杂度的问题转换成 O(n) 复杂度的问题。

#### diff 策略

下面介绍 React diff 算法的 3 个策略。

- 策略一：Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计。
- 策略二：拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构。
- 策略三：对于同一层级的一组子节点，它们可以通过唯一 id 进行区分。

基于以上策略，React 分别对 tree diff、component diff 以及 element diff 进行算法优化。

### diff 粒度

**tree diff**: DOM 节点跨层级的移动操作少到可以忽略不计，针对这一现象，React 通过 updateDepth 对 Virtual DOM 树进行层级控制，只会对相同层级的 DOM 节点进行比较，即同一个父节点下的所有子节点。当发现节点已经不存在时，则该节点及其子节点会被完全删除掉，不会用于进一步的比较.React 只会简单地考虑同层级节点的位置变换，而对于不同层级的节点，只有创建和删除操作

**component diff**:React 是基于组件构建应用的，对于组件间的比较所采取的策略也是非常简洁、高效的。

- 如果是同一类型的组件，按照原策略继续比较 Virtual DOM 树即可。
- 如果不是，则将该组件判断为 dirty component，从而替换整个组件下的所有子节点。
- 对于同一类型的组件，有可能其 Virtual DOM 没有任何变化，如果能够确切知道这点，那
  么就可以节省大量的 diff 运算时间。因此，React 允许用户通过 shouldComponentUpdate()
  来判断该组件是否需要进行 diff 算法分析

**element diff**:当节点处于同一层级时，diff 提供了 3 种节点操作，分别为 INSERT_MARKUP（插入）、MOVE_EXISTING（移动）和 REMOVE_NODE（删除）。

- INSERT_MARKUP：新的组件类型不在旧集合里，即全新的节点，需要对新节点执行插入操作。
- MOVE_EXISTING：旧集合中有新组件类型，且 element 是可更新的类型，generateComponent,Children 已调用 receiveComponent，这种情况下 prevChild=nextChild，就需要做移动操作，可以复用以前的 DOM 节点。
- REMOVE_NODE：旧组件类型，在新集合里也有，但对应的 element 不同则不能直接复用和更新，需要执行删除操作，或者旧组件不在新集合里的，也需要执行删除操作。

#### React Patch 方法

将 tree diff 计算出来的 DOM 差异队列更新到真实的 DOM 节点上，最终让浏览器能够渲染出更新的数据
