# JAQS回测模块介绍

JAQS有`jaqs.data`, `jaqs.research`, `jaqs.trade`, `jaqs.util`几个模块，其中回测、仿真、实盘交易都在交易模块`jaqs.trade`中实现，本文主要介绍JAQS完成回测所需的各个子模块。

## 需要哪些模块

我们把回测拆分为回测实例、策略、仓位管理、交易系统这4个子模块，下面来解释为什么：

首先，任何简单或复杂的策略的运行，都包含**决策**和**执行**这两个方面。决策即**在什么时候做什么事**，比如”当股票价格低于10元时购买200股“，”当两个期货合约价差达到3000元时卖出高价合约买入低价合约“，”每分钟按市价买入1000股股票，直到持仓为8000股“，”当模型预测小于-3时卖出对应标的“。执行即**将决策付诸实施**，通常需按一定规则来做，比如发送特定数据类型的一串信息，按顺序包含标的代码、订单类型、订单方向、订单价格、订单数量等。交易所规定的通信方式往往比较复杂，也更偏底层，而且不同交易所的规则可能不同，我们不希望在实现策略时费心考虑”我在交易哪个交易所的哪个品种“，”有哪些指令可以使用“这样的问题，因此自然地，我们会**把决策和执行分开**。分开后，前者称为交易策略，从较抽象的层面实现我们的想法；后者称为交易系统，专门负责把各种各样的决策用交易所接受的方式执行。

现在我们知道需要**交易策略**和**交易系统**两个模块，接下来我们继续拆分交易策略。

对于全自动运行的程序化交易策略，**持仓管理**十分重要，策略需要这些信息辅助决策，交易员也需要这些信息来进行监控。这里所说的持仓管理包括已发出订单的记录、撤销订单的记录、成交记录、仓位更新等，这些信息的更新与管理方式往往是固定的，如当前持仓量总是根据成交回报而更新。因此这些固定的操作也应抽象为一个子模块——**持仓管理模块**。

策略往往有参数，例如一个用到均线的策略，就需要设置均线的长度（5日、10日等），在运行策略前，需要初始化相关变量并读入这些参数配置。参数值是策略的一部分，但**读取参数**这个步骤则是固定的。除此之外，还有链接数据库、连接交易系统、按一定顺序启动并初始化各个模块等步骤，都应该从策略中分离出来。我们把这些步骤放置于**交易实例模块**中。回测时，我们还额外需要取出历史数据，模拟实盘的方式把数据交给策略，并把策略产生的订单模拟撮合，这些流程性的东西同样放在实例模块中。由于实盘与回测在这些流程上有所差异，我们分别实现了**回测实例模块**与**实盘实例模块**。

上文提到了交易系统，策略和交易系统间需要一套标准的方式来交互，即一套**API**；对于回测，需要对订单进行**模拟撮合**，因而还需要**撮合器。**

最后，为了满足模块间的相互访问及存储全局变量的需要，

作一总结，完成回测总共需要以下模块：

- 回测实例`BackestInstance`
- 策略`Strategy`
- 持仓管理`PortfolioManager`
- 交易API `TradeApi`
- 模拟撮合器`Simulator`
- 运行上下文`Context`



## 参考资料

FIX: https://www.fixtrading.org/

