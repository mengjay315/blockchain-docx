# ERC-20标准

ERC全称是**EthereumRequest for Comments**，即以**太坊开发者提交的协议提案**，后面的数字是提案编号。那么ERC20就是以太坊开发者提交的第20个协议提案。

#### transferFrom

Transfers `_value` amount of tokens from address `_from` to address `_to`, and MUST fire the `Transfer` event.

The `transferFrom` method is used for a withdraw workflow, allowing contracts to transfer tokens on your behalf.
This can be used for example to allow a contract to transfer tokens on your behalf and/or to charge fees in sub-currencies.
The function SHOULD `throw` unless the `_from` account has deliberately authorized the sender of the message via some mechanism.

*Note* Transfers of 0 values MUST be treated as normal transfers and fire the `Transfer` event.

``` js
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
```

##### translate-transferFrom

Transfers`_value`数额的tokens从地址`_from`到地址`_to`,必须触发`Transfer`事件。

`transferFrom`方法被用来一个提现的工作模式，允许合约为你转移tokens.

这可以用于允许合约为你转让代币或收取费用.

除了`_from`账户通过某些机制已经慎重地授权信息的发送者之外，否则这个方法将会抛出异常，

**注意**转移0 values也必须被认为是正常转移，并且触发`Transfer`事件。



#### approve

Allows `_spender` to withdraw from your account multiple times, up to the `_value` amount. If this function is called again it overwrites the current allowance with `_value`.

**NOTE**: To prevent attack vectors like the one [described here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/) and discussed [here](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729),
clients SHOULD make sure to create user interfaces in such a way that they set the allowance first to `0` before setting it to another value for the same spender.
THOUGH The contract itself shouldn't enforce it, to allow backwards compatibility with contracts deployed before

``` js
function approve(address _spender, uint256 _value) public returns (bool success)
```

##### translate-approve

允许"_spender"从你的账户里多次提现，直到”_value"这个数额。如果这个函数再次被调用，它将会使用“_value"覆写当前的限额。

**注意**：为了避免向量(vectors)攻击，客户端应该确保以这样一种方式创建用户接口，即在为同一个spender设置另一个值之前，首先设置限额(allowance)为0.

尽管合约本身不会强制执行，合约允许之前部署的合约向后兼容。

**向量攻击的讨论**

https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/

https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729

### github上开源的项目：关于ERC20，ERC721...

https://github.com/OpenZeppelin/openzeppelin-solidity

![](_v_images/20190527162236874_597137673.png)

https://openzeppelin.org/

![](_v_images/20190527162529267_1385062959.png)