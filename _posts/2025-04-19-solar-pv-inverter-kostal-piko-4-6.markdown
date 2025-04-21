---
layout: post
title: Kostal Piko 4.6 API
date: 2025-04-19 14:17:00
---

Lately i got interested in the data capture of certain energy values around our
house. One of those measurements are the values from our PV inverter. It is a
Piko 4.6 from Kostal.

Through some research and trial and error i discovered that there is an API
endpoint that can be queried for certain values. It is located at:
`http://<ip-of-your-inverter>/api/dxs.json?<list-of-data to query>`.

I discovered that the following numbers correspond to the following values:

| Dxs entry id | Value description                                          | Unit |
| ------------ | ---------------------------------------------------------- | ---- |
| 251658753    | Total yield - de: Ertrag gesamt                            | kWh  |
| 251658496    | Operating hours - de: Betriebsstunden                      | h    |
| 251659009    | Total household consumption - de: Hausverbrauch gesamt     | kWh  |
| 251659265    | Total self-consumption - de: Eigenverbrauch gesamt         | kWh  |
| 251659280    | Self-consumption rate - de: Eigenverbrauchsquote           | %    |
| 251659281    | Total degree of self-sufficiency - de: Autarkiegrad gesamt | %    |
| 251658754    | Yield today - de: Ertrag heute                             | Wh   |
| 251659010    | Household consumption today - de: Hausverbrauch heute      | Wh   |
| 251659266    | Self-consumption today - de: Eigenverbrauch heute          | Wh   |
| 251659279    | Degree of self-sufficiency - de: Autarkiegrad              | %    |
| 33556736     | Total DC input power - de: C Eingangsleistung gesamt       | W    |
| 67109120     | Total AC output power - de: C Ausgangsleistung gesamt      | W    |
| 33555202     | Input DC 1 voltage - de: Eingang DC 1 Spannung             | V    |
| 33555201     | Input DC 1 current - de: Eingang DC 1 Strom                | A    |
| 33555203     | Input DC 1 Power - de: Eingang DC 1 Leistung               | W    |
| 33555458     | Input DC 2 voltage - de: Eingang DC 2 Spannung             | V    |
| 33555457     | Input DC 2 current - de: Eingang DC 2 Strom                | A    |
| 33555459     | Input DC 2 power - de: ingang DC 2 Leistung                | W    |
| 67110400     | Mains frequency - de: Netzfrequenz                         | Hz   |
| 67110656     | Cosine phi - de: Cosinus phi                               | -    |
| 67110144     | Regulate to - de: Abregeln auf                             | %    |
| 67109378     | Voltage Phase 1 - de: Spannung Phase 1                     | V    |
| 67109377     | Current Phase 1 - de: Strom Phase 1                        | A    |
| 67109379     | Power Phase 1 - de: Leistung Phase 1                       | W    |
| 67109634     | Voltage Phase 2 - de: Spannung Phase 2                     | V    |
| 67109633     | Current Phase 2 - de: Strom Phase 2                        | A    |
| 67109635     | Power Phase 2 - de: Leistung Phase 2                       | W    |
| 67109890     | Voltage Phase 3 - de: Spannung Phase 3                     | V    |
| 67109889     | Current Phase 3 - de: Strom Phase 3                        | A    |
| 67109891     | Power Phase 3 - de: Leistung Phase 3                       | W    |

So in order to query the total yield, the total AC and DC power as well als the
DC powers of the two strings (1 and 2) and the phase powers, the query would
looked like the following:

```
http://<ip-of-your-inverter>/api/dxs.json?dxsEntries=251658753&dxsEntries=67109120&dxsEntries=33556736&dxsEntries=33555203&dxsEntries=33555459&dxsEntries=67109379&dxsEntries=67109635&dxsEntries=67109891
```

Executing this with curl produces the following output:

