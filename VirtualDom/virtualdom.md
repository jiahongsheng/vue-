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
     - 第一种是直接在vue对象的option中添加render字段
     - 第二种是像vue1.x版本那样写一个模板或者指定一个el根元素，它会首先转换成模板，经过Html语法解析器生成一个ast抽象语法树，
        对语法树做优化，然后把语法树转换成代码片段，最后通过代码片段生成funtion添加到option的render字段中。
