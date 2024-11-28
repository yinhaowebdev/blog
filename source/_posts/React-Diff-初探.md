title: React Diff 初探
author: Billy Yin
tags: []
categories: []
date: 2019-06-08 20:07:00
---
### 传统diff

[A Survey on Tree Edit Distance and Related Problems](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.100.2577&rep=rep1&type=pdf)

>  传统diff算法的复杂度为 O(n3)

### React Diff策略

从根节点开始比较

比较是递归的

**新旧节点元素/组件类型不一样， 则完全移除对应老节点和其子树上的节点并新建和插入节点树**

对比同一类型的元素时只更新改变的属性

针对子元素比较时，默认按照顺序，一一比较

也可以加key，建立新旧节点树的某一子元素集中元素的对应关系，节省一定的开销

React diff 的复杂度为O(n)

### 源码片段分析

``` js
// ReactMultiChild.js
_updateChildren: function(
      nextNestedChildrenElements,
      transaction,
      context,
    ) {
      var prevChildren = this._renderedChildren;
      var removedNodes = {};
      var mountImages = [];
      var nextChildren = this._reconcilerUpdateChildren(
        prevChildren,
        nextNestedChildrenElements,
        mountImages,
        removedNodes,
        transaction,
        context,
      );
      if (!nextChildren && !prevChildren) {
        return;
      }
      var updates = null;
      var name;
      // `nextIndex` will increment for each child in `nextChildren`, but
      // `lastIndex` will be the last index visited in `prevChildren`.
      var nextIndex = 0;
      var lastIndex = 0;
      // `nextMountIndex` will increment for each newly mounted child.
      var nextMountIndex = 0;
      var lastPlacedNode = null;
      for (name in nextChildren) {
        if (!nextChildren.hasOwnProperty(name)) {
          continue;
        }
        var prevChild = prevChildren && prevChildren[name];
        var nextChild = nextChildren[name];
        if (prevChild === nextChild) {
          updates = enqueue(
            updates,
            this.moveChild(prevChild, lastPlacedNode, nextIndex, lastIndex),
          );
          lastIndex = Math.max(prevChild._mountIndex, lastIndex);
          prevChild._mountIndex = nextIndex;
        } else {
          if (prevChild) {
            // Update `lastIndex` before `_mountIndex` gets unset by unmounting.
            lastIndex = Math.max(prevChild._mountIndex, lastIndex);
            // The `removedNodes` loop below will actually remove the child.
          }
          // The child must be instantiated before it's mounted.
          updates = enqueue(
            updates,
            this._mountChildAtIndex(
              nextChild,
              mountImages[nextMountIndex],
              lastPlacedNode,
              nextIndex,
              transaction,
              context,
            ),
          );
          nextMountIndex++;
        }
        nextIndex++;
        lastPlacedNode = ReactReconciler.getHostNode(nextChild);
      }
      // Remove children that are no longer present.
      for (name in removedNodes) {
        if (removedNodes.hasOwnProperty(name)) {
          updates = enqueue(
            updates,
            this._unmountChild(prevChildren[name], removedNodes[name]),
          );
        }
      }
      if (updates) {
        processQueue(this, updates);
      }
      this._renderedChildren = nextChildren;

      if (__DEV__) {
        setChildrenForInstrumentation.call(this, nextChildren);
      }
    },
```
1. 遍历nextChildren中的元素
              1.1 if prevChildren[name]=nextChildren[name] ( 有相同的element ) ，向updates队列中push一个移动element的任务, 更新lastIndex
              1.2 else 插入元素
2. 遍历prevChildren中的组件， 对于nextChildren中没有的元素，向updates队列中push一个删除element的任务
3. 将updates队列传入processQueue执行

其中移动element的逻辑
```
// ReactMultiChild.js
    moveChild: function(child, afterNode, toIndex, lastIndex) {
      // If the index of `child` is less than `lastIndex`, then it needs to
      // be moved. Otherwise, we do not need to move it because a child will be
      // inserted or moved before `child`.
      if (child._mountIndex < lastIndex) {
        return makeMove(child, afterNode, toIndex);
      }
    },
```
只有在被比较的prevChild的的index小于lastIndex时才会真正的移动节点