```
❯ curl -v "http://192.168.0.101/api/dxs.json?dxsEntries=251658753&dxsEntries=67109120&dxsEntries=33556736&dxsEntries=33555203&dxsEntries=33555459&dxsEntries=67109379&dxsEntries=67109635&dxsEntries=67109891"
*   Trying 192.168.0.101:80...
* Connected to 192.168.0.101 (192.168.0.101) port 80
> GET /api/dxs.json?dxsEntries=251658753&dxsEntries=67109120&dxsEntries=33556736&dxsEntries=33555203&dxsEntries=33555459&dxsEntries=67109379&dxsEntries=67109635&dxsEntries=67109891 HTTP/1.1
> Host: 192.168.0.101
> User-Agent: curl/8.9.1
> Accept: */*
>
* Request completely sent off
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Content-Type: text/plain
< Expires: Sun, 02 Jan 2000 11:11:11 GMT
<
* shutting down connection #0
{"dxsEntries":[{"dxsId":251658753,"value":41731.171875},{"dxsId":67109120,"value":2222.075928},{"dxsId":33556736,"value":2390.199707},{"dxsId":33555203,"value":1106.056152},{"dxsId":33555459,"value":1284.143555},{"dxsId":67109379,"value":744.989014},{"dxsId":67109635,"value":741.761047},{"dxsId":67109891,"value":735.325867}],"session":{"sessionId":0,"roleId":0},"status":{"code":0}}⏎
```

The returned data format is json even if it claims to be text/plain. When
inspecting the output more carefully we can see that we got the measurements for
all requested metrics:

```jsonc
{
  "dxsEntries": [
    // answers are returned in the "dxsEntries" array
    { "dxsId": 251658753, "value": 41731.171875 }, // Total yield
    { "dxsId": 67109120, "value": 2222.075928 }, // Total AC output power
    { "dxsId": 33556736, "value": 2390.199707 }, // Total DC input power
    { "dxsId": 33555203, "value": 1106.056152 }, // Input DC 1 Power
    { "dxsId": 33555459, "value": 1284.143555 }, // Input DC 2 power
    { "dxsId": 67109379, "value": 744.989014 }, // Power Phase 1
    { "dxsId": 67109635, "value": 741.761047 }, // Power Phase 2
    { "dxsId": 67109891, "value": 735.325867 } // Power Phase 3
  ],
  "session": { "sessionId": 0, "roleId": 0 },
  "status": { "code": 0 }
}
```

To make these values easier to use i want them to be in an Influx DB so that i
can work with them. I use
[Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) to ingest
this data. A possible configuration could look like this:

```toml
[[inputs.http]]
  urls = [
    "http://192.168.0.101/api/dxs.json?dxsEntries=251658753&dxsEntries=67109120&dxsEntries=33556736&dxsEntries=33555203&dxsEntries=33555459&dxsEntries=67109379&dxsEntries=67109635&dxsEntries=67109891"
  ]
  method = "GET"
  data_format = "json"
  json_query = "dxsEntries"
  name_override = "pv_generation"
  tag_keys = ["dxsId"]
  tagexclude = ["url"]

[[processors.starlark]]
  namepass = ["pv_generation"]
  source = '''
dxs_map = {
    "251658753": "total_yield",
    "67109120": "ac_output_power",
    "33556736": "dc_input_power",
    "33555203": "input_dc1_power",
    "33555459": "input_dc2_power",
    "67109379": "power_phase_1",
    "67109635": "power_phase_2",
    "67109891": "power_phase_3"
}

def apply(metric):
    dxsId = metric.tags.get("dxsId")
    if dxsId in dxs_map:
        val = metric.fields.get("value")
        val_key = dxs_map[dxsId]
        new_metric = Metric(metric.name)
        new_metric.time = metric.time
        new_metric.fields[val_key] = val
        for k,v in metric.tags.items():
            if k != "dxsId":
                new_metric.tags[k] = v
        return new_metric
    return None
'''
```

First it uses the http input to read the data from the http api. The output of
the http plugin itself is not that descriptive so i added a processor that
renames the fields from the dxs id's to more descriptive names.
