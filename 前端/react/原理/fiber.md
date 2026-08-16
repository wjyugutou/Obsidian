## fiber 是什么：
React 每次渲染维护一棵 fiber 树，每个组件对应一个 fiber 节点（一个持久的 JS 对象）。它不是临时数据——跨渲染存活。

### useState(0) 的 0 存在哪
fiber 节点上有个字段叫 memoizedState。第一次渲染，useState(0) 把 0 初始化进去；之后每次渲染，useState 都从 fiber 上读上次的值，而不是重新造。这就是"组件函数被反复执行但 state 不丢"的原因——函数只是"看图说话"，数据存在 fiber 这个"档案"里。

### 多个 useState 怎么区分
fiber.memoizedState 是一条单向链表：hook0 → hook1 → hook2…。每个 useState 按调用顺序挂到链表对应位置。

### 这就是 Hooks 规则（不能写进 if/循环）的根本原因：
条件一变化，这次渲染的 hook 调用顺序和链表对不上，值和 hook 就错位了——不是"React 规定"，是链表的索引机制决定的。这句是面试加分点。

### setState 后新值存哪：
不是立刻改 memoizedState。setState 把 update 塞进这个 hook 的 updateQueue，等下一次渲染的 reconcile 阶段才计算新值写回 memoizedState。——这也正是"setState 后读旧值"的最底层原因：不只是闭包，fiber 上的值此刻本来就没变。