

## React如何对比更新?

- 只对同级节点更新,如果DOM节点跨越层级了,则不会复用
- 不同类型的元素产出不同结构
- 可以通过key来复用原有的dom
    当key和type相同时,会复用
- 类型一致的节点才有继续diff的必要性

