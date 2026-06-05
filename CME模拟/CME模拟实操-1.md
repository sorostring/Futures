这些交易所，总是让人很以外。

## CME 模拟-1
- 初始时候：
**PRACTICE FUNDS**：100,000
**PROFIT/LOSS**：0.00
**MARGIN**：0.00
**AVAILABLE**：100,000.00

POSITION的情况也是什么都没有，尚且没有做任何BUYS和SELLS。

### <font color=blue>BUYS 3手成交</font>

操作：**Buy**, **QTY** = 3, **FILL PX** = 7590.75
操作完成之后，在**LAST** = 7592.00时：
**POSITION**: Long 3, BUYS 3 SELLS 0, **AVERAGE PX** = 7590.75, **Unrrealized P/L** = +18.75, **Realized P/L** = 0.00

**PRACTICE FUNDS**：100,018.75
**PROFIT/LOSS**：18.75
**MARGIN**：7,940.00
**AVAILABLE**：92,078.75

分析：
position尚未平掉，产生的盈亏都是因LAST成交价而变化的。
Long 3, Unrealized P/L = (Last 7592.00 - 7590.75)x3x5=18.75
Available = Practice Funds - Margin

### <font color=blue>BUYS 5手成交</font>

操作：**Buy**, **QTY** = 5, **FILL PX** = 7591.50
操作完成之后，在**LAST** = 7591.00时：
**POSITION**: Long 8, BUYS 8 SELLS 0, **AVERAGE PX** = 7590.22, **Unrrealized P/L** = -8.75, **Realized P/L** = 0.00

**PRACTICE FUNDS**：99,991.25
**PROFIT/LOSS**：-8.75
**MARGIN**：21,173.00
**AVAILABLE**：78,818.25

解析：
position没有平，产生的都是unrealized P/L，和LAST成交价挂钩。
细分来看，3手@7590.75 和 5手@7591.50，产生的加权均价应该是 8手@7591.218750，这里CME系统在POSITION表格中写的是 **AVERAGE PX** = 7591.22，这里不免让人疑惑，难道这么不严谨么？并不存在。这是因为：
- CME其实在POSTION TABLE中给出的AVERAGE PX是个加权值
- 但是反向操作（比如平仓）的时候，使用的原则是 **FIFO**
- **FIFO**: First In, First Out

这一点下面还会给你图示，一次给你讲明白!

### <font color=blue>SELL 6手成交</font>

操作：**Sell**, **QTY** = 6, **FILL PX** = 7593.50
操作完成之后，在**LAST** = 7594.50时：
**POSITION**: Long 2, BUYS 8 SELLS 2, **AVERAGE PX** = 7591.50, **Unrrealized P/L** = 30.00, **Realized P/L** = 71.25

**PRACTICE FUNDS**：100,101.25
**PROFIT/LOSS**：101.25
**MARGIN**：5294.00
**AVAILABLE**：94,807.25

这就是让人迷惑的地方了，SELL之前的 Long8，**AVERAGE PX** = 7590.22，怎么到了这一步骤完成后，POSITION Table中的**AVERAGE PX**，又等于 7591.50 了呢？还有就是这些
问题的核心就是：**FIFO**。你请仔细看下下图：

![](./CME_FIFO.png)

本例中，当你SELL 6手的时候，先平掉的是 `BUY 3手@7590.75`，然后是 `BUY 5手@7591.50`中的另外3个，剩余的 Long2，**average px依然还是 7591.50**！！

那么这样计算下来就有：
**REALIZED P/L** = 两部分 = (7593.50-7590.75)x3x5+(7593.50-7591.50)x3x5 = 41.25+30 = 71.25
**UNREALIZED P/L** = (7594.50-7591.5)x2x5=30
**PROFIT/LOSS** = **Unrealized P/L** + **Realized P/L** = 71.25+30=101.25
**MARGIN** = 5294.00
**AVAILABLE** = **PRACTICE FUNDS** - **MARGIN** = 100,101.25-5,294.00 = 94,807.25

### 本交易日settlement

so，折腾了一个交易日，现在本交易日出现了结算价：`7601.00$`。那么这个时候：所有的交易都要向结算价看齐了。因为LAST成交价不停在变，导致的 Unrealized P/L也在变，进而导致 Practice Funds (Equity) 也在时刻变化的情况，一瞬间打住了。Equity 突然这一瞬间不再跳个不停了。
- Realized P/L = 71.25，这个不会变了，因为都realized
- 持仓的 P/L ：相当于头寸向settlement price看齐（这就是**MTM**），那么 (7601-7591.50)x2x5= 95

理论上讲，这个时间要划转资金了，要交钱要扣钱对不。一开始有100,000.00，然后当天的equity是：
初始金额 + realized P/L + 持仓的P/L = 100,000.00 + 71.25 + 95 = 100,166.25
这个金额应该是一瞬间的，就是咬紧结算价的。但是当你又开盘的时候，practice funds/Equity又会开始跳动了！


### <font color=blue>隔夜仓第二天</font>
上一交易日的结算价：`7601.00`元

一直没下单，但是potion是有隔夜仓的情况下，POSITION table：
![](./CME-隔夜.png)

这个时候出现的就是：LAST = 7560.25
**Realized P/L** = 0
**Unrealized P/L** = (`LAST`7560.25 - `AVEARGE PX`7591.50)x2x5 = -312.50
**PROFIT/LOSS** = **Realized P/L** + **Unrealized P/L** = -312.50

但是呢，100,166.25 - (7560.25-7601.00)x2x5 = PRACTICE FUNDS = 99,758.75

那么还有一种算法：
上一个交易日的 realized P/L =71.25，这个就是我们的赚的钱了。不管隔夜持仓的问题：
初始 100,000.00 + `Realized P/L` 71.25 + `PROFIT/LOSS`-312.50 = `PRACTICE FUNDS` = 99,758.75

that's it


**PRINCIPLE**
最上面4个钱数，和 POSITION Table中的 `AVEARGE PX`、``


### SELL 2手成交

操作：**Sell**, **QTY** = 2, **FILL PX** = 7566.50

**POSITION**: 空白, BUYS 0 SELLS 2, **AVERAGE PX** 空白, **Unrrealized P/L** = 0.00, **Realized P/L** = -250.00

**PRACTICE FUNDS**：99,821.25
**PROFIT/LOSS**：-250.00
**MARGIN**：0.00
**AVAILABLE**：99,821.25

100,000.00 + 71.25 + (-250) = 99,821.25

那么金额就不会跳动了。