# solidity-require,assert使用

### 官网的文档
- https://solidity.readthedocs.io/en/v0.5.0/control-structures.html?highlight=assert#
- Error handling: Assert, Require, Revert and Exceptions

Solidity使用state-reverting异常来处理错误。 这种异常将回滚当前调用（及其所有子调用）状态的所有变化，并将错误标志给调用者。
函数assert和require可以用于检查条件，如果条件不满足则抛出异常。

assert函数只能用于测试内部错误，并检查不变量。
应该使用require函数来确认input或合约状态变量满足条件，或者验证调用外部合约的返回值。

assert风格的异常消耗调用中可用的所有gas，而require风格的异常将不会消耗从Metropolis版本开始的任何gas。

同样作为判断一个条件是否满足的函数，require会退回剩下的gas，而assert会烧掉所有的gas。对于两个函数应该在什么情况下使用，这里引用一段原文：

”The require function should be used to ensure valid conditions, such as inputs, or contract state variables are met, or to validate return values from calls to external contracts. If used properly, analysis tools can evaluate your contract to identify the conditions and function calls which will reach a failing assert. Properly functioning code should never reach a failing assert statement; if this happens there is a bug in your contract which you should fix.“

**Revert**

revert的用法和throw很像，也会撤回所有的状态转变。但是它有两点不同：1. 它允许你返回一个值；
2. 它会把所有剩下的gas退回给caller调用起来就像这样子：revert(‘Something bad happened’);  require(condition, ‘Something bad happened’);

**适合用Require的时候：**
验证一个用户的输入是否合法：ie. require(input<20);验证一个外部协议的回应：require(external.send(amount));判断执行一段语句的前置条件： ie. require(block.number > SOME_BLOCK_NUMBER) or require(balance[msg.sender]>=amount)；
require应该被最常使用到；一般用于函数的开头处。

**适合用Revert的时候：**
和require（）应用场景差不多，适合用在逻辑复杂的情况下。

**适合用Assert的时候：**
检查有没有上溢或者是下溢： ie. c = a+b; assert(c > b)检查常数： ie. assert(this.balance >= totalSupply);在完成变化后检查状态避免本不应该发生的情况出现，如程序的bugassert不应该被经常利用到；一般用于函数结尾处

##### 总而言之，require相当于是在业务层级做的检查，而assert相当于是在代码层级做的检查，
