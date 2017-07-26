[![StackShare](http://img.shields.io/badge/tech-stack-0690fa.svg?style=flat)](http://stackshare.io/orangesys/orangesys)
# orangesys-blueprint
orangesys-blueprint

# Architecture

```
  ┌────────────┐     ┌─────────────┐      ┌─────────────┐
  │  Telegraf  ├─┐   │   Telegraf  ├─┐    │   Telegraf  ├─┐
  └─┬──────────┘ ├─┐ └─┬───────────┘ ├─┐  └─┬───────────┘ ├─┐
    └─┬──────────┘ │   └─┬───────────┘ │    └─┬───────────┘ │
      └────────────┘     └─────────────┘      └─────────────┘
             △                  △                    △
             │                  │                    │
             └──────────────────┴───┬────────────────┘
                                    │
      https + jwt ┌─────────────────┘
                  │
                  ▼                                                   
       ┌─────────────────────┐                                        
       │       Ingress       │                                        
       └─────────────────────┘                                        
                  │                                                   
                  ▼                                                   
       ┌─────────────────────┐             ┌─────────────┐
       │        Kong         ├─┐◁─────────▶│   Postgres  │
       └─┬───────────────────┘ │           └─────────────┘
         └─────────────────────┘
                  │                               
                  └───────┐                      
                          │                     
              write only  │                     
                          │                     
         ┌────────────────┴─┬────────────────────┐                    
         │                  │                    │                    
         ▼                  ▼                    ▼                    
  ┌────────────┐     ┌─────────────┐      ┌─────────────┐             
  │    TSDB    ├─┐   │    TSDB     ├─┐    │     TSDB    ├─┐           
  └─┬──────────┘ ├─┐ └─┬───────────┘ ├─┐  └─┬───────────┘ ├─┐         
    └─┬──────────┘ │   └─┬───────────┘ │    └─┬───────────┘ │         
      └────────────┘     └─────────────┘      └─────────────┘         
             △                  △                    △
             │                  │                    │                
            ┌┘  read only       └┐                   └─┐                   
            │                    │                     │                                    
   ┌──── ───── ───── ┐    ┌──── ───── ────┐  ┌──── ───── ────┐                          
   │     Grafana     │    │    Grafana    │  │    Grafana    │                         
   └ ───── ───── ────┘    └── ───── ───── ┘  └── ───── ───── ┘                         
            │                     │                  │                
            └───────────┬─────────┴────┬─────────────┘                
                        │              △                                        
                        │              │                                        
                        │              └──────┐                      
                        ▼                     │                                                       
                   ┌──────────┐               │                    
                   │  slack   ├─┐             │       ┌───────────┐
                   └─┬────────┘ ├─┐           └──────▶│  MariaDB  │
                     └─┬────────┘ │                   └───────────┘             
                       └──────────┘                                             
```

## Components

### [Telegraf](https://github.com/influxdata/telegraf)

クライアントからメトリクスを集計し、転送します。https通信、jwt認証を利用します。

顧客毎、agent endpointが異なってます。

例)

```
[[outputs.orangesys]]
  urls = ["https://demo.i.orangesys.io/"]
  jwt_token = <jwt_token>
  database = "telegraf" # required
```

### [Kong](https://github.com/Mashape/kong)

API Gateway
- ユーザーからの[JWT](https://getkong.org/plugins/jwt/)認証
- [querystring](https://getkong.org/plugins/request-transformer/)をrewriteします。
  jwt認証からInfluxdb認証に切り替え、write only権限になります。


### Postgres

Kongに関する設定を保存するDataBase
[kong-clustering](https://getkong.org/docs/0.9.x/clustering)

監視データを保存します。顧客毎環境を分離します。

PLANによって、パフォーマンスが異なってます。

trackingのため、logの部分を修正します。kongのcorrelation-idをログに吐き出します。

### [Grafana](https://github.com/grafana/grafana)

PLANによって、パフォーマンスが異なってます。

[github grafana](https://github.com/grafana/grafana)

### alerting

現時点slackとwebhookを対応します。
_email機能が未開放 in Orangesys_

### MariaDB

Grafanaの設定、Dashboardを保存します。

databaseを顧客毎分けます、インスタンスが共通となります。

```
grafanaのdefault.ini
[database]
type = mysql
host = 127.0.0.1:3306
name = grafana
user = root
password = root
```

## License

MIT License, see [LICENSE](LICENSE).