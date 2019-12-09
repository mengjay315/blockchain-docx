# fabric-sdk-go 应用
### 2019年2月14日
对于fabric-sdk-go的应用,我之前有过记录,出过很多错误, 在**区块链学习.docx**中有相应的记录, 我现在清理了之前安装的工具,镜像, 重新开始操作一遍, 
## 以下操作终端都是开启了代理的
还是出现了很多错误,试了好多次才没再出错, git clone出错的最多, 很是郁闷, 一度怀疑是电脑有问题, git有问题, 把git完全卸载又重装, 结果还是那些问题, 又试了好多次才好的,

## 可能的原因是代理的网络不稳定, 导致git clone 出错

#### make depend 出错
![错误](_v_images/20190214165448242_1299399958.png)

![错误](_v_images/20190214165709828_1274402635.png)
![接上](_v_images/20190214165740447_1091449841.png)

#### make depend试了很多次,只有这次才是正确的:
![make depend 正确](_v_images/20190214165919841_521550458.png)
![接上](_v_images/20190214170057616_957129389.png)
![接上](_v_images/20190214170128687_1238181803.png)


#### make populate 出错
![make-populate](_v_images/20190214145102904_601970988.png)

##### make populate正确的结果
![make populate正确](_v_images/20190214171432071_1371873535.png)


![接上](_v_images/20190214171513600_1901787521.png)
![接上](_v_images/20190214171633733_129286915.png)
![接上](_v_images/20190214171714909_1116254203.png)
## 
## fabric-sdk-go demo
    上面关于 fabric-sdk-go 的环境配置好后, 在fabric-sdk-go 目录下创建的 测试demo, 包括了sdk的初始化, channel的创建, chaincoid的install 和 instantiat, chaincoid 的调用, 交易.
    步骤:
    (1) 模仿fabric-sdk-go 结构,创建项目目录,
    先创建work目录, 然后创建fixtures目录, 在下面创建dockerenv 和fabric 目录,把fabric-sdk-go/test/fixtures 目录下的dockerenv 和fabric 下的文件拷贝过来,dockerenv只拷贝以下三个文件, fabric目录拷贝 v1 和 v1.4
![work目录结构](_v_images/20190216182226070_319776940.png)

    再创建client目录,里面有html文件,和一个脚本文件
![client](_v_images/20190216183217756_1476231072.png)

    脚本文件: api.sh
![api.sh脚本文件](_v_images/20190219170756694_1801445146.png)

**这个脚本文件的作用**

    作为后端开发者, 可能不熟悉前端怎么与后台交互, 毕竟项目都有明确的分工, 后台开发只写后台代码, 测试后台代码时, 可以通过写一个shell脚本 实现后端代码的调用.
    这也是接口测试常用的方法.

** 进入work目录,执行mys,sh脚本**

    该脚本主要是开启fabric的docker环境.
![1](_v_images/20190216163857098_1672861193.png)

**相应的容器都已经起来了,说明docker环境没问题:**

    在Goland中,在work目录下, go run main.go,运行main.go文件是为了测试 SDK是否初始化,以及链码的一系列操作是否可以.出现了以下问题:
    
```
come to main
panic: failed to initialize channel: checking for joined targets failed: failed while checking if primary peer has already joined channel: failed to query channel for peer: cscc.GetChannels failed: SendProposal failed: Transaction processing for endorser [peer0.org1.example.com:7051]: Endorser Client Status Code: (2) CONNECTION_FAILED. Description: dialing connection timed out [peer0.org1.example.com:7051]

goroutine 1 [running]:
github.com/hyperledger/fabric-sdk-go/test/integration/util/runner.(*Runner).Initialize(0xc0004b3ee0)
	/home/gaojie/gopath/src/github.com/hyperledger/fabric-sdk-go/test/integration/util/runner/runner.go:114 +0x419
main.main()
	/home/gaojie/gopath/src/github.com/hyperledger/fabric-sdk-go/work/main.go:143 +0xf6

Process finished with exit code 2

```
    看了错误大概是 initialize channel, 我尝试了好几遍,还是同样的错误, 查了很多资料也找到明确的解决方法.我也对代码进行了debug, 找到了具体出错开始的地方, 但是我还是不明白怎么去解决.
    直到今天上课(2019年2月16日), 在分析这个项目的代码时, 说到了一处关于hosts文件的更改, 再看上面的错误, 最后是说连接超时, 连接失败,更改了hosts文件就可以,

**具体解释我还没弄明白:**

    org1peer1:
![](_v_images/20190220102525497_187530727.png)

由于域名没有指定和指定的IP地址映射,    我们是在本地环境下, 所以要在hosts文件里写上, 主机名和域名的映射, 这样端口之间才可以通信

**测试以下三个能否ping通**
```
peer0.org1.example.com
peer1.org1.example.com
 orderer.example.com
```
**出现错误,不能ping通**

### hosts文件更改
**更改了hosts文件如下,改了之后再运行main.go,就可以了**
![更改hosts文件](_v_images/20190216185932959_831361553.png)

**运行完 main.go文件后, 结果如下, 成功了**
![4](_v_images/20190216164018134_1679530678.png)

**再次查看镜像及容器:**

     生成了一个关于chaincode 的镜像, 还有一个容器, 该容器是chaincode执行调用的环境.
![2](_v_images/20190216163926416_1864961847.png)

至此说明SDK和 节点是通的
![](_v_images/20190219222930967_1482369459.png)

    接下来实现前端和 SDK 之间的通信
    为了实现前端和 SDK 的通信, 在main.go文件里创建RESTful接口,
![](_v_images/20190219223526757_1337053460.png)

    buildRouter(), 创建路由
