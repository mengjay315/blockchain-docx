# solidity测试脚本

## 写测试case时遇到的问题：
### 1. BigNumber转换问题
在DMAPlatform.test.js文件中，原来的测试代码如下：

```solidity
contract('dma/Platform', (accounts) => {
  let nftoken;
  let token;
  let platform;
  let allowedAccount;
  const id1 = 1;
  const id2 = 2;
  const id3 = 3;
  const id4 = 40;
  const seller = accounts[0];
  const buyer = accounts[1];
  const tokenTotalSupply = new web3.BigNumber('3e+26'); //
  const ownerSupply = new web3.BigNumber('3e+26');

  // To send the right amount of tokens, taking in account number of decimals.
  const decimalsMul = new web3.BigNumber('1e+18');

  //每个测试之前都会先执行这个
  beforeEach(async () => {

    const tonkenAmount = decimalsMul.mul(200);  //
    //ERC721
    nftoken = await NFTokenDMA.new('Foo', 'F');
    //ERC20
    token = await TokenDMA.new();
    //给买家 200
    await token.transfer(buyer, tonkenAmount);

    //绑定
    platform = await DMAPlatform.new(nftoken.address, token.address);
  });

```

`const tokenTotalSupply = new web3.BigNumber('3e+26')`;  //这是原来的写法，把'3e+26'转换成BigNumber

我进行 truffle test时，报错，说`web3.BigNumber`不是一个构造函数，我以为是js文件中没有引入web3，我`npm install web3 --save`, 然后在js文件中引入web3模块，再测试发现提示错误：`BigNumber`不是一个函数。

![](_v_images/20190608161005753_2095420104.png)

我先是查了web3.js 官方文档，以下是有关BigNumber的

https://web3js.readthedocs.io/en/1.0/web3-utils.html?highlight=BigNumber

![](_v_images/20190608161642743_676263854.png)

我又查了 The Ethereum Javascript API， 找到了问题原因：

https://ethereumbuilders.gitbooks.io/guide/content/en/ethereum_javascript_api.html?tdsourcetag=s_pcqq_aiomsg

![](_v_images/20190608161902550_862630732.png)

说的是`web3.js`依赖于 `BigBumber Library`,并且自动添加到js文件中，只有有了这个库，才能这样写：

`var balance = new BigNumber('131242344353464564564574574567456');`

- **BigBumber Library**
- https://github.com/MikeMcl/bignumber.js/

用于任意精度十进制和非十进制算术的JavaScript库。

![](_v_images/20190608162532164_258432286.png)


**Load**

The library is the single JavaScript file bignumber.js (or minified, bignumber.min.js).

- Browser:

<script src='path/to/bignumber.js'></script>

- Node.js:

- $ npm install bignumber.js
-const BigNumber = require('bignumber.js');

- ES6 module:

- import BigNumber from "./bignumber.mjs"

AMD loader libraries such as requireJS:

```js
require(['bignumber'], function(BigNumber) {
    // Use BigNumber here in local scope. No global BigNumber.
});
```
![](_v_images/20190608163025955_430126095.png)

知道问题后，更改了代码：

![](_v_images/20190608163122723_284971358.png)
![](_v_images/20190608163138634_447392911.png)


## 2. decimal 问题

```js
const decimalsMul = new BigNumber('1e+18');
const tonkenAmount = decimalsMul.mul(200);
```

truffle test时，提示错误：`decimalsMul.mul`不是一个函数，其实就是说 `mul`不是一个函数，

![](_v_images/20190608170243638_1705852389.png)

搜索后找到了 Decimal，https://www.npmjs.com/package/decimal

![](_v_images/20190608170601097_1115331782.png)

![](_v_images/20190608170649148_1610573000.png)

知道原因后，做了如下更改： npm install decimal --save, 然后引入 decimal

![](_v_images/20190608170736691_1855414754.png)

![](_v_images/20190608170908881_547728633.png)

原来我是没有加 `.toString()`的，再测试出错了，

![](_v_images/20190608171312822_762433612.png)

问题说明：问题出现在beforeEach方法里：
- const tonkenAmount = Decimal(decimalsMul).mul(200)
- await token.transfer(buyer, tonkenAmount);

![](_v_images/20190608171421055_367396716.png)

- transfer的第二个参数是uint256, 报的错就是说，传入transfer的第二个参数是无效的，和原来的uint256类型完全不相等
- console.log(tonkenAmount); 输出tonkenAmount的值，看看是什么，发现不是uint256类型

![](_v_images/20190608171432746_1367871614.png)

- await token.transfer(buyer, tonkenAmount.internal);  
- 我先是这样改，发现没有错误，
- 然后我又这样改，测试也没问题：

![](_v_images/20190608171455874_1923059254.png)

- 原来的写法是下面这样的，decimalMul.mul(200) 的结果就是表示一个数值
- 我开始也这样写，test出错，web3.BigNumber 不是一个函数，所以查资料后，改成了上面的那种，
- 这个改好后，再test,发现错误 decimalsMul.mul 不是一个函数， 再查资料，可以看上面的，就改成了const tonkenAmount = Decimal(decimalsMul).mul(200).internal，
- 测试可以通过


 - Decimal(decimalsMul).mul(200).toNumber() 
 - 我把internal去掉，改成了 `.toNumber()`,   console.log(tokenAmount)也能得到结果，但是还是出错在transfer函数里的第二个参数那里

![](_v_images/20190608171555554_429617156.png)

![](_v_images/20190608171620340_1502319006.png)


![](_v_images/20190608171636301_1229177243.png)
![](_v_images/20190608171646726_562206648.png)


是number类型，不是uint256类型

![](_v_images/20190608171659374_730059586.png)


![](_v_images/20190608171715378_1715820039.png)

这样也可以

![](_v_images/20190608171729680_850775750.png)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#### 总结：
在需要nodejs与solidity有数据交互的时候，**尽量避免使用toNumber**，**打日志的时候使用toString**，js传参数给solidity的时候直接使用BigNumber，假如要在js中进行算数计算，应直接使用BigNumber的相关方法诸如plus进行算数计算。



