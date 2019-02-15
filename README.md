# eureka-server

### 1.master版本
有如下4个服务：
- eureka-server
- bank-server
- company-server
- user-server

此版本演示了2种场景：
#### 1.consumer -> provider
bank-server -> user-server
#### 2.consumer -> provider1，provider2
bank-server -> user-server,company-server

### 2.0.1版本
有如下4个服务：
- eureka-server
- bank-server
- company-server
- user-server

此版本演示了1种场景：
#### 1.consumer -> provider1->provider2
bank-server -> company-server -> user-server

此场景下，company-server在链路中，不仅是provider，也是consumer。


### 3.角色转换
一个provider转变为consumer时，会涉及到调用其他服务，feign调用时，这个api接口所在包，需要显式的配置扫描。
比如，我feign包下，有个UserServiceApi，里面是comsumer调用provider的api接口，那需要在consumer项目中配置：

@EnableFeignClients("com.java4all.feign")



