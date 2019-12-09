# Solidity学习

## Solidity 开发文档

https://solidity.readthedocs.io/en/v0.5.0/

### Solidity in Depth » Contracts

When a contract is created, its constructor (a function declared with the constructor keyword) is executed once.

当一个合约被创建时，它的构造函数被执行一次。

A constructor is optional. Only one constructor is allowed, which means overloading is not supported.

一个构造函数是 可选的，仅仅一个构造函数是被允许的，意味着超出是不支持的。

After the constructor has executed, the final code of the contract is deployed to the blockchain. This code includes all public and external functions and all functions that are reachable from there through function calls. The deployed code does not include the constructor code or internal functions only called from the constructor.

在构造函数被执行之后，合约最后的的代码被部署到区块链上.这部分代码包含了所有 public 和 external 函数，所有的函数都可以通过函数回调到达这里.部署的代码并不包括构造函数代码或者仅仅从构造函数中调用的 internal 函数.

Internally, constructor arguments are passed **ABI encoded** after the code of the contract itself, but you do not have to care about this if you use **web3.js**.

在内部，构造函数参数在合约代码本身之后传递ABI编码，如果你使用`web3.js`不用关心这些。

If a contract wants to create another contract, the source code (and the binary) of the created contract has to be known to the creator. This means that cyclic creation dependencies are impossible.

如果合同想要创建另一个合同，则创建者要知道所创建的合同的源代码（和二进制）。这意味着循环创建依赖性是不可能的。

```solidity
pragma solidity >=0.4.22 <0.6.0;

contract OwnedToken {
    // `TokenCreator` is a contract type that is defined below.
    // It is fine to reference it as long as it is not used
    // to create a new contract.
    TokenCreator creator;
    address owner;
    bytes32 name;

    // This is the constructor which registers the
    // creator and the assigned name.
    constructor(bytes32 _name) public {
        // State variables are accessed via their name
        // and not via e.g. `this.owner`. Functions can
        // be accessed directly or through `this.f`,
        // but the latter provides an external view
        // to the function. Especially in the constructor,
        // you should not access functions externally,
        // because the function does not exist yet.
        // See the next section for details.
        owner = msg.sender;

        // We do an explicit type conversion from `address`
        // to `TokenCreator` and assume that the type of
        // the calling contract is `TokenCreator`, there is
        // no real way to check that.
        creator = TokenCreator(msg.sender);
        name = _name;
    }

    function changeName(bytes32 newName) public {
        // Only the creator can alter the name --
        // the comparison is possible since contracts
        // are explicitly convertible to addresses.
        if (msg.sender == address(creator))
            name = newName;
    }

    function transfer(address newOwner) public {
        // Only the current owner can transfer the token.
        if (msg.sender != owner) return;

        // We ask the creator contract if the transfer
        // should proceed by using a function of the
        // `TokenCreator` contract defined below. If
        // the call fails (e.g. due to out-of-gas),
        // the execution also fails here.
        if (creator.isTokenTransferOK(owner, newOwner))
            owner = newOwner;
    }
}

contract TokenCreator {
    function createToken(bytes32 name)
       public
       returns (OwnedToken tokenAddress)
    {
        // Create a new `Token` contract and return its address.
        // From the JavaScript side, the return type is
        // `address`, as this is the closest type available in
        // the ABI.
        return new OwnedToken(name);
    }

    function changeName(OwnedToken tokenAddress, bytes32 name) public {
        // Again, the external type of `tokenAddress` is
        // simply `address`.
        tokenAddress.changeName(name);
    }

    // Perform checks to determine if transferring a token to the
    // `OwnedToken` contract should proceed
    function isTokenTransferOK(address currentOwner, address newOwner)
        public
        pure
        returns (bool ok)
    {
        // Check an arbitrary condition to see if transfer should proceed
        return keccak256(abi.encodePacked(currentOwner, newOwner))[0] == 0x7f;
    }
}
```

### Function Modifiers

Modifiers can be used to easily change the behaviour of functions. For example, they can automatically check a condition prior to executing the function. Modifiers are inheritable properties of contracts and may be overridden by derived contracts.

修饰符可以很容易地用来改变函数的行为.例如，他们可以在执行函数之前自动检查条件.修饰符是合约的可继承属性，可以由派生合同覆盖。

**payable**

```solidity
contract Register is priced, owned {
    mapping (address => bool) registeredAddresses;
    uint price;

    constructor(uint initialPrice) public { price = initialPrice; }

    // It is important to also provide the
    // `payable` keyword here, otherwise the function will
    // automatically reject all Ether sent to it.
    function register() public payable costs(price) {
        registeredAddresses[msg.sender] = true;
    }

    function changePrice(uint _price) public onlyOwner {
        price = _price;
    }
}
```
**Mutex**

```solidity
contract Mutex {
    bool locked;
    modifier noReentrancy() {
        require(
            !locked,
            "Reentrant call."
        );
        locked = true;
        _;
        locked = false;
    }

    /// This function is protected by a mutex, which means that
    /// reentrant calls from within `msg.sender.call` cannot call `f` again.
    /// The `return 7` statement assigns 7 to the return value but still
    /// executes the statement `locked = false` in the modifier.
    function f() public noReentrancy returns (uint) {
        (bool success,) = msg.sender.call("");
        require(success);
        return 7;
    }
}
```

