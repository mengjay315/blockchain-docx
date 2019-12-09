# Security Considerations(安全考虑)

While it is usually quite easy to build software that works as expected, it is much harder to check that nobody can use it in a way that was **not** anticipated.

虽然构建按预期工作的软件通常很容易，然而，要检查没有人能以一种没有预料到的方式使用它会更加困难。

In Solidity, this is even more important because you can use smart contracts to handle tokens or, possibly, even more valuable things. Furthermore, every execution of a smart contract happens in public and, in addition to that, the source code is often available.

在Solidity中,这更为重要，因为你可以使用智能合约来处理token，或者甚至可能使用更有价值的东西。此外，智能合约的每一个执行都是公开的，除此之外，源代码通常是可以获得的。

Of course you always have to consider how much is at stake: You can compare a smart contract with a web service that is open to the public (and thus, also to malicious actors) and perhaps even open source. If you only store your grocery list on that web service, you might not have to take too much care, but if you manage your bank account using that web service, you should be more careful.

当然，你总是要考虑到有多大危险：智能合约和一个公开的web服务相比，智能合约更加开源。如果你仅仅是存储杂货列表在web服务器上，你你可能不必担心，但是如果你使用web服务器管理你的银行账户，你应该更加小心。

This section will list some pitfalls and general security recommendations but can, of course, never be complete. Also, keep in mind that even if your smart contract code is bug-free, the compiler or the platform itself might have a bug. A list of some publicly known security-relevant bugs of the compiler can be found in the [list of known bugs](https://solidity.readthedocs.io/en/v0.5.0/bugs.html#known-bugs), which is also machine-readable. Note that there is a bug bounty program that covers the code generator of the Solidity compiler.

本节将列出一些陷阱和一般安全建议，但当然可能永远不会完整。记住，即使你的智能合约代码没有错误，编译器或平台本身可能有一个bug。在`list of known bugs`这里你可以发现一个公开的编译器安全相关的bug，是机器可读的。请注意，有一个bug 奖励计划，它涵盖了Solidity编译器的代码生成器。

As always, with open source documentation, please help us extend this section (especially, some examples would not hurt)!

与往常一样，使用开源文档，请帮助我们扩展此部分

## Pitfalls(陷阱)

### Private Information and Randomness(私有的信息和随机性)

Everything you use in a smart contract is publicly visible, even local variables and state variables marked `private`.

你在一个智能合约中做的一切都是公开可见的，即使本地变量和状态变量被标记为`private`。

Using random numbers in smart contracts is quite tricky if you do not want miners to be able to cheat.

如果你不希望矿工能够作弊，在智能合约中使用随机数是非常棘手的。

### **Re-Entrancy(重入)**

Any interaction from a contract (A) with another contract (B) and any transfer of Ether hands over control to that contract (B). This makes it possible for B to call back into A before this interaction is completed. To give an example, the following code contains a bug (it is just a snippet and not a complete contract):

任何合约A和合约B的交互，任何以太币的转移，传递控制权给合约B。这使得合约B可以在此交互完成之前回调到合约A。给了一个例子，下面的代码包含了一个bug(它是一个代码片段，不是一个完整的合约)。

```javascript
pragma solidity >=0.4.0 <0.6.0;

// THIS CONTRACT CONTAINS A BUG - DO NOT USE
contract Fund {
    /// Mapping of ether shares of the contract.
    mapping(address => uint) shares;
    /// Withdraw your share.
    function withdraw() public {
        if (msg.sender.send(shares[msg.sender]))
            shares[msg.sender] = 0;
    }
}
```

The problem is not too serious here because of the limited gas as part of `send`, but it still exposes a weakness: Ether transfer can always include code execution, so the recipient could be a contract that calls back into `withdraw`. This would let it get multiple refunds and basically retrieve all the Ether in the contract. In particular, the following contract will allow an attacker to refund multiple times as it uses `call` which forwards all remaining gas by default:

这个问题在这不是很严重的，因为有`send`的gas限制。但是仍然暴露了一个缺点：以太币的转移总是包括代码的执行，因此，接收者可以是一个合约回调`withdraw`函数。这将使它获得多次退款并基本上检索合约中的所有以太币。特别地，下面的代码将会允许一个攻击者多次退款，当他使用`call`,将默认地发送完所有的gas。

```javascript
pragma solidity >=0.4.0 <0.6.0;

// THIS CONTRACT CONTAINS A BUG - DO NOT USE
contract Fund {
    /// Mapping of ether shares of the contract.
    mapping(address => uint) shares;
    /// Withdraw your share.
    function withdraw() public {
        (bool success,) = msg.sender.call.value(shares[msg.sender])("");
        if (success)
            shares[msg.sender] = 0;
    }
}
```

To avoid re-entrancy, you can use the Checks-Effects-Interactions pattern as outlined further below:

为了避免重入攻击，您可以使用下面进一步概述的`Checks-Effects-Interactions`模式：

```javascript
pragma solidity >=0.4.11 <0.6.0;

contract Fund {
    /// Mapping of ether shares of the contract.
    mapping(address => uint) shares;
    /// Withdraw your share.
    function withdraw() public {
        uint share = shares[msg.sender];
        shares[msg.sender] = 0;
        msg.sender.transfer(share);
    }
}
```

Note that re-entrancy is not only an effect of Ether transfer but of any function call on another contract. Furthermore, you also have to take multi-contract situations into account. A called contract could modify the state of another contract you depend on.

**注意**，**重入**不仅仅是以太币转移一个影响，**而是对另一个合约的任何函数调用的影响。此外，你还必须考虑多合约账户情况。一个被调用的合约可以更改你依赖的另一个合约的状态。**

### Gas Limit and Loops(gas限制和循环)

Loops that do not have a fixed number of iterations, for example, loops that depend on storage values, have to be used carefully: Due to the block gas limit, transactions can only consume a certain amount of gas. Either explicitly or just due to normal operation, the number of iterations in a loop can grow beyond the block gas limit which can cause the complete contract to be stalled at a certain point. This may not apply to `view` functions that are only executed to read data from the blockchain. Still, such functions may be called by other contracts as part of on-chain operations and stall those. Please be explicit about such cases in the documentation of your contracts.

循环没有固定的迭代次数，例如，循环依赖于存储值，必须要小心使用：由于区块gas的限制，交易仅仅花费一定的gas。无论是明确地还是仅仅由于正常操作，循环中的迭代次数可以超过区块gas限制，这可能导致完整的合约在某一点停滞。这不适用于`view`函数，因为它执行仅仅从区块链中读取数据。仍然，这些函数可以被其他合约回调，作为链操作的一部分，停滞在那里。请在你的合约文档里明确这种情况。

### Sending and Receiving Ether(发送或接收以太币)

- Neither contracts nor “external accounts” are currently able to prevent that someone sends them Ether. Contracts can react on and reject a regular transfer, but there are ways to move Ether without creating a message call. One way is to simply “mine to” the contract address and the second way is using `selfdestruct(x)`.
- 合约和外部账户当前都不能避免一些人发送以太币给他们。合约可以作出反应并拒绝定期转移，但是这有方式去移动以太币而不用创建一个信息调用。一个方式是  合约地址挖矿，第二个方式是用`selfstruct(x)`函数。

- If a contract receives Ether (without a function being called), the fallback function is executed. If it does not have a fallback function, the Ether will be rejected (by throwing an exception). During the execution of the fallback function, the contract can only rely on the “gas stipend” it is passed (2300 gas) being available to it at that time. This stipend is not enough to modify storage (do not take this for granted though, the stipend might change with future hard forks). To be sure that your contract can receive Ether in that way, check the gas requirements of the fallback function (for example in the “details” section in Remix).

  如果一个合约接收到以太币(没有一个函数被回调)，那么**fallback function**将会被执行。如果合约没有一个**fallback function**，以太币将会被拒绝(通过抛出一个异常)。由于执行了**fallback function**，合约可以仅仅依赖2300的gas薪酬。这个薪酬不够更改存储(不要认为这是理所当然的，薪酬可能在将来的硬分叉改变)。为了确保你的合约可以那种方式接收以太币，检查回调函数的gas要求(在Remix的细节章节有例子)。

- There is a way to forward more gas to the receiving contract using `addr.call.value(x)("")`. This is essentially the same as `addr.transfer(x)`, only that it forwards all remaining gas and opens up the ability for the recipient to perform more expensive actions (and it returns a failure code instead of automatically propagating the error). This might include calling back into the sending contract or other state changes you might not have thought of. So it allows for great flexibility for honest users but also for malicious actors.

  这有一种方式，当接收合约使用`addr.call.value(x)("")`时发送更多的gas，本质上和`addr.transfer(x)`相似的的，仅仅不同的是它将发送所有剩余的以太币，打开了接收者执行更昂贵操作的能力(代替自动广播错误，它返回一个错误码)。这可能包括回调发送合约或你可能没有想到的其他状态更改。因此，它为诚实用户提供了极大的灵活性，同时也为恶意行为者提供了灵活性。

- If you want to send Ether using `address.transfer`, there are certain details to be aware of:

  如果你要使用`address.transfer`发送以太币，有一些细节需要注意：

  1. If the recipient is a contract, it causes its fallback function to be executed which can, in turn, call back the sending contract.

     如果接收者是一个合约，它将会使`fallback function`执行，从而可以回调发送合同。

  2. Sending Ether can fail due to the call depth going above 1024. Since the caller is in total control of the call depth, they can force the transfer to fail; take this possibility into account or use `send` and make sure to always check its return value. Better yet, write your contract using a pattern where the recipient can withdraw Ether instead.

     发送以太币可以失败由于**调用深度达到1024**。由于调用者完全控制调用深度，因此可以强制转移失败;把这种可能性考虑在内或者使用`send`，并且确保总是检查它的返回值。**更好的方式是，写一个合约，接收者可以提取以太币**。

  3. Sending Ether can also fail because the execution of the recipient contract requires more than the allotted amount of gas (explicitly by using `require`, `assert`, `revert`, `throw` or because the operation is just too expensive) - it “runs out of gas” (OOG). If you use `transfer` or `send` with a return value check, this might provide a means for the recipient to block progress in the sending contract. Again, the best practice here is to use a [“withdraw” pattern instead of a “send” pattern](https://solidity.readthedocs.io/en/v0.5.0/common-patterns.html#withdrawal-pattern).

     发送以太币可能也会失败，因为接收者的合约执行要求比指定的gas更多(明确地使用 `require`, `assert`, `revert`, `throw`或者因为操作仅仅太昂贵了)-它用完了gas。如果你使用带有返回值检查的`transfer` or `send` ，这可能会给接收者提供一个方法在发送合约里阻塞进展。再次，最好的实践是使用提取(withdraw)模式代替发送(send)模式。

### Callstack Depth(回调堆栈 深度)

External function calls can fail any time because they exceed the maximum call stack of 1024. In such situations, Solidity throws an exception. Malicious actors might be able to force the call stack to a high value before they interact with your contract.

外部函数任何时候调用都会失败，因为它们超过了1024的最大回调深度，在这种情况下，Solidity会抛出异常。恶意的人可能在与你的合约交互前强制回调深度达到一个高的值。

Note that `.send()` does **not** throw an exception if the call stack is depleted but rather returns `false` in that case. The low-level functions `.call()`, `.callcode()`, `.delegatecall()` and `.staticcall()` behave in the same way.

注意，在那种情况下，如果回调堆栈是耗尽的，`send()`不会抛出异常，而是返回`false`。低级别的函数`.call()`, `.callcode()`, `.delegatecall()` and `.staticcall()`表现的一样。

### tx.origin

Never use `tx.origin` for authorization. Let’s say you have a wallet contract like this:

从来不要为了授权使用`tx.origin`。下面有一个钱包合约：

```javascript
pragma solidity >0.4.99 <0.6.0;

// THIS CONTRACT CONTAINS A BUG - DO NOT USE
contract TxUserWallet {
    address owner;

    constructor() public {
        owner = msg.sender;
    }

    function transferTo(address payable dest, uint amount) public {
        require(tx.origin == owner);
        dest.transfer(amount);
    }
}
```

Now someone tricks you into sending ether to the address of this attack wallet:

某人可以被欺骗你发送以太币到这个攻击钱包的地址。

```javascript
pragma solidity >0.4.99 <0.6.0;

interface TxUserWallet {
    function transferTo(address payable dest, uint amount) external;
}

contract TxAttackWallet {
    address payable owner;

    constructor() public {
        owner = msg.sender;
    }

    function() external {
        TxUserWallet(msg.sender).transferTo(owner, msg.sender.balance);
    }
}
```

If your wallet had checked `msg.sender` for authorization, it would get the address of the attack wallet, instead of the owner address. But by checking `tx.origin`, it gets the original address that kicked off the transaction, which is still the owner address. The attack wallet instantly drains all your funds.

如果你的钱包为了认证检查了`msg.sender`，将会得到攻击者钱包地址，而不是所有者地址。但是通过检查`tx.origin`，将会得到开始交易的原始地址，这个地址仍然是所有者地址。攻击钱包会立即地吸干你所有的资金

### Two’s Complement / Underflows / Overflows(两个的不足，下溢/上溢)

As in many programming languages, Solidity’s integer types are not actually integers. They resemble integers when the values are small, but behave differently if the numbers are larger. For example, the following is true: `uint8(255) + uint8(1) == 0`. This situation is called an *overflow*. It occurs when an operation is performed that requires a fixed size variable to store a number (or piece of data) that is outside the range of the variable’s data type. An *underflow* is the converse situation: `uint8(0) - uint8(1) == 255`.

在很多编程语言中，Solidity的整型不是真正的整型。当值是小的时候，类似整型，但是数字是大时就表现不一样了。例如，下面是true:`uint8(255) + uint8(1) == 0`。这种情况被叫作上溢。当一个操作被执行要求一个固定大小的变量存储一个数字(或者是数据)，它超出了变量数据类型的范围，这种情况会发生。一个下溢是相反的情况：`uint8(0) - uint8(1) == 255`。

In general, read about the limits of two’s complement representation, which even has some more special edge cases for signed numbers.



Try to use `require` to limit the size of inputs to a reasonable range and use the [SMT checker](https://solidity.readthedocs.io/en/v0.5.0/layout-of-source-files.html#smt-checker) to find potential overflows, or use a library likeSafeMath<https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/math/SafeMath.sol> if you want all overflows to cause a revert.

试图使用`require`去限制输入字节的大小到一个合理的范围，并且使用SMT checker发现一个潜在的上溢，或者使用一个像SafeMath的库，如果你想所有的上溢造成一个回滚。

### Minor Details(次要细节)

- Types that do not occupy the full 32 bytes might contain “dirty higher order bits”. This is especially important if you access `msg.data` - it poses a malleability risk: You can craft transactions that call a function `f(uint8 x)` with a raw byte argument of `0xff000001` and with `0x00000001`. Both are fed to the contract and both will look like the number `1` as far as `x` is concerned, but `msg.data` will be different, so if you use `keccak256(msg.data)` for anything, you will get different results.

  类型不占据全部的32个字节可能包含“脏的高排序位”。如果你访问`msg.data`这很重要-它具有可塑性风险：你可以使用0xff000001的原始字节参数和0x00000001来创建调用函数f（uint8 x）的交易。两者都传给合约，就`x`而言两者看起来都像数字1，但是`msg.data`将会是不同的，因此使用`keccak256(msg.data)`，你将会得到不同的结果。

## Recommendations(建议)

### Take Warnings Seriously(认真对待警告)

If the compiler warns you about something, you should better change it. Even if you do not think that this particular warning has security implications, there might be another issue buried beneath it. Any compiler warning we issue can be silenced by slight changes to the code.

如果编译器关于某些事情警告你，你应该好好地修改它。即使你不认为这个特别的警告有安全影响，在它下面可能埋藏着另一个问题。任何编译器的警告都可以对代码做轻微的改变而被压制。

Always use the latest version of the compiler to be notified about all recently introduced warnings.

总是使用最新版本的编译器可以被通知最近介绍的警告。

### Restrict the Amount of Ether(限制以太币的数量)

Restrict the amount of Ether (or other tokens) that can be stored in a smart contract. If your source code, the compiler or the platform has a bug, these funds may be lost. If you want to limit your loss, limit the amount of Ether.

限制存储在智能合约中的以太币的数量(或者其他代币)。如果你的源码，编译器，或者平台有一个bug，这些资金可能会丢失的。如果你想限制你的丢失，那么限制以太币的数量。

### Keep it Small and Modular(保持小和模块化的)

Keep your contracts small and easily understandable. Single out unrelated functionality in other contracts or into libraries. General recommendations about source code quality of course apply: Limit the amount of local variables, the length of functions and so on. Document your functions so that others can see what your intention was and whether it is different than what the code does.

让你的合约小并且容易理解的。在其他合约或库里挑出无关的功能。通常的建议是进程应用源码质量：限制变量的数量，函数的长度等等。注释你的函数，以便其他人可以看出你的意图，以及它是否与代码的不同。

### Use the Checks-Effects-Interactions Pattern

Most functions will first perform some checks (who called the function, are the arguments in range, did they send enough Ether, does the person have tokens, etc.). These checks should be done first.

大部分的函数都将会首先执行一些检查(谁调用了函数，参数是否在范围内，他们是否发送足够的以太币，这个人有多少token，等等)。这些检查首先应该被做。

As the second step, if all checks passed, effects to the state variables of the current contract should be made. Interaction with other contracts should be the very last step in any function.

作为第二步，如果所有检查都通过了，作用于当前合约应该使用的状态变量。在每一个函数中，和合约的交互应该是最后一步。

Early contracts delayed some effects and waited for external function calls to return in a non-error state. This is often a serious mistake because of the re-entrancy problem explained above.

合约过早地延时一些作用，并且等待外部的函数调用返回一个没有错误的状态。这常常是一个严重的错误，因为上面的**重入问题**已经解释了。

Note that, also, calls to known contracts might in turn cause calls to unknown contracts, so it is probably better to just always apply this pattern.

注意，对已知合约的调用可能反过来导致对未知合约的调用，因此最好始终应用此模式。

### Include a Fail-Safe Mode(包括一个失败安全模式)

While making your system fully decentralised will remove any intermediary, it might be a good idea, especially for new code, to include some kind of fail-safe mechanism:

当你的系统完全去中心化时，将会移除任何中介机构，它可能是一个好的想法，尤其是新的代码，包含一些失败安全机制：

You can add a function in your smart contract that performs some self-checks like “Has any Ether leaked?”, “Is the sum of the tokens equal to the balance of the contract?” or similar things. Keep in mind that you cannot use too much gas for that, so help through off-chain computations might be needed there.

你可以在你的合约里添加一个函数执行一些安全检查“....”。记住不能使用太多的gas，因此在这里帮助通过链下计算是需要的。

If the self-check fails, the contract automatically switches into some kind of “failsafe” mode, which, for example, disables most of the features, hands over control to a fixed and trusted third party or just converts the contract into a simple “give me back my money” contract.

如果安全检查失败，合约自动地转换到一些“失败安全模式”，例如，禁用大部分功能，把控制权转交到一个固定的并且可信的第三方，或者转换合约到一个简单的“把钱还给我”的合约。

### Ask for Peer Review

The more people examine a piece of code, the more issues are found. Asking people to review your code also helps as a cross-check to find out whether your code is easy to understand - a very important criterion for good smart contracts.

越多的人检查一段代码，越多的问题被发现，要求人们审查你的代码也有助于进行交叉检查，以确定你的代码是否易于理解 - 这是良好智能合约的一个非常重要的标准。

## Formal Verification(形式化验证)

Using formal verification, it is possible to perform an automated mathematical proof that your source code fulfills a certain formal specification. The specification is still formal (just as the source code), but usually much simpler.

使用形式化验证，可以执行自动数学证明，证明源代码符合某种正式规范。 规范仍然是正式的（就像源代码一样），但通常要简单得多。

Note that formal verification itself can only help you understand the difference between what you did (the specification) and how you did it (the actual implementation). You still need to check whether the specification is what you wanted and that you did not miss any unintended effects of it.

注意，形式化验证本身可以仅仅帮助你理解你做什么(规范)和如何做(实际的实现)的不同。你应该检查规范是否是你想要的，并且你没有错过它的任何意外影响。

