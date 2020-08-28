# tristar-to-telegraf
Python script to output data from ethernet enabled Morningstar Tristar charge controllers in JSON format for Telegraph/InfluxDB/Grafana stack.
 
Tested with Tristar TS-MPPT-60

Modified version of brocktice's tristar script https://github.com/brocktice/tristar-python-modbus
***
### Usage

python read_tsmppt.py `<ip address>` `<alias>`

**Example:** python3 read_tsmppt.py 192.168.1.100 Solar1

Output:

`{"SiteID": "Solar1", "battvoltmin": 26.52099609375, "battamps": 3.5791015625, "powerout": 96.0205078125, "battsense": 0.0, "arrayamps": 1.4111328125, "battwatts": 96.08131349086761, "batttempmin": 25, "state": 5, "arraywatts": 104.6773910522461, "heatsinktemp": 19, "battvolt": 26.8450927734375, "powerin": 104.8095703125, "battvoltmax": 26.9110107421875, "arrayvolt": 74.1796875, "batttempmax": 25, "batttemp": 25}`
***
### Telegraf.conf

```
[[inputs.exec]]
   commands = [
     "python3 /home/user/read_tsmppt.py 192.168.1.100 Solar1",
     "python3 /home/user/read_tsmppt.py 192.168.1.101 Solar2",
     "python3 /home/user/read_tsmppt.py 192.168.1.102 Solar3"
]
   tag_keys = [
    "SiteID"
  ]

   interval = "60s"
   timeout = "5s"
   name_suffix = "_SOLAR"
   data_format = "json"
```   
***
### Grafana Queries

**Voltage** `SELECT mean("battvolt") FROM "exec_SOLAR" WHERE ("SiteID" = 'Solar1') AND $timeFilter GROUP BY time($__interval) fill(linear)`

**Current** `SELECT mean("battamps") FROM "exec_SOLAR" WHERE ("SiteID" = 'Solar1') AND "battamps" < 100 AND $timeFilter GROUP BY time($__interval) fill(linear)`

**Batt Temp** `SELECT mean("batttemp") FROM "exec_SOLAR" WHERE ("SiteID" = 'Solar1') AND "batttemp" < 100 AND $timeFilter GROUP BY time($__interval) fill(linear)`

![Grafana Dashboard](https://github.com/bicnz/tristar-to-telegraf/raw/master/grafana-dashboard.png)
