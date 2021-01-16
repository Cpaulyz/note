## FIRST&FOLLOW

### FIRST集合

First(α) 是可从 α 推导得到的句型的**首终结符号**的集合

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116145324190.png" alt="image-20210116145324190" style="zoom: 50%;" />

**计算FIRST(X)集合算法：**

* 计算每个符号X的First(X)集合

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116150338983.png" alt="image-20210116150338983" style="zoom:67%;" />

* 计算每个符号串 α 的 First(α) 集合

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116150518674.png" alt="image-20210116150518674" style="zoom:67%;" />

### FOLLOW集合

Follow(A) 是可能在某些句型中紧跟在 A 右边的终结符的集合

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116150204396.png" alt="image-20210116150204396" style="zoom:67%;" />

**计算FOLLOW(X)集合算法：**

* 为每个非终结符 X 计算 Follow(X) 集合

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116150743310.png" alt="image-20210116150743310" style="zoom:67%;" />

## LL(1)

如果文法 G 的预测分析表是无冲突的, 则 G 是 LL(1) 文法

对于当前选择的非终结符, 仅根据输入中当前的词法单元即可确定需要使用哪条产生式

> 预测分析表指明了每个非终结符在面对不同的词法单元或文件结束符时, 该选择哪个产生式 (按编号进行索引) 或者报错

* L : 从左向右 (left-to-right) 扫描输入 
* L : 构建最左 (leftmost) 推导 
* 1 : 只需向前看一个输入符号便可确定使用哪条产生式

### 构建预测分析表

先计算FIRST、FOLLOW集合

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116152100751.png" alt="image-20210116152100751" style="zoom:67%;" />

*注：或者关系，两者都要检查！*

例子：

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116152249859.png" alt="image-20210116152249859" style="zoom:67%;" />

### 改造为LL(1)

#### 消除左递归

* 直接左递归的消除方法：

	![image-20210116152611509](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116152611509.png)

	例如：

	E → E + T | T

	消除后为：

	E → T E′ 

	E ′ → + T E′ | ϵ

* 非直接左递归

	思想：无环图

	![image-20210116153035174](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116153035174.png)

	![image-20210116153044330](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116153044330.png)

	例子：

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116153215385.png" alt="image-20210116153215385" style="zoom: 67%;" />

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116153234519.png" alt="image-20210116153234519" style="zoom:67%;" />

#### 提取左公因子

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116153257570.png" alt="image-20210116153257570" style="zoom:67%;" />

### LL(1)语法分析器伪代码

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116145239560.png" alt="image-20210116145239560" style="zoom:80%;" />

## LR

### 什么是LR

L : 从左向右 (Left-to-right) 扫描输入 

R : 构建反向 (Reverse) 最右推导

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116154534821.png" alt="image-20210116154534821" style="zoom:67%;" />

两大操作: **移入输入符号** 与 **按产生式归约**

主要问题：

* **何时规约？**
* **按哪条产生式规约？**

### LR分析表

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116154817549.png" alt="image-20210116154817549" style="zoom:67%;" />

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116154852959.png" alt="image-20210116154852959" style="zoom:67%;" />

* 例子：栈上的移入与规约

	![image-20210116161844367](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116161844367.png)

* 问题：在当前状态 (编号)下, 面对当前文法符号时, 该采取什么动作

	思路：可以用自动机跟踪状态变化 (自动机中的路径 ⇔ 栈中符号/状态编号)

* 何时规约？

	必要条件: 当前状态中, 已观察到**某个产生式的完整右部**，也就是**句柄**

	> **Definition (句柄 (Handle))** 
	>
	> 在输入串的 (唯一) 反向最右推导中, 如果下一步是逆用产生式 A → α 将 α 归约为 A, 则称 α 是当前句型的句柄。
	>
	> <img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116155640688.png" alt="image-20210116155640688" style="zoom:67%;" />

* Theorem 存在一种 LR 语法分析方法, 保证句柄总是出现在栈顶

## LR(0)

### LR(0)文法

**如果文法G的LR(0)分析表是无冲突的，则G是LR(0)文法**

无冲突：ACTION表中每个单元格最多只有一种操作

* L : 从左向右 (Left-to-right) 扫描输入 

* R : 构建反向 (Reverse) 最右推导 

* 0 : 归约时无需向前看

