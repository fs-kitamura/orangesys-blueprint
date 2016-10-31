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
       │       Traefik       │                                        
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
  │  Influxdb  ├─┐   │   Influxdb  ├─┐    │   Influxdb  ├─┐           
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
        slack ───▶ │ alerting ├─┐             │       ┌───────────┐
        email      └─┬────────┘ ├─┐           └──────▶│  MariaDB  │
                     └─┬────────┘ │                   └───────────┘             
                       └──────────┘                                             
```

## Components

### [Telegraf](https://github.com/influxdata/telegraf)

クライアントからメトリクスを集計し、転送します。https通信、jwt認証を利用します。

influxdbの[line protocol](https://docs.influxdata.com/influxdb/v1.0/write_protocols/line_protocol_reference/)を利用します。

顧客毎、agent endpointが異なってます。

例)

>```
[[outputs.influxdb]]
  urls = ["https://pqaxel.i.orangesys.io/?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiI2OTA1MmI0ZWMzODk0ZjZiOWJiM2Q0ZGE2NDg0ZGQ3NiJ9.9dEvXPYp7IhRjexFfBWR5uBMOoR0U7tJdBh-f4Fwxdw"]
  database = "telegraf" # required
>```

default database [telegraf]

### [Traefik](https://github.com/containous/traefik)

[ACME (Let's Encrypt)](https://docs.traefik.io/toml/#acme-lets-encrypt-configuration)を利用し、SSL証明書を自動更新します。

Kubernetesのingressと連動します。

[Using Traefik with TLS on Kubernetes](https://medium.com/@patrickeasters/using-traefik-with-tls-on-kubernetes-cb67fb43a948)

### [Kong](https://github.com/Mashape/kong)

API Gateway
- ユーザーからの[JWT](https://getkong.org/plugins/jwt/)認証
- [corelation-id](https://getkong.org/plugins/correlation-id/)を追加、tracking用
- [querystring](https://getkong.org/plugins/request-transformer/)をrewriteします。
  jwt認証からInfluxdb認証に切り替え、write only権限になります。


### Postgres

Kongに関する設定を保存するDataBase
[kong-clustering](https://getkong.org/docs/0.9.x/clustering)

### [Influxdb](https://github.com/influxdata/influxdb)

監視データを保存します。顧客毎環境を分離します。

PLANによって、パフォーマンスが異なってます。

trackingのため、logの部分を修正します。kongのcorrelation-idをログに吐き出します。

>```
#vim services/httpd/response_logger.go
detect(r.Header.Get("Orangesys-Request-Id"), "-"),
>```

[orangesys influxdb](https://github.com/gavinzhou/influxdb)
[influxdb Doc](https://docs.influxdata.com/influxdb/v1.0/)

### [Grafana](https://github.com/grafana/grafana)

influxdb dataのDashboardです。顧客毎環境を分離します。

PLANによって、パフォーマンスが異なってます。

[github grafana](https://github.com/grafana/grafana)

### alerting

Grafana4.0からalerting機能を追加します。

現時点mailとslackを対応します。

### MariaDB

Grafanaの設定、Dashboardを保存します。

databaseを顧客毎分けます、インスタンスが共通となります。

>```
grafanaのdefault.ini
[database]
type = mysql
host = 127.0.0.1:3306
name = grafana
user = root
password = root
>```

## License

MIT License, see [LICENSE](LICENSE).
