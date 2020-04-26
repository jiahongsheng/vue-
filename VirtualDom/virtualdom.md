## 虚拟dom的核心思想
   通过javascript对象表示树结构来构建一棵真正的DOM树，当数据状态发生变化时，可以直接修改这个javascript对象，接着对比修改后的javascript对象，记录
   下需要对页面做的Dom操作，然后将其应用到真正的DOM树上，实现视图的更新。
## 虚拟dom在vue2.0中的实现
   ### 创建VNode对象模拟DOM树
   在vue2.0中，virtual DOM是通过VNode类来表达的，每一个原生的Dom元素或者vue组件都对应一个VNode对象 
   ##### 1.VNode
        用来描述DOM节点的主要信息，包括tag、text、elm、data、parent、children等属性
        VNode对象主要生成方式：一种由普通的DOM对象元素生成 一种由Vue组件生成。
        两种方式的区别：在componentOptions的值上，如果是普通DOM元素生成的VNode对象，该值为空。
   ##### 2.VNodeComponentOptions
       VNode 中compentOptions属性的数据类型用来描述通过Vue组件生成VNode对象的一些组件的相关参数，
       包括Ctor、PropData、listener、parent、children、tag等属性
  ##### 3.VNodeData
       VNode 中data属性的数据类型用来描述VNode包含的一些节点数据，
       包括slot、ref、staticClass、style、class、props、attrs、transition、directives
  ##### 4.VNodeDirective
        VNode中directive属性的数据类型用来描述VNode存储的指令数据，包含name、value、oldValue、arg、modifier等
  ##### 问：VNode是如何在Vue中生成的呢？
     VNode生成最关键的点是通过调用render方法。
     render方法在vue2.0中有两种生成方式：
     1. 第一种是直接在vue对象的option中添加render字段
     2. 第二种是像vue1.x版本那样写一个模板或者指定一个el根元素，它会首先转换成模板，经过Html语法解析器生成一个ast抽象语法树，
        对语法树做优化，然后把语法树转换成代码片段，最后通过代码片段生成funtion添加到option的render字段中。
 ##### 问：ast语法树优化的过程，做了什么事情？
 1.会检测静态class名和attributes，这样他们在初始化渲染之后就永远都不会在被比对了。
 2.会检测出最大的静态子树（就是不需要动态性的子树）并且从渲染函数中萃取出来。这样每次重新渲染时，它就会直接重用完全相同的Vnode，同样跳出比对。
 
 ##### 我们在vue对象中指定el根元素，经过一系列编译操作后，最终生成render方法。
 ```
    (function(){
       with(this){
         return _h(_e('div',{
           staticAttrs:{"id":"app"}}),[_h(_e('h1'),[("hello" + _s(who))]
           )]
         )
       }
    })
 ```
 render方法使用了with方法来包裹代码块，with（this）中的属性和方法相当于通过this调用，这样写是为了减少代码量。这里的this指向的是vue对象实例
 ，_h,（_e）方法都是和Vnode相关创建的方法
 ```
 Vue.prototype._h =  renderElementWithChildren
 Vue.prototype._e =  renderElement
 ```
 ```
 1. renderElementWithChildren方法的功能是给一个VNode对象添加若干个子VNode，
 因为整个virtual DOM是一种树状结构，每个节点都可能会有若干个子节点。
 2. renderElement方法的功能是创建一个VNode对象，如果是一个reserved tag（比如html、head等合法的HTML标签），
 则会创建普通的DOM VNode对象；如果是一个component tag（通过vue注册的自定义component），则会创建Compontent VNode 对象，
 它的VNodeCompontentOptions不为null。
 ```
创建好VNode，下一步是将Virtual Dom渲染成真正的DOM
### VNode patch 生成 DOM树
      VNode转换成真正的DOM是通过patch（oldVnode，vnode，hydrating）方法实现的
      patch方法支持3个参数，其中oldVnode是一个真实的DOM或者是一个VNode对象，它表示当前的VNode；
      vnode是VNode对象类型，它表示待替换的VNode；hydrating是bool类型，它表示是否直接使用服务器渲染Dom元素。
      patch方法的运行逻辑看起来复杂。其中createElm和patchVnode是生成DOM的关键。
##### 1. createElm（vnode，insertVnodeQueue）
      该方法会根据vnode的数据结构创建真实的DOM节点，如果vnode有children，则会遍历这些子节点，递归调用createElm方法。
      InsertedVNodeQueue是记录子节点创建顺序的队列，每创建一个DOM元素就会往这个队列中插入当前的VNode。
      当整个VNode对象全部转换成真实的DOM树时，会依次调用这个队列中VNode hook的insert方法。
##### 2.patchVnode（Vnode，vnode，insertedVnodeQueue）
      该方法会通过比较新旧VNode节点，根据不同的状态对DOM做合理的更新操作（添加、删除、移动等），
      整个过程还会依次调用prepatch、update、postpatch等钩子函数。
      在编译阶段生成一些静态子树，在这个过程中由于不会改变而跳过比对。
      动态子树比较过程中比较核心的部分就是当新旧VNode同时存在children，通过updateChildren方法对子节点做更新。
#### 总结：
1. VNode通过patch方法生成DOM后，会调用mounted hook。至此，整个vue实例就创建完成了。
2. 当这个vue实例的watcher观察到数据变化时，会再次调用render方法生成新的VNode，接着调用patch方法对比新旧VNode来更新DOM。
