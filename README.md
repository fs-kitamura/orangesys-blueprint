# orangesys-blueprint
orangesys-blueprint

# Architecture

```
       ┌─────────────────────┐                                        
       │       Telegraf      ├─┐                                      
       └─┬───────────────────┘ ├─┐                                    
         └─┬───────────────────┘ │                                    
           └─────────────────────┘                                    
                      △                                               
      https + jwt ┌───┘                                               
                  ▼                                                   
       ┌─────────────────────┐                                        
       │       Traefik       │                                        
       └─────────────────────┘                                        
                  │                                                   
                  ▼                                                   
       ┌─────────────────────┐           ┌─────────────┐                             
       │        Kong         │◁─────────▶│   Postgres  │               
       └─────────────────────┘           └─────────────┘          
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

### [https://github.com/influxdata/telegraf][Telegraf]

クライアントからメトリクスを集計し、転送します。https通信、jwt認証を利用します。
influxdbの[https://docs.influxdata.com/influxdb/v1.0/write_protocols/line_protocol_reference/][line protocol]を利用します。
顧客毎、agent endpointが異なってます。
例)

>```
[[outputs.influxdb]]
  urls = ["https://pqaxel.i.orangesys.io/?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiI2OTA1MmI0ZWMzODk0ZjZiOWJiM2Q0ZGE2NDg0ZGQ3NiJ9.9dEvXPYp7IhRjexFfBWR5uBMOoR0U7tJdBh-f4Fwxdw"]
  database = "telegraf" # required
>```

default database [telegraf]

### [https://github.com/containous/traefik][Traefik]

[https://docs.traefik.io/toml/#acme-lets-encrypt-configuration][ACME (Let's Encrypt)] を利用し、SSL証明書を自動更新します。

Kubernetesのingressと連動します。

[https://medium.com/@patrickeasters/using-traefik-with-tls-on-kubernetes-cb67fb43a948#.kuk2s9gqh][Using Traefik with TLS on Kubernetes]


### [https://github.com/Mashape/kong][Kong]

API Gateway
- ユーザーからの[https://getkong.org/plugins/jwt/][JWT]認証
- [https://getkong.org/plugins/correlation-id/][corelation-id]を追加、tracking用
- [https://getkong.org/plugins/request-transformer/][querystring]をrewriteします。jwt認証からInfluxdb認証に切り替え、write only権限になります。


### Postgres

Kongに関する設定を保存するDataBase
[https://getkong.org/docs/0.9.x/clustering][kong-clustering]

### Influxdb

監視データを保存します。顧客毎環境を分離します。
PLANによって、パフォーマンスが異なってます。
trackingのため、logの部分を修正します。kongのcorrelation-idをログに吐き出します。

>```
#vim services/httpd/response_logger.go
detect(r.Header.Get("Orangesys-Request-Id"), "-"),
>```

[https://github.com/gavinzhou/influxdb][orangesys influxdb]
[https://docs.influxdata.com/influxdb/v1.0/][influxdb Doc]

### Grafana

influxdb dataのDashboardです。顧客毎環境を分離します。
PLANによって、パフォーマンスが異なってます。
[https://github.com/grafana/grafana][github grafana]

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
