# TICK / InfluxData

## Installation Telegraf

- OS X
```
brew update && brew install telegraf
```

- Ubuntu/Debian
```
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.5.2-1_amd64.deb
sudo dpkg -i telegraf_1.5.2-1_amd64.deb
```

## About Telegraf

Telegraf uses plugins to input and output data. Default output plugin is InfluxDB

---

## Installation InfluxDB

You can refer to this [link](https://docs.influxdata.com/influxdb/v1.5/introduction/installation/)
You can run the influxdb service (in Mac) with `brew services start influxdb`
or you can run manually with `https://docs.influxdata.com/influxdb/v1.5/introduction/installation/`

## About InfluxDB

- InfluxDB is a time-series database.
- Data in influxdb is organized by *time series*, which contain a measured value like *cpu load* or *temperature*.

The query syntax of influxdb is pretty similar with query languange out there like mysql and postgresql.
```sql
CREATE DATABASE mydb; // Create new database
SHOW DATABASES; // Show lists databases
USE mydb; // Use specific DB
```

Most of the influxdb command must operate againts specific database except `SHOW DATABASE`.

Influxdb have
- Zero to many `points`.
- Each `points` have
  - `time` (timestamp). optional influxdb will provide it manually if not included
  - `measurement` (e.g: *cpu_load*). required
  - At least have one key-value `field` (the measured value itself, e.g: *temperature=12.3*). required
  - Zero to many key-value `tags` containing any metadata about the value, e.g: *host=server01*, *region=EMEA*. optional

You can think `measurement` is a SQL table, where the primary key is always time. `tags` and `fields` are effectively
columns in the table while `tags` are indexed and `fields` are not.

`points` are written to influxdb using the Line Protocol which follows the following format:
```
<measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
```

Sample valid `points`
```
cpu,host=serverA,region=us_west value=0.64
payment,device=mobile,product=Notepad,method=credit billed=33,licenses=3i 1434067467100293230
stock,symbol=AAPL bid=127.46,ask=127.48
temperature,machine=unit42,type=assembly external=25,internal=37 1434067467000000000
```
You can have million of measurements, you don't have to define schema and `null` values aren't stored.



---

## Installation Kapacitor

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

## Configuration Kapacitor

Sample config can be found [here](https://github.com/influxdata/kapacitor/blob/master/etc/kapacitor/kapacitor.conf) or you can see the example by run `kapacitord config`.
To generate new configuration file, run `kapacitord config > kapacitor.generated.conf`

## About Kapacitor

Kapacitor is an open source data processing framework that makes it easy to create alerts, run ETL jobs and detect anomalies.
Key features:
- Process both streaming data and batch data
- Query data from InfluxDB on schedule, and receive data via the line protocol and any other method InfluxDB supports
- Perform any transformation currently possible in InfluxQL
- Store transformed data back in InfluxDB
- Add custom user defined functions to detect anomalies
- Integrate with HipChat, OpsGenie, Alerta, Sensu, PagerDuty, Slack and more

---

## Installation Chronograf / Grafana

#### Installing Chronograf

- OS X: `brew update && brew install chronograf`
- Ubuntu/Debian:
```
wget https://dl.influxdata.com/chronograf/releases/chronograf_1.4.2.3_amd64.deb
sudo dpkg -i chronograf_1.4.2.3_amd64.deb
```

#### Running Up Chronograf

- Run Chronograf server:
  - OS X: `brew services start chronograf`
  - Ubuntu/Debian: `sudo systemctl start chronograf`
- Now access from browser `localhost:8888`. You can change default port that Chronograf used
by provide the `--port` command, e.g `chronograf --port=9999` or by adding environment variable `PORT`.
Or you can change the port from `chronograf.service`. In Ubuntu it is located in `/lib/systemd/system/chronograf.service`.

#### Installing Grafana

- Ubuntu/Debian:
```
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_5.0.2_amd64.deb 
sudo dpkg -i grafana_5.0.2_amd64.deb 
```
- OS X: `brew update && brew install grafana`

#### Running Up Grafana

- Start Grafana server:
  - Ubuntu/Debian: `sudo systemctl start grafana-server`
  - OS X: `brew services start grafana`
- You can check the config file from:
  - Ubuntu/Debian: `sudo systemctl status grafana-server`
- To start Grafana server at boot time you can:
  - Ubuntu/Debian: `sudo update-rc.d grafana-server defaults` or `sudo systemctl enable grafana-server.service`
- Config files is located in:
  - Ubuntu/Debian: `/etc/grafana/grafana.ini`


---

## Integrating TICK component

Quick summary:
- Telegraf collects time-series data from a variety of sources
- InfluxDB stores a time-series data
- Chronograf/Graphana visualizes and graphs the time-series data
- Kapacitor provides alerting and detects anomalies in time-series data

Integrating the components:

#### Setup InfluxDB

- Start influxdb console with `influx` command.
- Create new database `create database your_db;`
- Create new admin user `create admin "username" with password 'user_password' with all privileges;`
- Check if new user and database is successfully created with `show users` and `show databases`
- Now open the influxdb config file. It is depend on your OS:
  - Ubuntu/Debian

  The config file should be located in `/etc/influxdb/influxdb.conf` but you can make sure
  with command `systemctl status influxdb`.

  - OS X

  In OS X, you can check where the config file is located with `brew info influxdb`.

- Inside influxdb config file, change the value of `auth-enabled` from `false` to `true`.
This config will enabled user authentication over HTTP/HTTPS. Then restart influxdb server
with `sudo systemctl restart influxdb` or `brew services restart influxdb` (execute similar command in you OS)

#### Setup Telegraf

- Update Telegraf config:
  - Ubuntu/Debian: `/etc/telegraf/telegraf.conf` or find with `sudo systemctl status telegraf`
  - OS X: `/usr/local/etc/telegraf.conf` or find with `brew info telegraf`
  
- Find `outputs.influxdb` in `telegraf.conf` and provide username and password that you already
create in steps above for influxdb
```
[[outputs.influxdb]]
  ...
  username = "username"
  password = "password"
  ...
```
and then restart Telegraf service `sudo systemctl restart telegraf` (execute similar command in your OS)

#### Setup Kapacitor

- Update Kapacitor config:
  - Ubuntu/Debian: `/etc/kapacitor/kapacitor.conf` or find with `sudo systemctl status kapacitor`
  - OS X: `/usr/local/etc/kapacitor.conf` or find with `brew info kapacitor`

- Find `[[influxdb]]` and add username and password to connect to influxdb database
```
[[influxdb]]
  ...
  username = "username"
  password = "password"
  ...
```
- Then restart Kapacitor service (the command is similar like the other component, just change the service name)
- You can verify if Kapacitor is running with `kapacitor task lists`


---

## Misc

- Mac
```
brew services [stop|start] service_name #
```

To see list services `brew services list`

- Ubuntu/Debian
```
sudo service service_name [stop|start|status]
sudo systemctl [restart|start|stop|status] service_name # It is similar to `service` command
```

