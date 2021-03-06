# Monitored Splunk + fluent-bit integration via HEC

## Functionality

This composition configures fluent-bit to read out memory and CPU metrics,
transform them and send them to Splunk via the HTTP Event Collector (HEC). The
main Splunk instance contains an example dashboard displaying the incoming
metrics.

End-to-end data pipeline visibility is provided by exposing metrics in
Prometheus format for fluent-bit, envoy and the Splunk forwarders.

## Notable features

 - Backpressure-sensitive heartbeat
 - Transport to remote Splunk platform over HTTP
 - Prometheus metrics for fluent-bit, envoy and splunkforwarder
 - Grafana dashboards

## Versions

 - [v1.2.0 - Replace nginx with envoy](https://github.com/michiel/docker-compose-splunk-fluentbit/tree/v1.2.0)
 - [v1.1.0 - Replace Splunk UF with nginx, transport events over HTTP to HEC](https://github.com/michiel/docker-compose-splunk-fluentbit/tree/v1.1.0)
 - [v1.0.0 - Ship HEC-ready events to Splunk UF, transport events via Splunk2Splunk TCP](https://github.com/michiel/docker-compose-splunk-fluentbit/tree/v1.0.0)

## Running

Bring up the containers by running,

    docker-compose up

To bring down and clean up the containers run,

    docker-compose down

## UI   

|                | URL                                              | Username  | Password |
|----------------|--------------------------------------------------|-----------|----------|
| Splunk UI      | [http://localhost:8000/](http://localhost:8000/) | admin     | admin    |
| Prometheus     | [http://localhost:9090/](http://localhost:9090/) | admin     | admin    |
| Grafana        | [http://localhost:3000/](http://localhost:3030/) | admin     | admin    |
| Envoy          | [http://localhost:8001/](http://localhost:8001/) | N/A       | N/A      |
| mtail          | [http://localhost:3903/](http://localhost:3903/) | N/A       | N/A      |
| mtail          | [http://localhost:3904/](http://localhost:3904/) | N/A       | N/A      |

## Composition

This docker-compose image uses,

 - [A public, official splunk enterprise image](https://hub.docker.com/r/splunk/splunk/)
 - [A public, official splunk enterprise universalforwarder image](https://hub.docker.com/r/splunk/universalforwarder/)
 - [A public, official fluentbit image](https://hub.docker.com/r/fluent/fluent-bit/)
 - [A public, official prometheus image](https://hub.docker.com/r/prom/prometheus)
 - [A public, official grafana image](https://hub.docker.com/r/grafana/grafana)
 - [A public, official envoy image](https://hub.docker.com/r/envoyproxy/envoy-alpine)
 - [A public, unofficial mtail image](https://hub.docker.com/r/jaymedh/docker-mtail/)

### Components

The Splunk HF, IDX and SHC components are all run by the main `splunk` image.

![fluent-bit Splunk HEC](/resource/splunk-fluentbit-components.png?raw=true "fluent-bit Splunk HEC")

### Data flow sequence

![fluent-bit Splunk HEC](/resource/splunk-fluentbit-sequence.png?raw=true "fluent-bit Splunk HEC")

## fluent-bit pipeline

![fluent-bit pipeline](/resource/fluent-bit-pipeline.png?raw=true "fluent-bit pipeline")

Main [`fluent-bit.conf`](/volumes/fluent-bit-etc/fluent-bit.conf)

It uses the fluent-bit `http` output plugin with the plugin's Basic
Authentication. See [Format events for HTTP Event Collector](https://docs.splunk.com/Documentation/Splunk/7.0.3/Data/FormateventsforHTTPEventCollector)
for the Splunk HEC documentation.

## Shell

For a shell on the containers, run the commands below.

    ./script/shell-splunk.sh
    ./script/shell-splunkforwarder01.sh
    ./script/shell-splunkforwarder02.sh
    ./script/shell-fluentbit.sh
    ./script/shell-prometheus.sh
    ./script/shell-grafana.sh
    ./script/shell-mtail01.sh
    ./script/shell-mtail02.sh

## Testing

To test the HEC, expose a forwarder port 8088 on localhost and run the cURL
command,

```
curl -u 'x:3e6ffd12-0f69-46bb-ad0d-71cffb661a0d' -X POST -d'
{
    "event": {
        "event_key1": "value1",
        "event_key1": "value1"
    },
    "fields": {
        "field_key1": "value1",
        "field_key1": "value1"
    }
}' http://localhost:8088/services/collector/event
```

If the interface is up and running, The expected response is,

```
{"text":"Success","code":0}
```