In an earlier version of Solidity, return statements in functions having modifiers behaved differently.

在早期版本的Solidity中，具有修饰符的函数中的return语句表现不同。

Explicit returns from a modifier or function body only leave the current modifier or function body. Return variables are assigned and control flow continues after the “_” in the preceding(先前的) modifier.

修饰符或函数体的显式返回仅保留当前修饰符或函数体。返回变量已分配，控制流在前一个修饰符中的“_”后继续。

Arbitrary expressions are allowed for modifier arguments and in this context, all symbols visible from the function are visible in the modifier. Symbols introduced in the modifier are not visible in the function (as they might change by overriding).

修饰符参数允许使用任意表达式，在此上下文中，函数中可见的所有符号在修饰符中都是可见的。修饰符中引入的符号在函数中不可见（因为它们可能会因重写而改变）。

### Functions

#### View Functions

**Functions can be declared `view` in which case they promise not to modify the state.**

**函数可以被声明为 `view`, 在这种情况下，它们承诺不改变状态。**

#### The following statements are considered modifying the stat

下面的声明被认为是会改变状态：

- Writing to state variables.
- Emitting events.
- Creating other contracts.
- Using selfdestruct.
- Sending Ether via calls.
- Calling any function not marked `view` or` pure`.
- Using low-level calls.
- Using inline assembly that contains certain opcodes.
- 使用包含某些操作码的内联汇编。

- **constant **on functions used to be an alias to `view`, but this was dropped in version 0.5.0.
- **constant**在函数中有一个别名 `view`, 但是在version 0.5.0被丢掉了。

- Getter methods are automatically marked `view`.
- 获得者方法被自动标记为 `view`.

#### Pure Functions

**Functions can be declared `pure` in which case they promise not to read from or modify the state.**

**函数被声明为pure,在这种情况下承诺不从状态中读取数据或更改状态**

- **Note:**
- If the compiler’s EVM target is Byzantium or newer (default) the opcode STATICCALL is used, which does not guarantee that the state is not read, but at least that it is not modified.
- 如果编译器的EVM的目标是 Byzantium 或者更新的(默认的)，则使用操作码STATICCALL，这不保证不读取状态，但至少不会修改状态。

#### In addition to the list of state modifying statements explained above, the following are considered reading from the state:

除了上面的状态更改的说明之外，下面的说明被认为是从状态中读取数据：

- Reading from state variables.
- Accessing` address(this).balance` or `<address>.balance`.
- Accessing any of the members of block, tx, msg (with the exception of msg.sig and msg.data).
- Calling any function not marked pure.
- Using inline assembly that contains certain opcodes.

- **Note:**
- Prior to version 0.5.0, the compiler did not use the `STATICCALL` opcode for `pure `functions. This enabled state modifications in pure functions through the use of invalid explicit type conversions. By using `STATICCALL` for pure functions, modifications to the state are prevented on the level of the EVM.

- **Warning:**
- It is not possible to prevent functions from reading the state at the level of the EVM, it is only possible to prevent them from writing to the state (i.e. only `view `can be enforced at the EVM level, `pure` can not).
- 不可能阻止函数在EVM级别读取状态，只能防止它们写入状态（在EVM级别`view`可以被强制执行，`pure`不能).

- Before version 0.4.17 the compiler did not enforce that `pure` is not reading the state. It is a compile-time type check, which can be circumvented doing invalid explicit conversions between contract types, because the compiler can verify that the type of the contract does not do state-changing operations, but it cannot check that the contract that will be called at runtime is actually of that type.
- 在0.4.17版本之前，编译器没有强制执行`pure`不读取状态.它是一种编译时类型检查，可以避免在合约类型之间进行无效的显式转换，因为编译器可以验证合约的类型，不会做状态改变操作，但它无法检查将在运行时调用的合约实际上是该类型。

#### **Fallback Function**

A contract can have exactly one unnamed function. This function cannot have arguments, cannot return anything and has to have `external `visibility. It is executed on a call to the contract if none of the other functions match the given function identifier (or if no data was supplied at all).

**一个合约可以恰好有一个未命名的函数.这个函数不能有参数，不能返回任何值，并且必须有外部可见性.如果没有其他函数与给定的函数标识符匹配（或者根本没有提供数据），则在调用合约时执行它。**

Furthermore, this function is executed whenever the contract receives plain Ether (without data). Additionally, in order to receive Ether, the fallback function must be marked `payable`. If no such function exists, the contract cannot receive Ether through regular transactions.

**此外，只要合约收到普通以太币（没有数据），就会执行此函数。此外，为了接收以太币，fallback 函数必须被标记为`payable`.如果没有这样的函数存在，合约不可能从常规的交易中接收以太币。**

In the worst case, the fallback function can only rely on 2300 gas being available (for example when send or transfer is used), leaving little room to perform other operations except basic logging. The following operations will consume more gas than the 2300 gas stipend:

在最坏的情况下，the fallback fuction只能依赖2300gas(例如，当发送或转移时用到),除基本日志记录外，几乎没有空间执行其他操作。以下操作将消耗比2300gas补贴更多的gas：

- Writing to storage
- 向存储器中写数据
- Creating a contract
- 创建一个合约
- Calling an external function which consumes a large amount of gas
- 回调一个会消耗大量gas的外部函数
- Sending Ether
- 发送以太币