上述 processQueue最后会执行到
```js
// DOMChildrenOperations.js
  processUpdates: function(parentNode, updates) {
    if (__DEV__) {
      var parentNodeDebugID = ReactDOMComponentTree.getInstanceFromNode(
        parentNode,
      )._debugID;
    }

    for (var k = 0; k < updates.length; k++) {
      var update = updates[k];
      switch (update.type) {
        case 'INSERT_MARKUP':
          insertLazyTreeChildAt(
            parentNode,
            update.content,
            getNodeAfter(parentNode, update.afterNode),
          );
          if (__DEV__) {
            ReactInstrumentation.debugTool.onHostOperation({
              instanceID: parentNodeDebugID,
              type: 'insert child',
              payload: {
                toIndex: update.toIndex,
                content: update.content.toString(),
              },
            });
          }
          break;
        case 'MOVE_EXISTING':
          moveChild(
            parentNode,
            update.fromNode,
            getNodeAfter(parentNode, update.afterNode),
          );
          if (__DEV__) {
            ReactInstrumentation.debugTool.onHostOperation({
              instanceID: parentNodeDebugID,
              type: 'move child',
              payload: {fromIndex: update.fromIndex, toIndex: update.toIndex},
            });
          }
          break;
        case 'SET_MARKUP':
          setInnerHTML(parentNode, update.content);
          if (__DEV__) {
            ReactInstrumentation.debugTool.onHostOperation({
              instanceID: parentNodeDebugID,
              type: 'replace children',
              payload: update.content.toString(),
            });
          }
          break;
        case 'TEXT_CONTENT':
          setTextContent(parentNode, update.content);
          if (__DEV__) {
            ReactInstrumentation.debugTool.onHostOperation({
              instanceID: parentNodeDebugID,
              type: 'replace text',
              payload: update.content.toString(),
            });
          }
          break;
        case 'REMOVE_NODE':
          removeChild(parentNode, update.fromNode);
          if (__DEV__) {
            ReactInstrumentation.debugTool.onHostOperation({
              instanceID: parentNodeDebugID,
              type: 'remove child',
              payload: {fromIndex: update.fromIndex},
            });
          }
          break;
      }
    }
  },
```
上述diff中用到的主要是INSERT_MARKUP，MOVE_EXISTING，REMOVE_NODE三种
每一种case对应的方法，最终是直接操纵DOM的，如

```js
function removeChild(parentNode, childNode) {
  if (Array.isArray(childNode)) {
    var closingComment = childNode[1];
    childNode = childNode[0];
    removeDelimitedText(parentNode, childNode, closingComment);
    parentNode.removeChild(closingComment);
  }
  parentNode.removeChild(childNode);
}

```

### Tip

#### 1 尽量避免跨层次移动组件

原因：这种情况下react会完全移除原有DOM，然后创建新的DOM，浪费性能

#### 2 尽量避免父元素类型变动而子元素类型不变的操作

原因： 同上，原本内容的DOM树会被销毁和重建

#### 3 列表类型使用key，且key不为列表下标

原因： 不使用key时默认根据元素顺序一一比较，插入元素时会造成的不必要的销毁和新建，加key之后可以变销毁/新建 为 移动DOM操作

#### 4 使用shouldComponentUpdate() 或 React.PureComponent 来避免不必要的diff检查和render

### 备注

上述代码来自React 15 ，React 16之后的版本引入了Fiber，优化了diff，待有时间了再总结一下

###### 参考

[https://react.docschina.org/docs/reconciliation.html](https://react.docschina.org/docs/reconciliation.html)
[http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.100.2577&rep=rep1&type=pdf](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.100.2577&rep=rep1&type=pdf)
[https://calendar.perfplanet.com/2013/diff/](https://calendar.perfplanet.com/2013/diff/)

