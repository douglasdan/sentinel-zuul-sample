# sentinel-zuul-sample

Zuul does not provide rateLimit function, If use `SentinelRibbonFilter` route filter. it wrapped by Hystrix Command. so only provide Service level 
circuit protect. 

Sentinel can provide `ServiceId` level and `API Path` level flow control for zuul gateway service. 
This is simple project for [alibaba/Sentinel](https://github.com/alibaba/Sentinel) spring cloud zuul integration sample. 

*Note*: this project is for zuul 1.

## Modules

**eureka-server**:

module used for discovery server. when service restart. it should be restart.  
- `EurekaServiceApplication`: runs eureka service on port 8671. access by `http://localhost:8761/`

**zuul-backend**

backend service after gateway. consist of two application book and coke.  
- `CokeApplication`: hosts a `/coke` ,`/except` service on port 9082
- `BookApplication`: hosts a `/coke` ,`/except` service with different implementation on port 9081

**zuul-gateway**

zuul gateway application.
- `GatewayApplication` runs zuul on port 8990.


## How to run

run as spring boot application

```bash
curl -i localhost:8990/coke/coke

curl -i localhost:8990/coke/block

curl -i localhost:8990/coke/except

curl -i localhost:8990/book/coke

```

## Integration Sentinel Filter

- `SentinelRibbonFilter`: extends `RibbonRoutingFilter`. override run method which add sentinel check. this project also enable **Hystrix Circuit**. 
- `SentinelOkHttpRoutingFilter`:  use okHttp3 as custom routing http client. and without **Hystrix** bind.

By default use `SentinelOkHttpRoutingFilter` as route filter:

```java
 // disable filter can be set here.
 public static void main(String[] args) {
        new SpringApplicationBuilder(GatewayApplication.class)
                .properties("zuul.RibbonRoutingFilter.route.disable=true",
                        "zuul.SentinelRibbonFilter.route.disable=true")
                .run(args);
    }
    
```


Filters create structure like:


```
1. add vm config: -Dcsp.sentinel.api.port=18990

2. curl http://localhost:18990/tree?type=root

EntranceNode: machine-root(t:3 pq:0 bq:0 tq:0 rt:0 prq:0 1mp:0 1mb:0 1mt:0)
-EntranceNode: coke(t:2 pq:0 bq:0 tq:0 rt:0 prq:0 1mp:0 1mb:0 1mt:0)
--coke(t:2 pq:0 bq:0 tq:0 rt:0 prq:0 1mp:0 1mb:0 1mt:0)
---/coke/coke(t:0 pq:0 bq:0 tq:0 rt:0 prq:0 1mp:0 1mb:0 1mt:0)
-EntranceNode: sentinel_default_context(t:0 pq:0 bq:0 tq:0 rt:0 prq:0 1mp:0 1mb:0 1mt:0)
-EntranceNode: book(t:1 pq:0 bq:0 tq:0 rt:0 prq:0 1mp:0 1mb:0 1mt:0)
--book(t:1 pq:0 bq:0 tq:0 rt:0 prq:0 1mp:0 1mb:0 1mt:0)
---/book/coke(t:0 pq:0 bq:0 tq:0 rt:0 prq:0 1mp:0 1mb:0 1mt:0)


```

`book` and `coke` are serviceId. 

`---/book/coke` is api path.


## Integration with Sentinel DashBord

1. start [Sentinel DashBord](https://github.com/alibaba/Sentinel/wiki/%E6%8E%A7%E5%88%B6%E5%8F%B0).

2. add vm property to zuul-gateway. `-Dcsp.sentinel.dashboard.server=localhost:8088 -Dcsp.sentinel.api.port=18990`. 

3. Sentinel has full rule config features. see [Dynamic-Rule-Configuration](https://github.com/alibaba/Sentinel/wiki/Dynamic-Rule-Configuration)

## Fallback

Zuul provide `FallbackProvider` to cope with fall back logic. 