![](_v_images/20190219223700174_1313585853.png)
    
    getvalue(), 获取前端 POST请求的内容, 并把内容反序列化成 bcInfo结构体, 
![](_v_images/20190219224018057_479194273.png)
    
    进入client目录,执行 ./api.sh, 进行post请求测试,结果如下:
 ![](_v_images/20190219224510595_834008565.png)

![](_v_images/20190219224526770_470576282.png)

    有结果说明前端和 SDK 是通的, 接着在终端 打开 demo1.html, 在页面上进行测试:
![](_v_images/20190219224959810_893721208.png)    
![](_v_images/20190219225020441_404456838.png)
![](_v_images/20190219225038317_1128653347.png)
![](_v_images/20190219225054414_1334176860.png)

    也能出现结果, 就一个地方不对, 反序列化后的结构体, Age是0, 我明明在页面上写的是23, 我反复试了几次, 获得的Age都是0,
![](_v_images/20190219225503118_1507791415.png)

![](_v_images/20190219225553139_809709072.png)

    demo1.html 关键代码:
![](_v_images/20190219225749571_924965339.png)    

    POST请求提交数据采用 ajax 的方式, 是异步的, 因为把数据提交到区块链上, 往往是需要时间的, 所以在设计的时候, 要考虑用异步的方式, 不会造成阻塞,
![](_v_images/20190219225038317_1128653347.png)

    根据控制台的结果可以看到, 先是出现 come to end, 由于 ajax 是异步的, 先执行最后的命令, 不会因为数据上链的过程而造成阻塞,

    再次查看镜像和容器, 多了一个chaincode的镜像和容器:
![](_v_images/20190219231317916_816693708.png)
![](_v_images/20190219231350234_1388499848.png)

**docker logs e448ef6858cd**, 查看chaincode容器的日志, 该容器为 chaincode 执行的容器, 
![](_v_images/20190219232019856_1013394976.png)

    通过docker logs命令可以查看容器的日志
![](_v_images/20190219232533051_89197287.png)

##### 该 chaincode 使用的是 example chaincode:
![](_v_images/20190221130815812_84740476.png)


### SDK
![](_v_images/20190222115218723_1416087855.png)

    chaincode容器是fabric镜像和SDK的桥梁
**SDK跟chaincode不在同一个环境执行**

1. SDK产生chaincode需要的执行环境 => docker container
2. SDK把chaincode代码发送到该容器中
3. 容器环境中加载, 初始化该chaincode
4. SDK调用chaincode


### chaincode
    Chaincode是一段由Go语言编写（支持其他编程语言，如Java），并能实现预定义接口的程序。Chaincode运行在一个受保护的Docker容器当中，与背书节点的运行互相隔离。Chaincode可通过应用提交的交易对账本状态初始化并进行管理。
![](_v_images/20190219231350234_1388499848.png) 

![](_v_images/20190222120057653_1449963254.png)

![](_v_images/20190222120210202_633158391.png)

![](_v_images/20190222120240098_1141381788.png)

![](_v_images/20190222120305731_1852014169.png)

#### 交易的生命周期和账本的交互
![](_v_images/20190222120332086_1086751700.png)

![](_v_images/20190222120546990_1281578533.png)

#### Chaincode API
    每个chaincode程序都必须实现 chiancode接口 ，接口中的方法会在响应传来的交易时被调用。特别地，Init（初始化）方法会在chaincode接收到instantiate（实例化）或者upgrade(升级)交易时被调用，进而使得chaincode顺利执行必要的初始化操作，包括初始化应用的状态；Invoke（调用）方法会在响应invoke（调用）交易时被调用以执行交易。
    其他chaincode shim APIs中的接口被称为chaincode存根接口( ChaincodeStubInterface)，用于访问及修改账本，并实现chaincode之间的互相调用。
![](_v_images/20190222121216181_2034716805.png)                                               
![](_v_images/20190222121248616_1235489682.png)

##### 官网关于chaincode的讲解
https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html

**chaincode API**
**Every chaincode program must implement the Chaincode interface:**
https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#chaincode-api

**The other interface in the chaincode “shim” APIs is the ChaincodeStubInterface:**
https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStubInterface

**chaincode的包**
"github.com/hyperledger/fabric/core/chaincode/shim"

![](_v_images/20190222122729957_1216329766.png)

![](_v_images/20190222122952036_869362075.png)

![](_v_images/20190222123206512_2053614898.png)
![](_v_images/20190222123249655_1101850701.png)


##### chaincode必须要实现的接口
![](_v_images/20190222121416989_1440185562.png)

##### 初始化Chaincode
    Init函数初始化Chaincode。值得留意的是chaincode升级同样会调用该函数。当我们编写的chaincode会升级现有chaincode时，需要确保适当修正Init函数。特别地，如果没有“迁移”操作或其他需要在升级中初始化的东西，那么就提供一个空的“Init”方法。
![](_v_images/20190222121558588_470017933.png)

##### 调用Chaincode
    我们需要调用ChaincodeStubInterface来获取参数。Invoke函数所需的传入参数正是应用想要调用的chaincode的名称。
![](_v_images/20190222121722255_1217871441.png)

##### 整合全部代码

    终于到了写main函数的最后关头，它将调用shim.Start函数。下面是包含整个chaincode程序的代码：
![](_v_images/20190222121816234_31416036.png)

    chaincode的main函数是一样的.
    
##### 与chaincode交互
    chaincode开发完成后, 有两种方式可以与链码交互:通过SDK或通过CLI命令行,
    通过CLI命令行的方式,命令比较多,繁琐, 容易出错.
    通过SDK的方式,简单,
    
一般模式下chaincode开发调试过程(使用CLI与命令行交互)
![](_v_images/20190222123432943_786061099.png)
![](_v_images/20190222123945606_1963040264.png)
