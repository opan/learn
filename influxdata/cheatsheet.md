# TICK / InfluxData

#### Installation Telegraf

- OS X
```
brew update && brew install telegraf
```

- Ubuntu/Debian
```
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.5.2-1_amd64.deb
sudo dpkg -i telegraf_1.5.2-1_amd64.deb
```

---

#### Installation InfluxDB

You can refer to this [link](https://docs.influxdata.com/influxdb/v1.5/introduction/installation/)
You can run the influxdb service (in Mac) with `brew services start influxdb`
or you can run manually with `https://docs.influxdata.com/influxdb/v1.5/introduction/installation/`

#### About InfluxDB

InfluxDB is a time-series database.

---

#### Installation Kapacitor

- OS X
```
brew update
brew install kapacitor
```

- Docker
```
docker pull kapacitor
```

- Ubuntu/Debian
```
wget https://dl.influxdata.com/kapacitor/releases/kapacitor_1.4.0_amd64.deb
sudo dpkg -i kapacitor_1.4.0_amd64.deb
```

To make `launchd` start kapacitor at login (OS X using Homebrew)
- `ln -sfv /usr/local/opt/kapacitor/*.plist ~/Library/LaunchAgents`
- Then load kapacitor `launchctl load ~/Library/LaunchAgents/homebrew.mxcl.kapacitor.plist`
- Or if you don't want/need `launchctl` you can just run `kapacitord -config /usr/local/etc/kapacitor.conf`

#### Configuration Kapacitor

Sample config can be found [here](https://github.com/influxdata/kapacitor/blob/master/etc/kapacitor/kapacitor.conf) or you can see the example by run `kapacitord config`.
To generate new configuration file, run `kapacitord config > kapacitor.generated.conf`

#### About Kapacitor

Kapacitor is an open source data processing framework that makes it easy to create alerts, run ETL jobs and detect anomalies.
Key features:
- Process both streaming data and batch data
- Query data from InfluxDB on schedule, and receive data via the line protocol and any other method InfluxDB supports
- Perform any transformation currently possible in InfluxQL
- Store transformed data back in InfluxDB
- Add custom user defined functions to detect anomalies
- Integrate with HipChat, OpsGenie, Alerta, Sensu, PagerDuty, Slack and more

---

#### Misc

- Mac
```
brew services [stop|start] service_name #
```

To see list services `brew services list`

- Ubuntu/Debian
```
sudo service service_name [stop|start|status]
```
