# Network monitoring

## Stack

- Opentelerimetry : LGTM

## Monitor server

- otel-lgtm container ( loki, grafana, tempo, prometheus )

## Client ( supervised server)

- clone repo
- pass to root user
- set `export OTEL_EXPORTER_OTLP_ENDPOINT=http://192.168.56.16:4318` 
- run exemple
