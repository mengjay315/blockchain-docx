### Plotting

##### Inputs
accountId: sha256(pubkey).long()  
nonce  

##### Calculating 8092 Hashes
Hash #8191: shabal256(accountId, nonce)  
Hash #8190: shabal256(Hash #8191, accountId, nonce)  
Hash #8189: shabal256(Hash #8190, Hash #8191, accountId, nonce), Hash data should not be more than 4096 bytes, so after iteration 128  
Hash #i: shabal256(last 128 hashes), until i=0  

##### Final Hash
Final Hash: shabal256(Hash#0-8191, accountId, nonce)  

##### XOR
Hash #i: Hash #i xor Final Hash  
Scoop #i: `Hash #(i*2)`, `Hash #(i*2 + 1)`  

##### Shuffling
shuffle hash data for poc2  
Scoop #i: `Hash #(i*2)`, `Hash #(8191 - i*2)`  

##### Plot file
Nonce #0: Scoop #0, ..., Scoop #4095  
Nonce #n: Scoop #0, ..., Scoop #4095  

##### Optimized plot file
Scoop #0: Scoop #0 nonce #0, ..., Scoop #0 nonce #n  
Scoop #i: Scoop #i nonce #0, ..., Scoop #i nonce #n  

### Mining  
- initPubkey: 0  
- initGenSig: 0  
- initBaseTarget: 18325193796  
- genId: sha256(pubkey).long()  
- genSig: shabal256(lastGenSig, lastGenId)  
- scoopIndex: shabal256(genSig, height).mod(4096)  
- target: shabal256(genSig, scoop[scoopIndex]) >> 24*8  
- targetTimeSpan: 24(blocks) * 4(minutes) * 60
- baseTarget: averageBaseTarget * actualTimeSpan / targetTimeSpan  
- deadline: target / baseTarget  

### Initial Base Target
基本原理是，从每个nonce中取出一个scoop，一直到取出的scoop对应的target是期望的target(足够小)，一个nonce对应一个target，经过一系列计算后的target是均匀的随机的，这是一个取多个独立同分布的均匀随机变量的最小值的问题  
##### n个独立同分布均匀随机变量的最小值的期望  
```math
E(X) = (b + n*a) / (n + 1), X ∈ (a, b)
```
X: 随机变量, 在burstcoin中就是target  
n: 随机变量的个数, 也是nonce的个数  
a: 0  
b: 由于target是8个字节, 所以是2^64-1  
E(X): 随机变量最小值的期望, 也就是我们期望的target  
所以公式可以转换为
```math
E(X) = (2^64 - 1) / (n + 1)
```
n对应着硬盘空间, 通过调节E(X)就可以应对硬盘空间大小的涨跌, 在burstcoin中, E(X)被拆成了baseTarget和deadline, 所以公式可以进一步转换为
```math
deadline * baseTarget = (2^64 - 1) / (n + 1)  
```
deadline是个固定数字, 因此只要调整baseTarget就能调整难度  
```math
baseTarget = (2^64-1) / ((n+1) * deadline)  
```
一个nonce包含4096个scoop, 所以n个nonce对应的硬盘空间是
```math
s = n*4096*64  
```
所以baseTarget与硬盘空间的对应关系是
```math
baseTarget = (2^64 - 1) / ((s / (4096*64) + 1) * deadline)
```
假设初始硬盘空间是1TiB, deadline是240, 那么
```math
initialBaseTarget = (2^64 - 1) / ((1024*1024*1024*1024 / (4096*64) + 1) * 240) = 18325189427
```
与burstcoin的initialBaseTarget略有区别, burstcoin应该是把公式简化为了
```math
initialBaseTarget = 2^64 / (1024*1024*1024*1024 / (4096*64) * 240) = 18325193796
```

### 难度
baseTarget越小, 需要的硬盘空间越大, 代表难度越大, 假设初始难度为1, 则
```math
difficulty = initialBaseTarget / baseTarget
```

### 网络容量
由上面的公式可知, 网络中的硬盘总容量为
```math
s = ((2^64 - 1) / (baseTarget*deadline) - 1) * 4096 * 64
```
或者
```math
s = initialBaseTarget / baseTarget * 1 TiB
```

