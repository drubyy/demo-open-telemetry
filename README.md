## Purpose
Project muốn check metrics (API chạy nào được gọi bao nhiêu lần ?...) và trace (API chạy flow thế nào ? đi qua những đâu ? từng đoạn đó chạy trong duration bao lâu ?...)

<b> => Apply flow: app -> otel-collector -> Trace UI (Prometheus / OpenObserve / Jaeger / ...) </b>

## How to demo?
##### Step 1:
```
docker compose up
```

##### Step 2:
```
go to http://localhost:8088/ping

or

curl http://localhost:8088/ping
```

##### Step 3:
Access to Jaeger / Prometheus / OpenObserve để xem trace / metric
