# SafeMath库的使用

https://blog.csdn.net/q187543/article/details/86742964

https://www.jianshu.com/p/26d66aa1c122

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 首先，为什么使用SafeMath？
为避免程序结果产生溢出，开发者应在运算中使用SafeMath。
### 何为溢出？
以太坊虚拟机（EVM）为整数指定固定大小的数据类型。这意味着一个整型变量只能有一定范围的数字表示。例如，一个 uint8 ，只能存储在范围 [0,255] 的数字。试图存储 256 到一个 uint8 将变成 0。**不加注意的话，只要没有检查用户输入又执行计算，导致数字超出存储它们的数据类型允许的范围，Solidity 中的变量就可以被用来组织攻击。**

什么是 溢出 (overflow)?

假设我们有一个 uint8, 只能存储8 bit数据。这意味着我们能存储的最大数字就是二进制 11111111 (或者说十进制的 2^8 - 1 = 255).

来看看下面的代码。最后 number 将会是什么值？
```
uint8 number = 255;
number++;
```
在这个例子中，我们导致了溢出 — 虽然我们加了1， 但是number 出乎意料地等于 0了。
下溢(underflow)也类似，如果你从一个等于 0 的 uint8 减去 1, 它将变成 255 (因为 uint 是无符号的，其不能等于负数)。


整数溢出的类型包括乘法溢出，加法溢出，减法溢出三种。

**解决方案：**
- 1.在涉及到运算之前，先进行溢出判断，保证计算出的数据不超出此数据类型的表示范围。
- 2.直接使用SafeMath库函数进行计算。

![](_v_images/20190608153937211_634664050.png)


**SafeMath的使用**
第一步：导入SafeMath库文件
第二步，引用

```solidity
pragma solidity >=0.4.22 <0.6.0;
 
import "./safemath.sol";       //这里引用
 
contract jisuan{
    uint a=2;
    uint b=5;
    uint c=8;
    //引入safemath库
    using SafeMath for uint256;          //使用的方式
    
    //加法
    function addNum()public view returns(uint d){
        d=a.add(b);
    }
    //减法
    function subNum()public view returns(uint d){
        d=b.sub(a);
    }
    //乘法
    function mulNum()public view returns(uint d){
        d=a.mul(b);
    }
    //除法
    function divNum()public view returns(uint d){
        d=c.div(a);
    }
}
```

**using for 的使用**

using的前提是需要直接或者间接的导入某个library.

![](_v_images/20190608154434044_1452948760.png)

