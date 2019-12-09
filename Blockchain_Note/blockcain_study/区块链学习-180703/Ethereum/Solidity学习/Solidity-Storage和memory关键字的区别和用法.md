# Solidity-Storage和memory关键字的区别和用法

在 Solidity 中，有两个地方可以存储变量 —— `storage`以及`memory`。

`Storage` 变量是指**永久存储在区块链中的变量**。 `Memory` 变量则是**临时的，当外部函数对某合约调用完成时，内存型变量即被移除**。

状态变量（在函数之外声明的变量）默认为`“storage”`形式，并永久写入区块链；而在函数内部声明的变量默认是`“memory”`型的，它们函数调用结束后消失。

**然而也有一些情况下，你需要手动声明存储类型，主要用于处理函数内的结构体和数组时：**

```solidity
pragma solidity ^0.4.19;

contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // 如果写成 Sandwich mySandwich = sandwiches[_index];
    // Solidity 将会给出警告，告诉你应该明确在这里定义 `storage` 或者 `memory`。
    // 这里定义为 `storage`:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...这样 `mySandwich` 是指向 `sandwiches[_index]`的指针
    // 在存储里，另外...
    mySandwich.status = "Eaten!";
    // ...这将永久把 `sandwiches[_index]` 变为区块链上的存储
    // 如果你只想要一个副本，可以使用`memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...这样 `anotherSandwich` 就仅仅是一个内存里的副本了
    // 另外
    anotherSandwich.status = "Eaten!";
    // ...将仅仅修改临时变量，对 `sandwiches[_index + 1]` 没有任何影响
    // 不过你可以这样做:
    sandwiches[_index + 1] = anotherSandwich;
    // ...如果你想把副本的改动保存回区块链存储
  }
}
```

简单的说，上述代码：

- Sandwich storage mySandwich = sandwiches[_index];拿到的是**引用/句柄/指针**
- Sandwich memory anotherSandwich = sandwiches[_index + 1];拿到的**是一份拷贝**
