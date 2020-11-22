1. 请简述 React 16 版本中初始渲染的流程

   答：  
   1.获取渲染的 react root 根节点, 这是根渲染容器, 初始化容器  
   2.初始化 FiberRoot, 其由 legacyCreateRootFromDOMContainer 传递 react root 节点对象生成 3.获取 根渲染容器的第一个子元素的实例对象, 存储在 callback 中, 同步调用 updateContainer 方法处理更新
   4.updateContainer 中创建一个 update 对象, 将更新的组件、父作用域传入的 callback 函数存储在 payload 对象中, 再调用 enqueueUpdate 存储到更新队列、scheduleWork 方法调度更新目标对象  
   5.构建 Fiber 对象, 进入 render 阶段, 构建 workInProgress Fiber 树, 调用 performUnitOfWork 方法生成各个 Fiber, 其返回传入的 Fiber 的 next 属性(由父到子构建, 存储下一个要构建的子级 Fiber 对象),如果 next 属性没有值, 则说明当前 Fiber 对象没有子 Fiber, 则调用 completeUnitOfWork 方法(从下至上移动到该节点的兄弟节点, 如果一直往上没有兄弟节点就返回父节点, 最终会到达 root 节点)来搜索其他 Fiber 节点进行构建
   6.commit 阶段, 提交更新好的对象, 渲染到 dom 中

2. 为什么 React 16 版本中 render 阶段放弃了使用递归

   答：  
   1.更新操作的时候, dom 更新、渲染交替执行, 会占用主程, 可能会造成卡帧  
   2.采用循环模拟递归, 而且比对的过程是利用浏览器的空闲时间完成的, 不会长期占用主线程. 将 dom 更新与渲染操作分开, 用协调层来对 dom 中需要的更新进行标记, 用调度层来控制 dom 更新的时间, 用渲染层来进行 dom 的更新操作

3. 请简述 React 16 版本中 commit 阶段的三个子阶段分别做了什么事情

   答：  
   1.commit 1：如果类组件获取过快照(渲染过), 执行生命周期函数 getSnapshotBeforeUpdate 获取更新前的 dom 元素状态, 其返回至会作为参数传递到 componentDidUpdate  
   2.commit 2：取出要 commit 的 Fiber 对象, 从父节点到后代节点, 依次对同级节点进行比较, 生成(更新)到 Fiber 中  
   3.commit 3：提交事件、执行 componentDidMount(如果是更新, 执行 componentDidUpdate), 调用事件队列中存储的对象中的回调方法执行更新

4. 请简述 workInProgress Fiber 树存在的意义是什么

   答：在 React 中，DOM 的更新采用了双缓存技术双缓存技术, 致力于更快速的 DOM 更新.  
   1.当前 dom 中 react Root 节点的抽象层, 其内容与 react Root dom 对象的内容一致  
   2.其本身也是一个 RootFiber, 由 FiberRoot 的 current 属性指定(被指定的才称为 workInProgress Fiber), 与其他的 RootFiber 交替来更新 dom 及相关操作  
   3.这样的操作, 使得所有提交的更新先存储在 RootFiber 层, 更新时 FiberRoot 的 current 属性指向 RootFiber 时, 将更新全部渲染到 dom 中  
   4.意义：将所有更新操作的结构先存储在一个对象中, 然后再同步渲染, 避免了更新、渲染交替执行, 造成的页面卡帧问题.
