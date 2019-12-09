# Creating Contracts via `new`

A contract can create other contracts using the `new` keyword. The full code of the contract being created has to be known when the creating contract is compiled so recursive creation-dependencies are not possible.

```solidity
pragma solidity >0.4.99 <0.6.0;

contract D {
    uint public x;
    constructor(uint a) public payable {
        x = a;
    }
}

contract C {
    D d = new D(4); // will be executed as part of C's constructor

    function createD(uint arg) public {
        D newD = new D(arg);
        newD.x();
    }

    function createAndEndowD(uint arg, uint amount) public payable {
        // Send ether along with the creation
        D newD = (new D).value(amount)(arg);
        newD.x();
    }
}

```

As seen in the example, it is possible to send Ether while creating an instance of D using the `.value() `option, but it is not possible to limit the amount of gas. If the creation fails (due to out-of-stack, not enough balance or other problems), an exception is thrown.

`D d = new D(4); // will be executed as part of C's constructor`

![](_v_images/20190608222232890_1204902454.png)

**通过 new()，创建合约时，合约中的构造函数将会被执行**

![](_v_images/20190608222434262_615889759.png)
![](_v_images/20190608222502359_956243504.png)

![](_v_images/20190608222538599_950559078.png)
![](_v_images/20190608222611746_1228161754.png)

