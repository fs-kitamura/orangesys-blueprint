# test in aws ec2

## isntall docker

```bash
yum update -y
yum install docker -y

/etc/init.d/docker start
```

## run influxdb in docker

```bash
docker run -it -d -p 8086:8086 influxdb
curl -I localhost:8086/ping
```

## install prometheus in ec2

```bash
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -zxvf prometheus-2.0.0.linux-amd64.tar.gz
cp prometheus-2.0.0.linux-amd64/prometheus /usr/local/bin/
cp prometheus-2.0.0.linux-amd64/promtool /usr/local/bin/
cp -r prometheus-2.0.0.linux-amd64/consoles /etc/prometheus
cp -r prometheus-2.0.0.linux-amd64/console_libraries /etc/prometheus
```

## install node_explorer in ec2

```bash
curl -LO https://github.com/prometheus/node_exporter/releases/download/v0.15.1/node_exporter-0.15.1.linux-amd64.tar.gz
tar -zxvf node_exporter-0.15.1.linux-amd64.tar.gz
cp node_exporter-0.15.1.linux-amd64/node_exporter /usr/local/bin/
```

### run it 

```basah
node_exporter &
```

## create prometheus database in influxdb

```bash
docker-enter cc74d2fe9ff1
influx
create database prometheus
```

## configure prometheums.yml

```yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']

# Remote write configuration (for Graphite, OpenTSDB, or InfluxDB).
remote_write:
- url: "http://localhost:8086/api/v1/prom/write?db=prometheus"
# Remote read configuration (for InfluxDB only at the moment).
remote_read:
- url: "http://localhost:8086/api/v1/prom/read?db=prometheus"
```


## run it

```bash
/usr/local/bin/prometheus  --config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries &
```

## install grafana

```bash
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-4.6.3-1.x86_64.rpm
sudo yum localinstall grafana-4.6.3-1.x86_64.rpm -y
```

## configure grafana dashbard with webui

## bug

```bash
panic:interface conversion: influxql.Expr is *influxql.StringLiteral, not *influxql.RegexLiteral
```

fix with influxdb 1.4.3

https://github.com/influxdata/influxdb/issues/9134

