## 需求

* rocketmq集群部署在网络一，业务方应用部署在网络二中，网络一和网络二可以通信
* broker的主从之间通信、broker和namesrv之间的通信使用网络一
* 在网络二中的应用可以连接namesrv暴露出来的broker地址

## 难点

* 首先broker的地址是启动时候注册到namesrv上的，broker可以配置brokerIP1,brokerIP2。slaver、sdk等通过namesrv的接口查询broker的地址，但是查询到的始终是brokerIP1。
* 如果配置brokerIP1为网络一中的地址（相当于内网地址），那么sdk是无法访问的
* 如果配置brokerIP1为网络一和网络二的公共网络地址，那么看起来slaver和sdk都可以正常连接，但是又有两个问题，第一主从之间通过外网流量通信，第二，可能存在某种安全策略，禁止网络一直接访问公共网络

## 解决方案
* broker配置brokerIP1为外网地址，配置brokerIP2为内网地址
* 修改namesrv所有涉及查询broker地址的接口实现，请求中的扩展字段extFields增加一个key:useIP2，如果useIP2=true，namesrv就返回brokerIP2，否则返回brokerIP1
* 服务端集群使用修改过后的rocketmq源码重新编译打包部署（client模块设置了useIP2=true），业务方使用原生的sdk

## 注意
* 基于apache rocketmq release-4.2.0分支开发，新分支release-4.2.0-useIP2
* 现有tag:rocketmq-all-4.2.0-useIP2，我们部门内部已经经过测试并部署到生产环境使用
* 本次修改不改变rocketmq的任何核心功能和逻辑，仅仅修改broker地址获取的相关逻辑

-----