- Like any function, the fallback function can execute complex operations as long as there is enough gas passed on to it.
- 与任何函数一样，只要有足够的gas传递给它，**the fallback 函数**就可以执行复杂的操作。

- **Note**
- Even though the fallback function cannot have arguments, one can still use msg.data to retrieve any payload supplied with the call.
- 即使the fallback 函数不能有参数，仍然可以使用msg.data来检索随调用提供的任何payload。

- **Warning**
-  The fallback function is also executed if the caller meant to call a function that is not available. If you want to implement the fallback function only to receive ether, you should add a check like` require(msg.data.length == 0)` to prevent invalid calls.
- 如果调用者打算调用不可用的函数，也会执行 the fallback 函数。如果要实现the fallback，只接收以太币，你可以添加一个检查像`require(msg.data.length == 0)` 去避免无效的回调。

- **Warning**
- Contracts that receive Ether directly (without a function call, i.e. using `send` or `transfer`) but do not define a fallback function throw an exception, sending back the Ether (this was different before Solidity v0.4.0). So if you want your contract to receive Ether, you have to implement a payable fallback function.
- **合约可以直接接收以太币**(不用回调函数，使用 `send`或 `transfer`)**但是没有定义the fallback 函数抛出异常**，发送回Ether（这在Solidity v0.4.0之前是不同的）。**因此，如果你想你的合约接收以太币，你必须实现`payable` fallback 函数。**

- **Warning**
- A contract without a payable fallback function can receive Ether as a recipient of a coinbase transaction (aka miner block reward) or as a destination of a `selfdestruct`.
- **没有payable fallback函数的合约可以接收以太币作为coinbase交易的接收者（也称为矿工块奖励）或作为自我毁灭的终点**。

- A contract cannot react to such Ether transfers and thus also cannot reject them. This is a design choice of the EVM and Solidity cannot work around it.
- 合约不能对此类以太币转移作出反应，因此也不能拒绝它们。这是EVM的设计选择，而Solidity无法解决它。

- It also means that `address(this).balance` can be higher than the sum of some manual accounting implemented in a contract (i.e. having a counter updated in the fallback function).
- 它还意味着`address(this).balance`可以高于合约中实现的某些手工会计的总和（即在fallback 函数中更新计数器）。

```javascript
pragma solidity >0.4.99 <0.6.0;

contract Test {
    // This function is called for all messages sent to
    // this contract (there is no other function).
    // Sending Ether to this contract will cause an exception,
    // because the fallback function does not have the `payable`
    // modifier.
    function() external { x = 1; }
    uint x;
}


// This contract keeps all Ether sent to it with no way
// to get it back.
contract Sink {
    function() external payable { }
}

contract Caller {
    function callTest(Test test) public returns (bool) {
        (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
        require(success);
        // results in test.x becoming == 1.

        // address(test) will not allow to call ``send`` directly, since ``test`` has no payable
        // fallback function. It has to be converted to the ``address payable`` type via an
        // intermediate conversion to ``uint160`` to even allow calling ``send`` on it.
        address payable testPayable = address(uint160(address(test)));

        // If someone sends ether to that contract,
        // the transfer will fail, i.e. this returns false here.
        return testPayable.send(2 ether);
    }
}
```

#### Function Overloading 函数重载

A contract can have multiple functions of the same name but with different parameter types. This process is called “overloading” and also applies to inherited functions. The following example shows overloading of the function `f`in the scope of contract` A`.

```solidity
pragma solidity >=0.4.16 <0.6.0;

contract A {
    function f(uint _in) public pure returns (uint out) {
        out = _in;
    }

    function f(uint _in, bool _really) public pure returns (uint out) {
        if (_really)
            out = _in;
    }
}
```

Overloaded functions are also present in the external interface. It is an error if two externally visible functions differ by their Solidity types but not by their external types.

```solidity
pragma solidity >=0.4.16 <0.6.0;

// This will not compile
contract A {
    function f(B _in) public pure returns (B out) {
        out = _in;
    }

    function f(address _in) public pure returns (address out) {
        out = _in;
    }
}

contract B {
}
```

Both f function overloads above end up accepting the address type for the ABI although they are considered different inside Solidity.

上述两个f函数重载最终都接受了ABI的地址类型，尽管它们在Solidity内被认为是不同的。


##### Overload resolution and Argument matching   重载解析和参数匹配

- Overloaded functions are selected by matching the function declarations in the current scope to the arguments supplied in the function call. Functions are selected as overload candidates if all arguments can be implicitly converted to the expected types. If there is not exactly one candidate, resolution fails.
- 通过将当前作用域中的函数声明与函数调用中提供的参数进行匹配来选择重载函数。 如果所有参数都可以隐式转换为期望的类型，则选择函数作为重载候选，如果没有一个候选者，则解析失败。

- **Note**
- Return parameters are not taken into account for overload resolution.
- 重载解析不考虑返回的参数。

```solidity
pragma solidity >=0.4.16 <0.6.0;

contract A {
    function f(uint8 _in) public pure returns (uint8 out) {
        out = _in;
    }

    function f(uint256 _in) public pure returns (uint256 out) {
        out = _in;
    }
}
```

