### 介绍
一种方式用来进行敏感分析是attribute grammar，或者attributed-free grammar<br/>
attribute grammar 由上下文无关文法增加了一系列指定计算的rule.<br/>
每个rule定义了值或者attribute.依据其他属性的值<br/>
rule将属性和语法标记联系起来，语法树上每一个语法标志的实例都有一个对应属性的实例<br/>
实例：
![](https://raw.githubusercontent.com/cookieli/image/master/compiler/Attribute_Grammar.png) 
这是一个符号数的上下文无关文法.<br/>
为了基于此建立attriute grammar，我们必须决定每个结点需要的性质，同时要精心打造与产生式对应的规则，这些规则为属性定义值。<br/>
定义结果如图:<br/>
![](https://raw.githubusercontent.com/cookieli/image/master/compiler/completed-contribute.png) 

由图可知，每个rule定义了一个属性的值基于常量和其他符号的属性值<br/>
rule可从左至右或由右至左的传递信息.<br/>
输入一个二进制的string，最终会产生一个annoted parse tree.如下图所示：<br/>
![](https://raw.githubusercontent.com/cookieli/image/master/compiler/parse_tree-and-dependence-trees.png) 
图左侧是注解解析树，右侧是规则依赖图。<br/>
合成属性是由其叶子结点和常量由下而上定义的属性<br/>
继承属性是由其父节点，兄弟结点，自身结点属性定义的属性。<br/>

关系依赖图捕获了数据的流动，形成了偏序图(if noncircular)，最终属性的计算顺序依赖于该偏序，计算顺序不依赖于其在grammar中的顺序。<br/>
为了完成对确定parse tree的计算，必须去构造evaluator.<br/>
我们可以通过evaluator generator构造，evaluator generator将attribute grammar的规则指定为输入，产出evaluator代码作为输出，这也是attribute grammar的好处，工具以高级别，非程序的规范作为输入，并且自动产出实现。<br/>
atribute-grammar至关重要的一点在于attribute rules可以和上下文无关文法的产生式联系起来，同时rule是函数式的，他们最终的计算结果无关于计算过程。<br/>
### Evaluation Methods
许多属性计算技巧主要分为三类:<br/>

#### 1. Dynamic methods
这种技术依赖于atttributed parse tree.高德纳的论文里在每个rule的操作数都是可用的以后可迅速计算rule.<br/>
实践中可以采用队列实现<br/>
我们还可以建立属性依赖图，拓扑排序，使用拓扑顺序计算属性<br/>

#### 2. Oblivious Methods
系统的设计者选择一种被认为适合于属性语法和评估环境的方法。计算顺序独立于attribute grammar 和 解析树。<br/>
方法有从左到右遍历，自上而下遍历，混合遍历。<br/>
这些方法实现简单，运行负荷小，但无法从树中学到信息来提高性能.<br/>
#### 3. Rule-Baesd Methods

### Circularity

前文所述的依赖图有可能出现环，解决方法;<br/>
1. avoidance 通过限制attribute grammar在一个类中，该类无法产生环，例如只允许合成属性和常数属性，更为通用: strongly noncircular attribute grammars.<br/>
2. Evaluation会给环中的属性一个合适的值。<br/>