### LR(0)自动机

**Definition (LR(0) 项 (Item))** 

文法 G 的一个 LR(0) 项是 G 的某个产生式加上一个位于体部的点

*项指明了语法分析器已经观察到了某个产生式的某个前缀*

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116155852780.png" alt="image-20210116155852780" style="zoom:67%;" />

**Definition (项集)** 

项集就是若干项构成的集合

*句柄识别自动机的一个状态可以表示为一个项集*

**Definition (项集族)** 

项集族就是若干项集构成的集合

*句柄识别自动机的状态集可以表示为一个项集族*

**Definition (增广文法 (Augmented Grammar))** 

文法 G 的增广文法 G′ 是在 G 中加入产生式 S ′ → S 得到的文法

*目的: 告诉语法分析器何时停止分析并接受输入符号串* 

*当语法分析器面对 $且要使用 S ′ → S 进行归约时, 输入符号串被接受*

**Example：LR(0) 句柄识别自动机（红圈表示接受状态）**

![image-20210116160058568](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116160058568.png)

### 构造LR(0)自动机

需要知道的：闭包的计算

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116160544085.png" alt="image-20210116160544085" style="zoom: 67%;" />

* init

	![image-20210116160621662](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116160621662.png)

* 演化

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116160734039.png" alt="image-20210116160734039" style="zoom:67%;" />

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116160645402.png" alt="image-20210116160645402" style="zoom:67%;" />

* 接受状态

	![image-20210116160754703](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116160754703.png)

* accept状态

	![image-20210116161533524](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116161533524.png)

	下，遇见$的转移

	**千万不要漏掉accept！**

### 构造LR(0)分析表

1. 先构造出LR(0)自动机，每个自动机的状态对应LR(0)分析表中的一个状态

2. 根据以下规则，构造LR(0)分析表

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116161228528.png" alt="image-20210116161228528" style="zoom:67%;" />

3. 如果文法 G 的LR(0) 分析表是无冲突的, 则 G 是 LR(0) 文法

## SLR(1)

Simple LR(1)

### LR(0)存在的问题

LR(0) 分析表每一行 (状态) 所选用的归约产生式是相同的

### 改进

对LR(0)的规约规则进行改进

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116163930031.png" alt="image-20210116163930031" style="zoom:67%;" />

## LR(1)

### LR(0)与SLR(1)存在的问题

* LR0

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116163530517.png" alt="image-20210116163530517" style="zoom:67%;" />

* SLR1

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116164433903.png" alt="image-20210116164433903" style="zoom:67%;" />

### LR(1)项

**Definition (LR(1) 项 (Item))** 

[A → α · β, a] (a ∈ T ∪ {$}) 此处, a 是向前看符号, 数量为 1

思想: α 在栈顶, 且剩余输入中开头的是可以从 βa 推导出的符号串

也就是说，[A → α·, a]只有下一个输入符号为 a 时, 才可以按照 A → α 进行归约

### LR(1)自动机

* LR1闭包计算

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116164646724.png" alt="image-20210116164646724" style="zoom:67%;" />

* LR1初始化

	![image-20210116164709796](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116164709796.png)

* LR1的GOTO计算

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116164727388.png" alt="image-20210116164727388" style="zoom: 67%;" />

* LR1自动机构造

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116164748274.png" alt="image-20210116164748274" style="zoom:67%;" />

	
### LR(1)分析表
<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116170629909.png" alt="image-20210116170629909" style="zoom:67%;" />

## LALR(1)

### LR(1)的问题

LR(1) 虽然强大, 但是生成的 LR(1) 分析表可能过大, 状态过多

LALR(1) : 合并具有相同核心 LR(0)项的状态 (忽略不同的向前看符号)

### 合并核心项

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210116171016290.png" alt="image-20210116171016290" style="zoom: 80%;" />

例如，合并图中的(4,7),(3,6),(8,9)

### 引入冲突？

对于 LR(1) 文法, 合并得到的 LALR(1) 分析表是否会引入冲突？

* **不会**引入**移入/归约**冲突

	假设合并后出现冲突，[A → α·, a] 与 [B → β · aγ, b]

	则在 LR(1) 自动机中, 存在某状态同时包含 [A → α·, a] 与 [B → β · aγ, c] (c随便是什么)

* **可能会**引入**归约/归约**冲突

