## Grafana HTTP JSON Datasource - a generic backend datasource

This is a generic http json data source. It is based on the official [simple json datasource](https://github.com/grafana/simple-json-datasource)
plugin by grafana labs. All features are also submitted as PR to the official plugin.

Your backend needs to implement 4 urls:

 * `/` should return 200 ok. Used for "Test connection" on the datasource config page.
 * `/search` used by the find metric options on the query tab in panels.
 * `/query` should return metrics based on input.
 * `/annotations` should return annotations.
 * `/tag-keys` should return tag keys for ad hoc filters.
 * `/tag-values` should return tag values for ad hoc filters.

## Installation

To install this plugin using the `grafana-cli` tool:
```
sudo grafana-cli plugins install exozet-http-json-datasource
sudo service grafana-server restart
```
See [here](https://grafana.com/plugins/grafana-http-json-datasource/installation) for more
information.

If you want to install the latest version from github, run:

```
sudo grafana-cli --pluginUrl https://github.com/grafana/simple-json-datasource/archive/master.zip plugins install exozet-http-json-datasource 
sudo service grafana-server restart
```


### Example backend implementations

- https://github.com/bergquist/fake-simple-json-datasource
- https://github.com/smcquay/jsonds

### Query API

Example `timeserie` request
``` javascript
{
  "panelId": 1,
  "range": {
    "from": "2016-10-31T06:33:44.866Z",
    "to": "2016-10-31T12:33:44.866Z",
    "raw": {
      "from": "now-6h",
      "to": "now"
    }
  },
  "rangeRaw": {
    "from": "now-6h",
    "to": "now"
  },
  "interval": "30s",
  "intervalMs": 30000,
  "targets": [
     { "target": "upper_50", "refId": "A", "type": "timeserie" },
     { "target": "upper_75", "refId": "B", "type": "timeserie" }
  ],
  "adhocFilters": [
    "key": "City"
    "operator": "=",
    "value": "Berlin"
  ]
  "format": "json",
  "maxDataPoints": 550
}
```

Example `timeserie` response
``` javascript
[
  {
    "target":"upper_75", // The field being queried for
    "datapoints":[
      [622,1450754160000],  // Metric value as a float , unixtimestamp in milliseconds
      [365,1450754220000]
    ]
  },
  {
    "target":"upper_90",
    "datapoints":[
      [861,1450754160000],
      [767,1450754220000]
    ]
  }
]
```

If the metric selected is `"type": "table"`, an example `table` response:
```json
[
  {
    "columns":[
      {"text":"Time","type":"time"},
      {"text":"Country","type":"string"},
      {"text":"Number","type":"number"}
    ],
    "rows":[
      [1234567,"SE",123],
      [1234567,"DE",231],
      [1234567,"US",321]
    ],
    "type":"table"
  }
]
```

### Annotation API

The annotation request to the backend is a POST request to
the /annotations endpoint in your datasource. The JSON request body looks like this:
``` javascript
{
  "range": {
    "from": "2016-04-15T13:44:39.070Z",
    "to": "2016-04-15T14:44:39.070Z"
  },
  "rangeRaw": {
    "from": "now-1h",
    "to": "now"
  },
  "annotation": {
    "name": "deploy",
    "datasource": "HTTP JSON Datasource",
    "iconColor": "rgba(255, 96, 96, 1)",
    "enable": true,
    "query": "#deploy"
  }
}
```

Grafana expects a response containing an array of annotation objects in the
following format:

``` javascript
[
  {
    annotation: annotation, // The original annotation sent from Grafana.
    time: time, // Time since UNIX Epoch in milliseconds. (required)
    title: title, // The title for the annotation tooltip. (required)
    tags: tags, // Tags for the annotation. (optional)
    text: text // Text for the annotation. (optional)
  }
]
```

Note: If the datasource is configured to connect directly to the backend, you
also need to implement an OPTIONS endpoint at /annotations that responds
with the correct CORS headers:

```
Access-Control-Allow-Headers:accept, content-type
Access-Control-Allow-Methods:POST
Access-Control-Allow-Origin:*
```

### Search API

Example request
``` javascript
{ target: 'upper_50' }
```

The search api can either return an array or map.

Example array response
``` javascript
["upper_25","upper_50","upper_75","upper_90","upper_95"]
```

Example map response
``` javascript
[ { "text" :"upper_25", "value": 1}, { "text" :"upper_75", "value": 2} ]
```

### Tag Keys API

Example request
``` javascript
{ }
```

The tag keys api returns:
```javascript
[
    {"type":"string","text":"City"},
    {"type":"string","text":"Country"}
]
```

### Tag Values API

Example request
``` javascript
{"key": "City"}
```

The tag values api returns:
```javascript
[
    {'text': 'Eins!'},
    {'text': 'Zwei'},
    {'text': 'Drei!'}
]
```

### Dev setup

This plugin requires node 6.10.0

`npm install -g yarn`
`yarn install`
`npm run build`

### Changelog

1.4.0

- added tag-keys + tag-values api
- added adHocFilters parameter to query body
- initial fork from official [simple json datasource](https://github.com/grafana/simple-json-datasource)

### If using Grafana 2.6
NOTE!
for grafana 2.6 please use [this version](https://github.com/grafana/simple-json-datasource/commit/b78720f6e00c115203d8f4c0e81ccd3c16001f94) of the official simple-json-datasource plugin.

Copy the data source you want to /public/app/plugins/datasource/. Then restart grafana-server. The new data source should now be available in the data source type dropdown in the Add Data Source View.
