# fabric v1.0学习
## hyperledger-fabric官网
[fabric官网](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)

## 1. Logging日志模块
### 1.1 go-logging

    Fabric日志系统主要使用了第三方包go-logging, 可在github.com/op/go-logging下载。
    go doc详细使用： https://www.godoc.org/github.com/op/go-logging
### 1.2 fllogging
    Fabric在go-logging的基础上，Fabric封装出了flogging,代码集中在 fabric/common/flogging目录下，供项目全局使用。

## 2. 错误机制设计

## 3.




```go
//initConfig initializes viper config
func InitConfig() error {

	viper.AddConfigPath("./")
	viper.SetConfigName("core")

	viper.SetEnvPrefix(ProjectName)
	viper.AutomaticEnv()
	replacer := strings.NewReplacer(".", "_")
	viper.SetEnvKeyReplacer(replacer)
	err := viper.ReadInConfig()
	if err != nil {
		return fmt.Errorf("fatal error config file: %s ", err)
	}

	return nil
}
```

### 3.4 grpc服务
