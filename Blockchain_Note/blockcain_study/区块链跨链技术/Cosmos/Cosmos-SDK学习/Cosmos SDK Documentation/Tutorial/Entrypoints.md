# Entrypoints

In golang the convention is to place files that compile to a binary in the `./cmd` folder of a project. For your application there are 2 binaries that you want to create:

在golang中，约定是将编译为二进制的文件放在项目的`./cmd`文件夹中。对于你的应用你需要创建两个二进制文件：

- `nsd`: This binary is similar to `bitcoind` or other cryptocurrency daemons in that it maintains p2p connections, propagates transactions, handles local storage and provides an RPC interface to interact with the network. In this case, Tendermint is used for networking and transaction ordering.
- `nsd`:这个二进制文件和`bitcoind`或其他加密资产的后台进程是相似的，在后台进程中，它包含了p2p连接，传播交易，处理本地存储和与网络交互的RPC接口。在这种情况下，Tendermint用于网络和交易排序。

- `nscli`: This binary provides commands that allow users to interact with your application.

- `nscli`:这个二进制文件提供了允许用户与你的应用交互的命令。

To get started create two files in your project directory that will instantiate these binaries:

开始在项目目录中创建两个将实例化这些二进制文件的文件：

- `./cmd/nsd/main.go`
- `./cmd/nscli/main.go`

## nsd
---------------------------------------------------------------------------------------------------------------------------------------------------------

Start by adding the following code to `cmd/nsd/main.go`:

![](_v_images/20190617171227232_308561869.png)




Notes on the above code:

- Most of the code above combines the CLI commands from Tendermint, Cosmos-SDK and your Nameservice module.



## nscli
---------------------------------------------------------------------------------------------------------------------------------------------------------

Finish up by building the `nscli` command:

![](_v_images/20190617171727642_916705864.png)



Note:

- The code combines the CLI commands from Tendermint, Cosmos-SDK and your Nameservice module.
- The [cobra CLI documentation](http://github.com/spf13/cobra) will help with understanding the above code.
- You can see the `ModuleClient` defined earlier in action here.
- Note how the routes are included in the `registerRoutes` function.

## Now that you have your binaries defined its time to deal with [dependency management and build your app](https://cosmos.network/docs/tutorial/gomod.html)!

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Building your app

## Makefile
---------------------------------------------------------------------------------------------------------------------------------------------------------

Help users build your application by writing a` ./Makefile` in the root directory that includes common commands:

NOTE: The below Makefile contains some of same commands as the Cosmos SDK and Tendermint Makefiles.


## How about including Ledger Nano S support?

This requires a few small changes:

- Create a file` Makefile.ledger` with the following content:


- Add include `Makefile.ledge`r at the beginning of the `Makefile`:


## go.mod
---------------------------------------------------------------------------------------------------------------------------------------------------------

Golang has a few dependency management tools. In this tutorial you will be using [Go Modules](https://github.com/golang/go/wiki/Modules).` Go Modules` uses a `go.mod` file in the root of the repository to define what dependencies the application needs. Cosmos SDK apps currently depend on specific versions of some libraries. The below manifest contains all the necessary versions. To get started replace the contents of the ./go.mod file with the `constraints` and `overrides` below:

![](_v_images/20190617173515356_1230441019.png)


![](_v_images/20190617174018754_881119706.png)

![](_v_images/20190617173957041_1697680113.png)

![](_v_images/20190617174956268_2056852510.png)


## Building the app

命令行执行：

![](_v_images/20190617175015837_2098075275.png)

## Congratulations, you have finished your nameservice application! Try [running and interacting with it](https://cosmos.network/docs/tutorial/build-run.html)!






