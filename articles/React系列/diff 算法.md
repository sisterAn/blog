## 深入diff 算法

diff 作为 Virtual DOM 的加速器，其算法上的改进优化是React页面渲染的基础和性能保障，本节从源码入手，深入剖析diff算法。

React 中醉值得称道的莫过于Virtual DOM与diff的完美结合，尤其是其高效的diff算法，可以帮助我们在页面蔌渲染的时候，计算出Virtual DOM真正变化的部分，并只针对该部分进行的原生DOM操作，而不是渲染整个页面，从而保证了每次操作后，页面的高效渲染。

### 一. 传统的 diff 算法

计算一个树形结构转换成另一个树形结构的最少操作，是一个复杂且值得研究的问题，传统 diff 算法通过循环递归的方法对节点进行操作，算法复杂度 为O(n^3^)，其中n为树中节点的总数，这效率太低了，如果 React 只是单纯的引入 diff 算法，而没有任何的优化的话，其效率远远无法满足前端渲染所需要的性能。那么React 是如何实现一个高效、稳定的 diff 算法。

### 二. diff 源码解读

React 将 Virtual DOM 树转换为 actual DOM 树的最小操作的过程称为调和， diff 算法便是调和的结果，React 通过制定大胆的策略，将 O(n^3^)的时间复杂度转换成 O(n)。

#### 1. diff 策略

下面是 React diff 算法的 3 个策略：

- 策略一：Web UI 中 DOM 节点跨层级的移动操作特别少。可以忽略不计。
- 策略二：拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构。
- 策略三：对于同一层级的一组子节点，它们可以通过唯一 id 进行区分。

基于以上三个策略，React 分别对 tree diff、component diff 以及 element diff 进行算法优化。

#### 2. tree diff

对于策略一，React 对树的算法进行了简介明了的优化，即对树进行分层比较，两颗树只会对同一层级的节点进行比较。

既然 DOM 节点跨层级的移动，可以少到忽略不计，针对这种现象，React 通过 updateDepth 对 Virtual DOM 树进行层级控制，只对相同层级的DOM节点进行比较，即同一父节点下的所有子节点，当发现该节点已经不存在时，则该节点及其子节点会被完全删除掉，不会用于进一步的比较。这样只需要对树进行一次遍历，便能完成整个DOM树的比较。

```js
// updateChildren 源码
updateChildren: function (nextNestedChildrenElements, transaction, context) {
    updateDepth ++;
    var errorThrown = true;
    try {
        this._updateChildren(nextNestedChildrenElements, transaction, context);
        errorThrown = false;
    } finally {
        updateDepth --;
        if (!updateDepth) {
            if (errorThrown) {
                clearQueue();
            } else {
                processQueue();
            }
        }
    }
}
```

那么就会有这样的问题：

**如果出现了 DOM 节点跨层级的移动操作，diff 会有怎样的表现喃？**

我们举个例子看一下：

如下图2-1，A节点（包括其子节点）整个需要跨层级移动到D节点下，React会如何操作喃？

![diff](/Users/a123/Desktop/Study/JS/img/diff.png)

**图2-1 DOM层级变换**

由于 React 只会简单的考虑同层级节点的位置变换，对于不同层级的节点，只有创建和删除操作。当根节点R发现子节点中A消失了，就会直接销毁A；当D节点发现多了一个子节点A，就会创建新的A子节点（包括其子节点）。执行的操作为：

create A —> create B —> create C —> delete A

所以。当出现节点跨级移动时，并不会像想象中的那样执行移动操作，而是以 A 为根节点的整个树被整个重新创建，这是影响 React 性能的操作，所以 **官方建议不要进行 DOM 节点跨层级的操作**。

> 在开发组件中，保持稳定的 DOM 结构有助于性能的提升。例如，可以通过CSS隐藏或显示节点，而不是真正的移除或添加 DOM 节点。

#### 3. component diff

React 是基于组件构建应用的，对于组件间的比较所采取的策略也是非常简洁、高效的。