Calling `f(50)` would create a type error since `50` can be implicitly converted both to `uint8` and `uint256` types. On another hand` f(256)` would resolve to `f(uint256)` overload as `256` cannot be implicitly converted to` uint8`.

#### Events

Solidity events give an abstraction on top of the EVM’s logging functionality. Applications can subscribe and listen to these events through the RPC interface of an Ethereum client.

Solidity事件在EVM的日志记录功能之上提供了抽象。应用程序可以通过以太坊客户端的RPC接口订阅和监听这些事件。

Events are inheritable members of contracts. When you call them, they cause the arguments to be stored in the transaction’s log - a special data structure in the blockchain. These logs are associated with the address of the contract, are incorporated into the blockchain, and stay there as long as a block is accessible (forever as of the Frontier and Homestead releases, but this might change with Serenity). The Log and its event data is not accessible from within contracts (not even from the contract that created them).

It is possible to request a simple payment verification (SPV) for logs, so if an external entity supplies a contract with such a verification, it can check that the log actually exists inside the blockchain. You have to supply block headers because the contract can only see the last 256 block hashes.

You can add the attribute `indexed` to up to three parameters which adds them to a special data structure known as `“topics” `instead of the data part of the log. If you use arrays (including `string` and `bytes`) as indexed arguments, its Keccak-256 hash is stored as a topic instead, this is because a topic can only hold a single word (32 bytes).

All parameters without the `indexed` attribute are **ABI-encoded **into the data part of the log.

Topics allow you to search for events, for example when filtering a sequence of blocks for certain events. You can also filter events by the address of the contract that emitted the event.

For example, the code below uses the `web3.js subscribe("logs")` `method` to filter logs that match a topic with a certain address value:

```javascript
var options = {
    fromBlock: 0,
    address: web3.eth.defaultAccount,
    topics: ["0x0000000000000000000000000000000000000000000000000000000000000000", null, null]
};
web3.eth.subscribe('logs', options, function (error, result) {
    if (!error)
        console.log(result);
})
    .on("data", function (log) {
        console.log(log);
    })
    .on("changed", function (log) {
});
```

The hash of the signature of the event is one of the topics, except if you declared the event with the anonymous specifier. This means that it is not possible to filter for specific `anonymous` events by name.

```solidity
pragma solidity >=0.4.21 <0.6.0;

contract ClientReceipt {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );

    function deposit(bytes32 _id) public payable {
        // Events are emitted using `emit`, followed by
        // the name of the event and the arguments
        // (if any) in parentheses. Any such invocation
        // (even deeply nested) can be detected from
        // the JavaScript API by filtering for `Deposit`.
        emit Deposit(msg.sender, _id, msg.value);
    }
}
```
The use in the JavaScript API is as follows:

```javascript
var abi = /* abi as generated by the compiler */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

var event = clientReceipt.Deposit();

// watch for changes
event.watch(function(error, result){
    // result contains non-indexed arguments and topics
    // given to the `Deposit` call.
    if (!error)
        console.log(result);
});


// Or pass a callback to start watching immediately
var event = clientReceipt.Deposit(function(error, result) {
    if (!error)
        console.log(result);
});
```

The output of the above looks like the following (trimmed):

```json
{
   "returnValues": {
       "_from": "0x1111…FFFFCCCC",
       "_id": "0x50…sd5adb20",
       "_value": "0x420042"
   },
   "raw": {
       "data": "0x7f…91385",
       "topics": ["0xfd4…b4ead7", "0x7f…1a91385"]
   }
}
```

##### Low-Level Interface to Logs  日志的低级接口

It is also possible to access the low-level interface to the logging mechanism via the functions `log0`, `log1`,` log2`, `log3` and `log4.` `logi `takes` i + 1 `parameter of type `bytes32`, where the first argument will be used for the data part of the log and the others as topics. The event call above can be performed in the same way as

```solidity
pragma solidity >=0.4.10 <0.6.0;

contract C {
    function f() public payable {
        uint256 _id = 0x420042;
        log3(
            bytes32(msg.value),
            bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
            bytes32(uint256(msg.sender)),
            bytes32(_id)
        );
    }
}
```

where the long hexadecimal(十六进制) number is equal to` keccak256("Deposit(address,bytes32,uint256)")`, the signature of the event.

#### Additional Resources for Understanding Events (理解事件的额外资源)

- Javascript documentation        https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events
- Example usage of events        https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol
- How to access them in js        https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js

#### Inheritance(继承)

Solidity supports multiple inheritance by copying code including polymorphism.

All function calls are virtual, which means that the most derived function is called, except when the contract name is explicitly given.

When a contract inherits from other contracts, only a single contract is created on the blockchain, and the code from all the base contracts is copied into the created contract.

