# eureka-server

##### 此demo有多项目，多分支，请根据场景，拉取对应项目的对应分支demo测试。

### 1.master分支
有如下4个服务：
- eureka-server             branch master
- bank-server               branch master
- company-server            branch master
- user-server               branch master

此版本演示了2种场景：
#### 1.consumer -> provider
bank-server -> user-server
#### 2.consumer -> provider1，provider2
bank-server -> user-server,company-server

### 2.0.1分支
有如下4个服务：
- eureka-server             branch 0.1
- bank-server               branch 0.1
- company-server            branch 0.1
- user-server               branch 0.1

此版本演示了1种场景：
#### 1.consumer -> provider1->provider2
bank-server -> company-server -> user-server

此场景下，company-server在链路中，不仅是provider，也是consumer。

### 3.0.2分支
有如下5个服务：
- eureka-server 8761        branch 0.2
- eureka-server2 8762       branch 0.2
- bank-server               branch 0.2
- company-server            branch 0.2
- user-server               branch 0.2

eureka-server,eureka-server2为一个eureka集群，其他服务注册到eureka-server上。

### 4.版本0.4
有如下4个服务：
- eureka-server 8761        branch 0.2
- eureka-server2 8762       branch 0.2
- bank4                     branch master
- user4                     branch master

eureka-server,eureka-server2为一个eureka集群，其他服务注册到eureka-server上。
此版本演示了1种场景：
#### 1.consumer -> provider
bank4 -> user4

此版本的存在，是为了演示在byteTCC 0.4.x版本。

### 5.版本号对应
#### 1.demo的master分支，0.1分支，0.2分支，对应的版本号：

- byteTCC 0.5.x

- Springboot 2.1.2.RELEASE

- SpringCloud Greenwich.RELEASE

#### 2.版本0.4对应的版本号：

- byteTCC 0.4.x

- Springboot 1.5.4.RELEASE

- SpringCloud Edgware.SR4

由于0.4.x版本，和0.5.x版本，整合方式相差较大，个人认为，0.4.x版本不太友好，请提前搞清自己项目的Springboot和Springcloud版本，选择对应的byteTCC版本。

### 6.角色转换
一个provider转变为consumer时，会涉及到调用其他服务，feign调用时，这个api接口所在包，需要显式的配置扫描。

比如，我feign包下，有个UserServiceApi，里面是comsumer调用provider的api接口，那需要在consumer项目中配置：

@EnableFeignClients("com.java4all.feign")

### 7.涉及变量传递
参考0.1版本
#### 1.自行处理
由于tcc的每一个阶段，使用的参数是相同的，
try中如果对接口参数做了任何处理，那么在cc阶段，也需要做相同的处理。
比如，controller中，方法中拿到参数后做了处理，然后去调用的try的实现，
那么这个处理，必须也同步在confirm和cancel中。

此种方式，比较原始，建议在框架层面来处理。

#### 2.框架处理
借助CompensableContextAware，CompensableContext，
在try阶段，把处理后的值存入上下文中，cc阶段直接获取使用即可。

set value参考：https://github.com/distributed-demo/company-server/blob/0.1/src/main/java/com/java4all/service/impl/CompanyServiceImpl.java

get value参考：https://github.com/distributed-demo/company-server/blob/0.1/src/main/java/com/java4all/service/impl/CompanyServiceConfirm.java

### 8.负载粒度
ByteTCC计划对负载均衡的支持粒度，可分为两种：
- a、按事务进行负载均衡；
- b、按请求进行负载均衡。
#### 1.按事务进行负载均衡
在某个事务T内，consumer端应用app1首次向provider端应用app2（集群环境）发起请求时，
ByteTCC使用random负载均衡策略将其随机分发到一个app2实例（如inst2）；

后续app1在该事务T内再次向app2发起请求时，将始终落在inst2（即首次请求的处理实例）上。
#### 2.按请求进行负载均衡
consumer端应用app1向provider端应用app2（集群环境）发起请求时，
ByteTCC始终按业务系统指定的负载均衡策略将请求分发到一个app2实例。

注意：ByteTCC 0.4.x版本默认为支持“按事务进行负载均衡”的特性。

### 9.幂等性
ByteTCC不要求service的实现逻辑具有幂等性，ByteTCC在TCC事务提交/回滚时，虽然也可能会多次调用confirm/cancel方法，但是ByteTCC可以确保每个confirm/cancel方法仅被"执行并提交"一次。所以，在使用ByteTCC时可以仅关注业务逻辑，而不必考虑事务相关的细节。

1.Confirm操作虽然可能被多次调用，但是其参与的LocalTransaction均由ByteTCC事务管理器控制，一旦Confirm操作所在的LocalTransaction事务被ByteTCC事务管理器成功提交，则ByteTCC事务管理器会标注该Confirm操作成功，后续将不再执行该Confirm操作。

2.Cancel操作的控制原理同Confirm操作。需要说明的是，Cancel操作只有在Try阶段所在的LocalTransaction被成功提交的情况下才会被调用，Try阶段所在的LocalTransaction被回滚时Cancel操作不会被执行。

### 10.调用成环
有关这个场景，此项目未作测试，贴一下作者对此问题的看法：
一般来讲两个服务互相依赖（或者，调用链路中存在环）不是一个很好的设计，不过ByteTCC也允许这种情况，但是如果这个调用环中还存在修改同一条记录的情况，这个ByteTCC就解决不了了。只能改一下业务设计了，比如异步？或者要不就把第二次修改放到confirm中执行，因为执行confirm的时候，一般而言，try基本上都已经commit了。
### 11.异常处理
try 会改变事务走向。

### 12.可参考文档：

##### byteTCC地址
https://github.com/liuyangming/ByteTCC

##### byteTCC官方手册
https://github.com/liuyangming/ByteTCC/wiki/User-Guide-0.5.x

https://github.com/liuyangming/ByteTCC/wiki/User-Guide-0.4.x

##### byteTCC官方样例
https://github.com/liuyangming/ByteTCC-sample

##### 上述文档中整合样例
https://github.com/distributed-demo

##### 相关博客
https://blog.csdn.net/weixin_39800144/column/info/30988

- Wechat:1186355422
- blog:https://it4all.blog.csdn.net