- 如果是同一类型的组件，按照原策略继续比较 Virtual DOM 树即可
- 如果不是，则将该组件判断为 dirty component，从而替换整个组件下的所有子节点
- 对于同一类型下的组件，有可能其 Virtual DOM 没有任何变化，如果能确切知道这一点，那么就可以节省大量的 diff 算法时间。因此， React 允许用户通过 `shouldComponentUpdate()`来判断该组件是否需要大量 diff 算法分析。

![component_diff](/Users/a123/Desktop/Study/JS/img/component_diff.png)

**图3-1 component diff**

如上图3-1，当 D 组件变成 G 时，即使这两个组件结构相似，但一旦 React 判断D和G是两个不同类型的组件时，就不会再比较这两个组件的结构，直接进行删除组件D， 重新创建组件G及其子组件。虽然这两个组件是不同类型单结构类似，diff 算法会影响性能，正如 React 官方博客所言：

**不同类型的组件很少存在相似 DOM 树的情况，因此，这种极端因素很难在实际开发过程中造成重大影响。**

#### 4. element diff

当节点处于同一层级时，diff 提供三种节点操作：

- INSERT_MARKUP（插入）：如果新的组件类型不在旧集合里，即全新的节点，需要对新节点执行插入操作。
- MOVE_EXISTING （移动）：旧集合中有新组件类型，且 element 是可更新的类型，generatorComponentChildren 已调用 receiveComponent，这种情况下 prevChild=nextChild，就需要做移动操作，可以复用以前的 DOM 节点。
- REMOVE_NODE  （删除）：旧组件类型，在新集合里也有，但对应的 elememt 不同则不能直接复用和更新，需要执行删除操作，或者旧组件不在新集合里的，也需要执行删除操作。

```js
// INSERT_MARKUP
function makeInsertMarkup(markup, afterNode, toIndex) {
    return {
        type: ReactMultiChildUpdateTypes.INSERT_MARKUP,
        content: markup,
        fromIndex: null,
        fromNode: null,
        toIndex: toIndex,
        afterNode: afterNode
    }
}
// MOVE_EXISTING
function makeMove(child, afterNode, toIndex) {
    return {
        type: ReactMultiChildUpdateTypes.MOVE_EXISTING,
        content: null,
        fromIndex: child._mountIndex,
        fromNode: ReactReconciler.getNativeNode(child),
        toIndex: toIndex,
        afterNode: afterNode
    }
}
// REMOVE_NODE
function makeRemove(child, node) {
    return {
        type: ReactMultiChildUpdateTypes.REMOVE_NODE,
        content: null,
        fromIndex: child._mountIndex,
        fromNode: node,
        toIndex: null,
        afterNode: null
    }
}
```

下面由三个例子加深我们的理解

> 例1：旧集合A、B、C、D四个节点，更新后的新集合为B、A、D、C节点，对新旧集合进行 diff 算法差异化对比，发现 B!=A，则创建并插入B节点到新集合，并删除旧集合中A，以此类推，创建A、D、C，删除 B、C、D。如下图4-1

![element_diff](/Users/a123/Desktop/Study/JS/img/element_diff.png)

**图4-1 节点 diff**

React发现这样操作非常繁琐冗余，因为这些集合里含有相同的节点，只是节点位置发生了变化而已，却发生了繁琐的删除、创建操作，实际上只需要对这些节点进行简单的位置移动即可。

**针对这一现象，React 提出了优化策略：** 

**允许开发者对同一层级的同组子节点，添加唯一key进行区分，虽然只是小小的改动，但性能上却发生了翻天覆地的变化。**

> 例2：看下图

进行对新旧集合的 diff 差异化对比，通过 key 发现新旧集合中包含的节点是一样的，所以可以通过简单的位置移动就可以更新为新集合，React 给出的 diff 结果为：B、D不做任何操作，A、C移动即可。

![对节点进行diff差异化对比](/Users/a123/Desktop/Study/JS/img/对节点进行diff差异化对比.png)

**图4-2 对节点进行 diff 差异化对比**

步骤：

- 初始化，lastIndex = 0， nextIndex = 0

