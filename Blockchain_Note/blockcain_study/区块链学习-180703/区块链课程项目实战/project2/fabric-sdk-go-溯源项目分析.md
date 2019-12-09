# fabric-sdk-go-项目
**步骤**

1.  创建vendor

         新建项目目录，把原来fabric-sdk-go下的vendor目录copy过来

2. 在vendor下封装SDK

        vendor目录下fabric接口相关的类库，
![](_v_images/20190308171752740_628751297.png)

        mkdir vendor/github.com/hyperledger
        把fabric-sdk-go/internal/github.com/hyperledger/* >> vendor/github.com/hyperledger
 ![](_v_images/20190308172027907_1119678686.png)

        mkdir vendor/github.com/hyperledger/fabric-sdk-go
        把fabric-sdk-go下的 pkg和third_party 拷贝到 vendor/github.com/hyperledger/fabric-sdk-go
        完整的hyperledger
![](_v_images/20190308172607097_1062052703.png)

3. 测试代码
![](_v_images/20190308172747874_891055798.png)

4. 根据之前对测试代码的分析， 把相应的类库文件copy过来
![](_v_images/20190308173115684_471906324.png)

        在fabric-sdk-go目录中， 这几个文件都在 intergration包下， 现在我把这几个文件copy到我的项目下， 由于文件不是很多，因此，不再把这些文件封装到对应的包下， 直接都放在main 包下，因此在这几个文件中要把 包名改为 main。
5. 修改各个文件的错误

        由于包名修改为了main， 在各个文件都会出现错误，根据具体的错误修改。具体的可参考项目2的讲解。
        执行代码时一定是 go build prj2, 直接go build main.go会出错
 ![](_v_images/20190308174015604_1306731649.png)

6. 错误基本修正，开启网络

        当go build prj2时出现如下错误，说明项目各个文件没有打的问题了，出现这个错误是因为没开启docker 服务
![](_v_images/20190308174140829_1053052805.png)

        把toos文件夹copy过来， 执行 mys.sh, 随后在go build prj2, 根据结果的错误再修改， 详细错误见项目2讲解。
![](_v_images/20190308174350287_1445471659.png)
![](_v_images/20190308174412815_2079085337.png)

7. 代码优化

        经过上面的几步，已完成修改，测试代码可以跑通，接下来就是代码优化，把各个文件原来写死的变量定义，以及配置文件的路径等等，都定义在配置文件里，通过读取配置文件来获取，代码修改时直接修改配置文件，方便代码后期的维护。
        core.yaml中的sdk的configPath的命名在开发环境，测试环境和正式环境都不一样，
        如果是迁移到kafka环境，还要修改对应的配置项。
        核心的配置文件：core.yaml
 ![](_v_images/20190308175205664_699599325.png)

        在base.go中 增加读取core配置文件的的函数
![](_v_images/20190308175352178_737072880.png)

        在main.go的main方法里调用 初始化core配置文件的的函数， 在main方法里一开始就初始化了core配置文件，那么在接下来的初始化channel以及chaincode的操作中，都可以直接获取core配置文件的值。
![](_v_images/20190308175834010_1250941600.png)

8. 加入Restful服务

        代码优化完成，可以实现内部的调用， 加入Restful服务，则可以实现外部调用，
        在main.go里面增加
 ![](_v_images/20190308181555921_1077242956.png)
![](_v_images/20190308181641744_162562809.png)
![](_v_images/20190308181716263_470796190.png)

        setResponseTypeBSI函数， 设置响应头的类型，设置跨域，
        跨域的问题：关于跨域,以及跨域的几种方式
        https://www.cnblogs.com/chenshishuo/p/4919224.html
        notFound 没有发现路由的响应信息

9. 加入业务逻辑

        业务逻辑部分在chain.go中定义，如block, tx， queryKV， invoke, query, queryWithBlock, 对业务逻辑中关键部分的处理在base.go中定义。
![](_v_images/20190308183209793_394124603.png)

        Post请求的invoke和query，通过脚本实现调用： client/invoke.sh ,  client/query.sh
![](_v_images/20190308184411041_427655459.png)

        每次设值后，进行查询都只能查询到最新的Key所对应的值，不能查询历史数据，所以在chain.go中增加一个查询历史数据的业务逻辑 queryWithBlock,
        invoke以及query操作都会调用chainCode，要查询历史数据，需要修改chainCode，增加处理历史数据的部分，
![](_v_images/20190308184914013_1166854440.png)

**chainCode修改：**
![](_v_images/20190308185420752_1392818094.png)

**修改的关键部分**
![](_v_images/20190308185450812_1693896733.png)
![](_v_images/20190308185608782_317781011.png)

**chainCode修改后，再进行设值，查询， 则可以查询到历史交易数据**
![](_v_images/20190308185739054_1685661400.png)

10. 项目docker化
把做好的项目做成一个docker 镜像， 那么可以在不同的平台部署，
老师做好的docker 镜像：xuxinlai2002/prj2：.v0.9

11. kafaka , zookeeper的介绍

13. k8s, rancher, swarm的使用