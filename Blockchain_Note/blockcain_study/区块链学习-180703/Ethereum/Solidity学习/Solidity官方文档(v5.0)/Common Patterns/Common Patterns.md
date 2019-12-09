# Common Patterns(共同的模式)

## Withdrawal from Contracts(从合约中提取钱)

The recommended method of sending funds after an effect is using the withdrawal pattern. Although the most intuitive method of sending Ether, as a result of an effect, is a direct `transfer`call, this is not recommended as it introduces a potential security risk. You may read more about this on the [Security Considerations](https://solidity.readthedocs.io/en/v0.5.0/security-considerations.html#security-considerations) page.

在一个作用后，**使用提款模式是发送资金的推荐方法**。虽然最直观的发送以太币作为作用的结果方法是直接`transfer`调用，这是不推荐的，因为它引入了一个潜在的安全风险。你可以在这个页面了解更多`Security Considerations`。

The following is an example of the withdrawal pattern in practice in a contract where the goal is to send the most money to the contract in order to become the “richest”, inspired by [King of the Ether](https://www.kingoftheether.com/).

下面的一个例子是提取模式在一个合约上的实践，这个合约的目的是发送大部分的以太币到合约中，为了成为“最富有的”，受到了了`king of ther Ether`的启发。

In the following contract, if you are usurped as the richest, you will receive the funds of the person who has gone on to become the new richest.



```javascript
pragma solidity >0.4.99 <0.6.0;

contract WithdrawalContract {
    address public richest;
    uint public mostSent;

    mapping (address => uint) pendingWithdrawals;

    constructor() public payable {
        richest = msg.sender;
        mostSent = msg.value;
    }

    function becomeRichest() public payable returns (bool) {
        if (msg.value > mostSent) {
            pendingWithdrawals[richest] += msg.value;
            richest = msg.sender;
            mostSent = msg.value;
            return true;
        } else {
            return false;
        }
    }

    function withdraw() public {
        uint amount = pendingWithdrawals[msg.sender];
        // Remember to zero the pending refund before
        // sending to prevent re-entrancy attacks
        pendingWithdrawals[msg.sender] = 0;
        msg.sender.transfer(amount);
    }
}
```

// **Remember to zero the pending refund before sending to prevent re-entrancy attacks**

**在发送之前记得把待发送的资金置0，以避免重入攻击**。

This is as opposed to the more intuitive sending pattern:

这与更直观的发送方式相反：

```javascript
pragma solidity >0.4.99 <0.6.0;

contract SendContract {
    address payable public richest;
    uint public mostSent;

    constructor() public payable {
        richest = msg.sender;
        mostSent = msg.value;
    }

    function becomeRichest() public payable returns (bool) {
        if (msg.value > mostSent) {
            // This line can cause problems (explained below).
            richest.transfer(msg.value);
            richest = msg.sender;
            mostSent = msg.value;
            return true;
        } else {
            return false;
        }
    }
}
```

Notice that, in this example, an attacker could trap the contract into an unusable state by causing `richest` to be the address of a contract that has a fallback function which fails (e.g. by using `revert()` or by just consuming more than the 2300 gas stipend transferred to them). That way, whenever `transfer` is called to deliver funds to the “poisoned” contract, it will fail and thus also `becomeRichest` will fail, with the contract being stuck forever.

**注意**，在这个例子中，一个攻击者通过导致`richest`成为一个拥有`fallback function`的合约地址使合约陷入了一个不可使用的状态，会失败的，(比如，可以使用`revert()`)，或者花费超过2300的gas转给它们。那种方式，无论何时调用`transfer`发送资金到“坏的”地址，将会失败的，`becomeRichest`也会失败，**合约将永远被卡主**。

In contrast, if you use the “withdraw” pattern from the first example, the attacker can only cause his or her own withdraw to fail and not the rest of the contract’s workings.

**相比之下，如果你使用“withdraw”模式，攻击者仅仅可以造成他或她的提取失败，不会影响合约的其他运作。**

## Restricting Access(限制访问)

Restricting access is a common pattern for contracts. Note that you can never restrict any human or computer from reading the content of your transactions or your contract’s state. You can make it a bit harder by using encryption, but if your contract is supposed to read the data, so will everyone else.



You can restrict read access to your contract’s state by **other contracts**. That is actually the default unless you declare make your state variables `public`.

Furthermore, you can restrict who can make modifications to your contract’s state or call your contract’s functions and this is what this section is about.

The use of **function modifiers** makes these restrictions highly readable.





## State Machine

Contracts often act as a state machine, which means that they have certain **stages** in which they behave differently or in which different functions can be called. A function call often ends a stage and transitions the contract into the next stage (especially if the contract models **interaction**). It is also common that some stages are automatically reached at a certain point in **time**.



An example for this is a blind auction contract which starts in the stage “accepting blinded bids”, then transitions to “revealing bids” which is ended by “determine auction outcome”.



Function modifiers can be used in this situation to model the states and guard against incorrect usage of the contract.

在这种情况下，可以使用函数修饰符来模拟状态并防止合约的错误使用。

### Example

In the following example, the modifier `atStage` ensures that the function can only be called at a certain stage.



Automatic timed transitions are handled by the modifier `timeTransitions`, which should be used for all functions.

**Note**

**Modifier Order Matters**. If atStage is combined with timedTransitions, make sure that you mention it after the latter, so that the new stage is taken into account.

**修饰符排序事项**。如果atStage和timedTransitions结合使用，确保你在后者之后提及它，以便考虑新的阶段。

Finally, the modifier `transitionNext` can be used to automatically go to the next stage when the function finishes.

最后，当函数完成后，修饰符`transitionNext`可以被自动使用到下一个阶段。

**Note**

**Modifier May be Skipped**. This only applies to Solidity before version 0.4.0: Since modifiers are applied by simply replacing code and not by using a function call, the code in the transitionNext modifier can be skipped if the function itself uses return. If you want to do that, make sure to call nextStage manually from those functions. Starting with version 0.4.0, modifier code will run even if the function explicitly returns.

**修饰符可能被跳过**。这个只在0.4.0之前应用：因为修饰符通过简单地的替代代码被应用，而不是使用一个函数调用，如果函数本身使用了返回，在transitionNext修饰符中的代码可以被跳过，如果你想那样做，确保从那些函数里手动地调用nestStage.从0.4.0版本开始，修饰符代码将会运行，即使函数有明确的返回。