- 从新集合取出节点B，发现旧集合中也有节点B，并且B.__mountIndex = 1，lastIndex = 0，不满足 B._mountIndex < lastIndex，则不对B操作，并且更新 lastIndex= Math.max(prevChild._mountIndex, lastIndex)，并将B的位置更新为新集合中的位置prevChild._mountIndex = nextIndex，即B._mountIndex = 0, nextIndex ++ 进入下一步
- 从新集合取出节点A，发现旧集合中也有节点A，并且A.__mountIndex = 0，lastIndex = 1，满足 A._mountIndex < lastIndex，则对A进行移动操作，enqueue( updates, makeMove(prevChild, lastPlacedNode, nextIndex))并且更新 lastIndex= Math.max(prevChild._mountIndex, lastIndex)，并将A的位置更新为新集合中的位置prevChild._mountIndex = nextIndex，即A._mountIndex = 1, nextIndex ++ 进入下一步
- 依次进行操作，可以根据下面代码执行的步骤实现，这里不再赘述

操作为：

```js
updateChildren1: function(prevChildren, nextChildren) { // 旧集合 新集合 
    var updates = null
    var name
    // lastIndex 是 prevChildren 中最后一个索引，nextIndex 是 nextChildren 中每个节点的索引
    var lastIndex = 0
    var nextIndex = 0
    
    for (name in nextChildren) { // 对新集合的节点进行循环遍历
        if (!nextChildren.hasOwnProperty(name)) { 
            continue
        }
        var prevChild = prevChildren && prevChildren[name] 
        var nextChild = nextChildren[name]
        // 通过唯一的key判断新旧集合是否有相同的节点
        if (prevChild === nextChild) {  // 新旧集合有相同的节点
            // 如果子节点的 index 小于 lastIndex 则移动该节点
            if (prevChild._mountIndex < lastIndex) {
                // 获取移动节点
                let moveNode = makeMove(prevChild, lastPlacedNode, nextIndex)
                // 存入差异队列
                updates = enqueue(
                    updates,
                    moveNode
                )
            } // 这是一种顺序优化手段，lastIndex 一直在更新表示访问过的节点一直在prevChildren最大的位置，如果当前访问的节点比 lastIndex 大，说明当前访问的节点在旧结合中就比上一个节点靠后，则该节点不会影响其它节点的位置，因此不插入差异队列，不要执行移动操作，只有访问的节点比 lastIndex 小时，才需要进行移动操作。 
            // 更新lastIndex
            lastIndex= Math.max(prevChild._mountIndex, lastIndex)
            // 将prevChild的位置更新为在新集合中的位置
            prevChild._mountIndex = nextIndex
        } else {
            if (prevChild) {// 如果没有相同节点且prevChild存在
                // 更新lastIndex
                lastIndex = Math.max(prevChild._mountIndex, lastIndex)
            }
        }
        // 进入下一个节点的判断
        nextIndex ++
    }
    // 如果存在更新，则处理更新队列
    if (updates) {
        processQueue(this, updates)
    } 
    // 更新 DOM
    this._renderedChildren = nextChildren
}

function enqueue(queue, update) {
    // 如果有更新，将其存入 queue
    if (update) {
        queue = queue || []
        queue.push(update)
    }
    return queue
}

// 处理队列的更新
function processQueue (inst, updateQueue) {
    ReactComponentEnvironment.processChildrenUpdates(
        inst,
        updateQueue
    )
}
```



> 例3：看下图

![创建、删除、移动节点](/Users/a123/Desktop/Study/JS/img/创建、删除、移动节点.png)

**图4-3 创建、移动、删除节点**

可以看出在这个例子中，有新增的节点，还有需要删除的节点，具体怎么操作，请大胆的尝试一下吧

#### 5. 源码

