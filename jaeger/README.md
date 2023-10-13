# Jaeger

This docker compose setup starts Jaeger and hotrod services.

To start the service run:

```console
docker compose up

use -d option to start the services in detached mode
```

### Ports
Below Ports are exposed by the services, you can always change them according to your need in the yml

| Port   | Description       |
|--------|-------------------|
| 6831   | Jaeger Agent Port |
| 16686  | Jaeger UI Port    |
| 8082   | Hotrod UI Port    |

### Infrastructure model


![Infrastructure model](.infragenie/infrastructure_model.png)