The general inheritance system is very similar to `Python’s`(https://docs.python.org/3/tutorial/classes.html#inheritance), especially concerning multiple inheritance, but there are also some `differences`(https://solidity.readthedocs.io/en/v0.5.0/contracts.html?highlight=view#multi-inheritance).

Details are given in the following example.

```solidity
pragma solidity >0.4.99 <0.6.0;

contract owned {
    constructor() public { owner = msg.sender; }
    address payable owner;
}

// Use `is` to derive from another contract. Derived
// contracts can access all non-private members including
// internal functions and state variables. These cannot be
// accessed externally via `this`, though.
contract mortal is owned {
    function kill() public {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

// These abstract contracts are only provided to make the
// interface known to the compiler. Note the function
// without body. If a contract does not implement all
// functions it can only be used as an interface.
contract Config {
    function lookup(uint id) public returns (address adr);
}

contract NameReg {
    function register(bytes32 name) public;
    function unregister() public;
 }

// Multiple inheritance is possible. Note that `owned` is
// also a base class of `mortal`, yet there is only a single
// instance of `owned` (as for virtual inheritance in C++).
contract named is owned, mortal {
    constructor(bytes32 name) public {
        Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
        NameReg(config.lookup(1)).register(name);
    }

    // Functions can be overridden by another function with the same name and
    // the same number/types of inputs.  If the overriding function has different
    // types of output parameters, that causes an error.
    // Both local and message-based function calls take these overrides
    // into account.
    function kill() public {
        if (msg.sender == owner) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).unregister();
            // It is still possible to call a specific
            // overridden function.
            mortal.kill();
        }
    }
}

// If a constructor takes an argument, it needs to be
// provided in the header (or modifier-invocation-style at
// the constructor of the derived contract (see below)).
contract PriceFeed is owned, mortal, named("GoldFeed") {
   function updateInfo(uint newInfo) public {
      if (msg.sender == owner) info = newInfo;
   }

   function get() public view returns(uint r) { return info; }

   uint info;
}
```

Note that above, we call `mortal.kill()` to “forward” the destruction request. The way this is done is problematic, as seen in the following example:

注意上面，我们调用mortal.kill（）来“转发”销毁请求。完成此操作的方式存在问题，如以下示例所示：

```solidity
pragma solidity >=0.4.22 <0.6.0;

contract owned {
    constructor() public { owner = msg.sender; }
    address payable owner;
}

contract mortal is owned {
    function kill() public {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

contract Base1 is mortal {
    function kill() public { /* do cleanup 1 */ mortal.kill(); }
}

contract Base2 is mortal {
    function kill() public { /* do cleanup 2 */ mortal.kill(); }
}

contract Final is Base1, Base2 {
}
```

A call to `Final.kill()` will call `Base2.kill` as the most derived override, but this function will bypass `Base1.kill`, basically because it does not even know about `Base1`. The way around this is to use `super`:



```solidity
pragma solidity >=0.4.22 <0.6.0;

contract owned {
    constructor() public { owner = msg.sender; }
    address payable owner;
}

contract mortal is owned {
    function kill() public {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

contract Base1 is mortal {
    function kill() public { /* do cleanup 1 */ super.kill(); }
}


contract Base2 is mortal {
    function kill() public { /* do cleanup 2 */ super.kill(); }
}

contract Final is Base1, Base2 {
}
```

If `Base2` calls a function of `super`, it does not simply call this function on one of its base contracts. Rather, it calls this function on the next base contract in the final inheritance graph, so it will call `Base1.kill()` (note that the final inheritance sequence is – starting with the most derived contract: Final, Base2, Base1, mortal, owned). The actual function that is called when using super is not known in the context of the class where it is used, although its type is known. This is similar for ordinary virtual method lookup.

##### Constructors

- A constructor is an optional function declared with the `constructor` keyword which is executed upon contract creation, and where you can run contract initialisation code.
- 构造函数是使用constructor关键字声明的可选函数，该关键字在创建合约时执行，您可以在其中运行合约初始化代码。

- Before the constructor code is executed, state variables are initialised to their specified value if you initialise them inline, or zero if you do not.
- 在执行构造函数代码之前，如果以内联方式初始化状态变量，则将状态变量初始化为指定值，否则初始化为零。

-After the constructor has run, the final code of the contract is deployed to the blockchain. The deployment of the code costs additional gas linear to the length of the code. This code includes all functions that are part of the public interface and all functions that are reachable from there through function calls. It does not include the constructor code or internal functions that are only called from the constructor.
- 构造函数运行后，合约的最终代码将部署到区块链.代码的部署花费额外的gas线性到代码的长度(也就是说花费gas的多少依赖于代码量)。此代码包括作为公开接口一部分的所有函数以及可通过函数调用从那里访问的所有函数。它不包括仅从构造函数调用的构造函数代码或内部函数。

Constructor functions can be either` public` or `internal`. If there is no constructor, the contract will assume the default constructor, which is equivalent to `constructor() public {}`. For example:

```solidity
pragma solidity >0.4.99 <0.6.0;

contract A {
    uint public a;

    constructor(uint _a) internal {
        a = _a;
    }
}

contract B is A(1) {
    constructor() public {}
}
```

A constructor set as internal causes the contract to be marked as abstract.

- **Warning**
- Prior to version 0.4.22, constructors were defined as functions with the same name as the contract. This syntax was deprecated and is not allowed anymore in version 0.5.0.
- 在版本0.4.22之前，构造函数被定义为与合约具有相同名称的函数。不推荐使用此语法，在0.5.0版中不再允许使用此语法。

##### Arguments for Base Constructors

The constructors of all the base contracts will be called following the linearization rules explained below. If the base constructors have arguments, derived contracts need to specify all of them. This can be done in two ways:

```solidity
pragma solidity >=0.4.22 <0.6.0;

contract Base {
    uint x;
    constructor(uint _x) public { x = _x; }
}

// Either directly specify in the inheritance list...
contract Derived1 is Base(7) {
    constructor() public {}
}

// or through a "modifier" of the derived constructor.
contract Derived2 is Base {
    constructor(uint _y) Base(_y * _y) public {}
}
```

One way is directly in the inheritance list (`is Base(7)`). The other is in the way a modifier is invoked as part of the derived constructor (`Base(_y * _y)`). The first way to do it is more convenient if the constructor argument is a constant and defines the behaviour of the contract or describes it. The second way has to be used if the constructor arguments of the base depend on those of the derived contract. Arguments have to be given either in the inheritance list or in modifier-style in the derived constructor. Specifying arguments in both places is an error.

如果基类的构造函数参数依赖于派生合同的参数，则必须使用第二种方法。参数必须在继承列表中或在派生构造函数中以修饰符样式给出。在两个地方指定参数是一个错误。

If a derived contract does not specify the arguments to all of its base contracts’ constructors, it will be abstract.

如果派生合约没有指定所有基础合约构造函数的参数，那么它将是抽象的。

##### Multiple Inheritance and Linearization  多重继承和线性化

Languages that allow multiple inheritance have to deal with several problems. One is the `Diamond Problem`. Solidity is similar to Python in that it uses `“C3 Linearization”` to force a specific order in the directed acyclic graph (DAG) of base classes. This results in the desirable property of monotonicity but disallows some inheritance graphs. Especially, the order in which the base classes are given in the `is` directive is important: You have to list the direct base contracts in the order from “most base-like” to “most derived”. Note that this order is the reverse of the one used in Python.

允许多重继承的语言必须处理几个问题。一个是 `Diamond Problem(https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)`.Solidity类似于Python，因为它使用“C3线性化(https://en.wikipedia.org/wiki/C3_linearization)”来强制基类的有向无环图（DAG）中的特定顺序，这导致了单调性的理想特性，但不允许一些遗传图。特别是，在is指令中给出基类的顺序很重要：你必须按照从“最类似基数”到“最多派生”的顺序​​列出直接基本合约。请注意，此顺序与Python中使用的顺序相反。

Another simplifying way to explain this is that when a function is called that is defined multiple times in different contracts, the given bases are searched from right to left (left to right in Python) in a depth-first manner, stopping at the first match. If a base contract has already been searched, it is skipped.

解释这一点的另一种简化方法是，在不同的合约中调用多次定义的函数，以深度优先的方式从右到左（在Python中从左到右）搜索给定的基数，在第一次搜索后停下来。如果已经搜索了基本合约，则会跳过该合约。

In the following code, Solidity will give the **error** “Linearization of inheritance graph impossible”.

```solidity
pragma solidity >=0.4.0 <0.6.0;

contract X {}
contract A is X {}
// This will not compile
contract C is A, X {}
```

The reason for this is that   `C` requests `X` to override `A` (by specifying `A`, `X` in this order), but `A` itself requests to override `X`, which is a contradiction that cannot be resolved.

原因是 `C`请求 `X`去覆盖 `A`(通过指定 `A`, `X`以这样排序)，但是 ` A`自身可以请求去覆盖 `X`,这是一个不能解决的矛盾。

##### Inheriting Different Kinds of Members of the Same Name

When the inheritance results in a contract with a function and a modifier of the same name, it is considered as an error. This error is produced also by an event and a modifier of the same name, and a function and an event of the same name. As an exception, a state variable getter can override a public function.

当继承导致与函数和同名修饰符的合约时，它被认为是一个错误。此错误也由同名的事件和修饰符产生，以及同名的函数和事件，作为例外，状态变量getter可以覆盖公开函数。

#### Abstract Contracts(抽象合约)

Contracts are marked as abstract when at least one of their functions lacks an implementation as in the following example (note that the function declaration header is terminated by `;`):

当至少其中一个函数缺少实现时，合同被标记为抽象，如下例所示（注意函数声明头以;结尾):

```solidity
pragma solidity >=0.4.0 <0.6.0;

contract Feline {
    function utterance() public returns (bytes32);
}
```

Such contracts cannot be compiled (even if they contain implemented functions alongside non-implemented functions), but they can be used as base contracts:

此类合约无法编译（即使它们包含已实现的函数以及未实现的函数），但它们可用作基本合约：

```solidity
pragma solidity >=0.4.0 <0.6.0;

contract Feline {
    function utterance() public returns (bytes32);
}

contract Cat is Feline {
    function utterance() public returns (bytes32) { return "miaow"; }
}
```

- If a contract inherits from an abstract contract and does not implement all non-implemented functions by overriding, it will itself be abstract.
- 如果合约继承自抽象合约并且没有通过覆盖实现所有未实现的函数，那么它本身就是抽象的。

- Note that a function without implementation is different from a Function Type even though their syntax looks very similar.
-请注意，没有实现的函数与函数类型不同，即使它们的语法看起来非常相似。 

- Example of function without implementation (a function declaration):
- 没有实现的函数示​​例（函数声明）：

```solidity
function foo(address) external returns (address);
```

- Example of a Function Type (a variable declaration, where the variable is of type `function`):
- 函数类型的示例（变量声明，其中变量的类型为function）：

```solidity
function(address) external returns (address) foo;
```

Abstract contracts decouple the definition of a contract from its implementation providing better extensibility and self-documentation and facilitating patterns like the Template method and removing code duplication. Abstract contracts are useful in the same way that defining methods in an interface is useful. It is a way for the designer of the abstract contract to say “any child of mine must implement this method”.

抽象合约将合约的定义与其实现分离，从而提供更好的可扩展性和自我记录，并促进模板方法和删除代码重复等模式。抽象合约与在接口中定义方法很有用的方式非常有用。这是抽象合同的设计者说“我的任何一个孩子必须实现这种方法”的一种方式。

#### Interfaces

Interfaces are similar to abstract contracts, but they cannot have any functions implemented. There are further restrictions:

接口类似于抽象合约，但它们不能实现任何功能。还有其他限制：

- They cannot inherit other contracts or interfaces.
- 它们不能继承其他合约或接口
- All declared functions must be external.
- 所有声明的函数必须是外部的
- They cannot declare a constructor.
- 它们不能声明构造函数
- They cannot declare state variables.
- 它们不能声明状态=变量

- Some of these restrictions might be lifted in the future.
- 其中一些限制可能会在将来解除。

-Interfaces are basically limited to what the Contract ABI can represent, and the conversion between the ABI and an interface should be possible without any information loss.
- 接口基本上限于Contract ABI可以表示的内容，并且ABI和接口之间的转换应该是可能的，而不会丢失任何信息。

Interfaces are denoted by their own keyword:

接口由它们自己的关键字表示：

```solidity
pragma solidity >=0.4.11 <0.6.0;

interface Token {
    enum TokenType { Fungible, NonFungible }
    struct Coin { string obverse; string reverse; }
    function transfer(address recipient, uint amount) external;
}
```

- Contracts can inherit interfaces as they would inherit other contracts.
- 合约可以继承接口，因为它们将继承其他合约。

- Types defined inside interfaces and other contract-like structures can be accessed from other contracts: `Token.TokenType` or `Token.Coin`.
- 接口和其他类似合约结构中定义的类型可以从其他合约访问：Token.TokenType或Token.Coin。

#### Libraries（这一部分，后面的还不是和明白，等遇到例子再看）

Libraries are similar to contracts, but their purpose is that they are deployed only once at a specific address and their code is reused using the `DELEGATECALL` (`CALLCODE` until Homestead) feature of the EVM. This means that if library functions are called, their code is executed in the context of the calling contract, i.e. `this` points to the calling contract, and especially the storage from the calling contract can be accessed. As a library is an isolated piece of source code, it can only access state variables of the calling contract if they are explicitly supplied (it would have no way to name them, otherwise). Library functions can only be called directly (i.e. without the use of` DELEGATECALL`) if they do not modify the state (i.e. if they are `view` or `pure` functions), because libraries are assumed to be stateless. In particular, it is not possible to destroy a library.

库是和合约相似的，但是它们的目的是仅仅在指定的地址上部署一次，使用EVM的DELEGATECALL（CALLCODE直到Homestead）功能重用它们的代码。这就意味着如果库函数被调用，它们的代码在被调用的合约的上下文中执行，例如，this指向回调合约，尤其是存储器，可以从回调合约那里访问。由于库是一个孤立的源代码，如果它们被明确地提供的话，它仅仅可以访问回调合约的状态变量(否则，它将没有方式去命名它们)。库函数如果它们没有更改状态(列如，如果它们是 `view` 或`pure`函数)，库函数可以仅仅直接被回调(例如，不使用`DELEGATECALL`)，因为假定库是无状态的。特别是，不可能销毁库。

**Note**

Until version 0.4.20, it was possible to destroy libraries by circumventing Solidity’s type system. Starting from that version, libraries contain a `mechanism(https://solidity.readthedocs.io/en/v0.5.0/contracts.html?highlight=view#call-protection)` that disallows state-modifying functions to be called directly (i.e. without`DELEGATECALL`).

在版本0.4.20之前，可以通过规避Solidity的类型系统来销毁库。从该版本开始，库包含一种机制，该机制不允许直接调用状态修改函数（即没有DELEGATECALL）。

Libraries can be seen as implicit base contracts of the contracts that use them. They will not be explicitly visible in the inheritance hierarchy, but calls to library functions look just like calls to functions of explicit base contracts (`L.f() ` `if `L is the name of the library). Furthermore, `internal` functions of libraries are visible in all contracts, just as if the library were a base contract. Of course, calls to internal functions use the internal calling convention, which means that all internal types can be passed and types `stored in memory` will be passed by reference and not copied. To realize this in the EVM, code of internal library functions and all functions called from therein will at compile time be pulled into the calling contract, and a regular `JUMP` call will be used instead of a `DELEGATECALL`.

库可以被视为使用它们的合约的隐含基础合约。它们不会在继承层次结构中明确可见，但是对库函数的调用看起来就像调用显式基本合约的函数一样。此外，`internal`库函数在所有合约中都是可见的，就好像库是基本合约一样，当然了，对内部函数的调用使用内部调用约定，这意味着可以传递所有内部类型，并且存储在内存中的类型将通过引用传递而不是复制。在EVM中实现这一点，内部库函数的代码和从其中调用的所有函数将在编译时被拉入调用合约，并且将使用常规JUMP调用而不是DELEGATECALL。

The following example illustrates how to use libraries (but manual method be sure to check out using for for a more advanced example to implement a set).

```solidity
pragma solidity >=0.4.22 <0.6.0;

library Set {
  // We define a new struct datatype that will be used to
  // hold its data in the calling contract.
  struct Data { mapping(uint => bool) flags; }

  // Note that the first parameter is of type "storage
  // reference" and thus only its storage address and not
  // its contents is passed as part of the call.  This is a
  // special feature of library functions.  It is idiomatic
  // to call the first parameter `self`, if the function can
  // be seen as a method of that object.
  function insert(Data storage self, uint value)
      public
      returns (bool)
  {
      if (self.flags[value])
          return false; // already there
      self.flags[value] = true;
      return true;
  }

  function remove(Data storage self, uint value)
      public
      returns (bool)
  {
      if (!self.flags[value])
          return false; // not there
      self.flags[value] = false;
      return true;
  }

  function contains(Data storage self, uint value)
      public
      view
      returns (bool)
  {
      return self.flags[value];
  }
}

contract C {
    Set.Data knownValues;

    function register(uint value) public {
        // The library functions can be called without a
        // specific instance of the library, since the
        // "instance" will be the current contract.
        require(Set.insert(knownValues, value));
    }
    // In this contract, we can also directly access knownValues.flags, if we want.
}
```

![](_v_images/20190531224413147_1447150645.png)

```solidity
pragma solidity >=0.4.16 <0.6.0;

library BigInt {
    struct bigint {
        uint[] limbs;
    }

    function fromUint(uint x) internal pure returns (bigint memory r) {
        r.limbs = new uint[](1);
        r.limbs[0] = x;
    }

    function add(bigint memory _a, bigint memory _b) internal pure returns (bigint memory r) {
        r.limbs = new uint[](max(_a.limbs.length, _b.limbs.length));
        uint carry = 0;
        for (uint i = 0; i < r.limbs.length; ++i) {
            uint a = limb(_a, i);
            uint b = limb(_b, i);
            r.limbs[i] = a + b + carry;
            if (a + b < a || (a + b == uint(-1) && carry > 0))
                carry = 1;
            else
                carry = 0;
        }
        if (carry > 0) {
            // too bad, we have to add a limb
            uint[] memory newLimbs = new uint[](r.limbs.length + 1);
            uint i;
            for (i = 0; i < r.limbs.length; ++i)
                newLimbs[i] = r.limbs[i];
            newLimbs[i] = carry;
            r.limbs = newLimbs;
        }
    }

    function limb(bigint memory _a, uint _limb) internal pure returns (uint) {
        return _limb < _a.limbs.length ? _a.limbs[_limb] : 0;
    }

    function max(uint a, uint b) private pure returns (uint) {
        return a > b ? a : b;
    }
}

contract C {
    using BigInt for BigInt.bigint;

    function f() public pure {
        BigInt.bigint memory x = BigInt.fromUint(7);
        BigInt.bigint memory y = BigInt.fromUint(uint(-1));
        BigInt.bigint memory z = x.add(y);
        assert(z.limb(1) > 0);
    }
}
```

![](_v_images/20190531224512604_986358927.png)

##### Call Protection For Libraries

![](_v_images/20190531224128849_1994959721.png)

#### Using For

![](_v_images/20190531224208817_965273888.png)

```solidity
pragma solidity >=0.4.16 <0.6.0;

// This is the same code as before, just without comments
library Set {
  struct Data { mapping(uint => bool) flags; }

  function insert(Data storage self, uint value)
      public
      returns (bool)
  {
      if (self.flags[value])
        return false; // already there
      self.flags[value] = true;
      return true;
  }

  function remove(Data storage self, uint value)
      public
      returns (bool)
  {
      if (!self.flags[value])
          return false; // not there
      self.flags[value] = false;
      return true;
  }

  function contains(Data storage self, uint value)
      public
      view
      returns (bool)
  {
      return self.flags[value];
  }
}

contract C {
    using Set for Set.Data; // this is the crucial change
    Set.Data knownValues;

    function register(uint value) public {
        // Here, all variables of type Set.Data have
        // corresponding member functions.
        // The following function call is identical to
        // `Set.insert(knownValues, value)`
        require(knownValues.insert(value));
    }
}
```

It is also possible to extend elementary types in that way:

```solidity
pragma solidity >=0.4.16 <0.6.0;

library Search {
    function indexOf(uint[] storage self, uint value)
        public
        view
        returns (uint)
    {
        for (uint i = 0; i < self.length; i++)
            if (self[i] == value) return i;
        return uint(-1);
    }
}

contract C {
    using Search for uint[];
    uint[] data;

    function append(uint value) public {
        data.push(value);
    }

    function replace(uint _old, uint _new) public {
        // This performs the library function call
        uint index = data.indexOf(_old);
        if (index == uint(-1))
            data.push(_new);
        else
            data[index] = _new;
    }
}
```

![](_v_images/20190531224339793_162300491.png)

- 注意：所有的库函数调用是真实的EVM函数调用。 这就意味着如果你传递内存类型或值类型，一个副本(copy)将会被执行，也可以传递`self`变量， 唯一的不同是，当使用`storage`引用变量时，没有副本将会被执行。
- 因为 `storage`类型的变量是 引用类型。