```js
_updateChildren: function(nextNestedChildrenElements, transaction, context) {
    var prevChildren = this._renderedChildren  // 旧集合
    var removedNodes = {} // 需要删除的节点集合
    var nextChildren = this._reconcilerUpdateChildren(prevChildren, nextNestedChildrenElements, removedNodes, transaction, context) // 新集合
    
    // 如果不存在 prevChildren 及 nextChildren，则不做 diff 处理
    if (!prevChildren && !nextChildren) {
        return
    }
    var updates = null
    var name
    // lastIndex 是 prevChildren 中最后一个索引，nextIndex 是 nextChildren 中每个节点的索引
    var lastIndex = 0
    var nextIndex = 0
    var lastPlacedNode = null
    
    for (name in nextChildren) { // 对新集合的节点进行循环遍历
        if (!nextChildren.hasOwnProperty(name)) { 
            continue
        }
        var prevChild = prevChildren && prevChildren[name] 
        var nextChild = nextChildren[name]
        // 通过唯一的key判断新旧集合是否有相同的节点
        if (prevChild === nextChild) {  // 新旧集合有相同的节点
            // 如果子节点的 index 小于 lastIndex 则移动该节点，并加入差异队列
            updates = enqueue(
                updates,
                this.moveChild(prevChild, lastPlacedNode, nextIndex, lastIndex)
            )// 这是一种顺序优化手段，lastIndex 一直在更新表示访问过的节点一直在prevChildren最大的位置，如果当前访问的节点比 lastIndex 大，说明当前访问的节点在旧结合中就比上一个节点靠后，则该节点不会影响其它节点的位置，因此不插入差异队列，不要执行移动操作，只有访问的节点比 lastIndex 小时，才需要进行移动操作。 
            // 更新lastIndex
            lastIndex= Math.max(prevChild._mountIndex, lastIndex)
            // 将prevChild的位置更新为在新集合中的位置
            prevChild._mountIndex = nextIndex
        } else {
            if (prevChild) {// 如果没有相同节点且prevChild存在
                // 更新lastIndex
                lastIndex = Math.max(prevChild._mountIndex, lastIndex)
                // 通过遍历 removedNodes 删除子节点 prevChild
            }
            // 初始化并创建节点
            updates = enqueue(
                updates,
                this._mountChildAtIndex(nextChild, lastPlacedNode, nextIndex, transaction, context)
            )
        }
        // 进入下一个节点的判断
        nextIndex ++
        lastPlacedNode = ReactReconciler.getNativeNode(nextChild)
    }
    // 如果父节点不存在，则将其子节点全部移除
    for (name in removedNodes) {
        if (removedNodes.hasOwnProperty(name)) {
            updates = enqueue(
                updates,
                this._unmountChild(prevChildren[name], removedNodes[name])
            )
        }
    }
    // 如果存在更新，则处理更新队列
    if (updates) {
        processQueue(this, updates)
    } 
    this._renderedChildren = nextChildren
}

function enqueue(queue, update) {
    // 如果有更新，将其存入 queue
    if (update) {
        queue = queue || []
        queue.push(update)
    }
    return queue
}

// 处理队列的更新
function processQueue (inst, updateQueue) {
    ReactComponentEnvironment.processChildrenUpdates(
        inst,
        updateQueue
    )
}

// 移动节点
moveChild: function(child, afterNode, toIndex, lastIndex) {
    // 如果子节点的 index 小于 lastIndex 则移动该节点
    if (child._mountIndex < lastIndex) {
        return makeMove(child, afterNode, toIndex)
    }
}

// 创建节点
createChild: function(child, afterNode, mountIndex) {
    return makeInsertMarkup(mountIndex, afterNode, child._mountIndex)
}

// 删除节点
removeChild: function(child, node) {
    return makeRemove(child, node)
}

// 卸载已经渲染的子节点
_unmountChild: function(child, node) {
    var update = this.removeChild(child, node)
    child._mountIndex = null
    return update
}

// 通过提供的名称，实例化子节点
_mountChildAtIndex: function(child, afterNode, index, transaction, context) {
    var mountIndex = ReactReconciler.mountComponent(child, transaction, this, this._nativeContainerInfo, context)
    child._mountIndex = index
    return this._createChild(child, afterNode, mountIndex)
}
```



