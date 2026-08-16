## diff 时 React 靠什么判断两个节点能不能复用

### diff 复用条件（只有两个）：type + key 相同

- type（标签名/组件类型）相同 + key 相同 → 复用 fiber 节点和 DOM 实例，只更新 props/children
- type 不同 → 整个子树销毁重建

### key 具体起什么作用

在同层兄弟之间标识节点的"身份"。没有 key 时 React 按位置猜，有 key 时按身份找——它解决的是"排序/插入/删除时，谁是原来的谁"

## index 当 key 的坑

列表 [A,B,C]，头部插入 X → [X,A,B,C]，key 用 index：
旧: A(key=0)  B(key=1)  C(key=2)
新: X(key=0)  A(key=1)  B(key=2)  C(key=3)
React 的匹配逻辑（key 相同 = 身份相同，复用 DOM 和 state）：
- 新 key=0 的 X → 匹配旧的 key=0（A）→ 复用 A 的 DOM 和内部 state，把内容替换成 X
- 新 key=1 的 A → 匹配旧的 key=1（B）→ 复用 B 的 DOM 和 state，内容换成 A
- 新 key=2 的 B → 匹配旧的 key=2（C）→ 复用 C 的 DOM 和 state，内容换成 B
- 新 key=3 的 C → 没有旧节点匹配 → 新建
结果灾难：每个节点都保留了"别人"的 DOM 实例和内部 state。

最经典的 bug——如果每个元素里有个 input，你原来在 A 的框里打了字，头部插入后那些字跟着 A 的 DOM 跑到了新位置（内容换成 X 的那个框里）。受控 input 错乱、组件内部状态串味、动画状态错位，全来了。
而用稳定 id 当 key：React 识别出 X 是纯新增 → 只插一个节点，A/B/C 的 DOM 零改动。这就是 key 的全部意义。