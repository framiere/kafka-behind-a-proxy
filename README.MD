# Testing kafka-proxy

In order to test [kafka-proxy](https://github.com/grepplabs/kafka-proxy) we are simulating 2 networks

* ccloud
* onpremise

The main command is the following:

```yaml
    command: server --bootstrap-server-mapping=broker-1:9092,0.0.0.0:9092,kafka-proxy:9092 --bootstrap-server-mapping=broker-2:9092,0.0.0.0:9093,kafka-proxy:9092 --bootstrap-server-mapping=broker-3:9092,0.0.0.0:9094,kafka-proxy:9092
```

Let's decompose the first argument

`--bootstrap-server-mapping=broker-1:9092,0.0.0.0:9092,kafka-proxy:9092`

1. We are bootstraping the `broker-1:9092` kafka server
2. We are asking to proxy `broker-1:9092` via `0.0.0.0:9092` 
3. We are announcing this proxy to clients as `kafka-proxy:9092`

To test:

`docker-compose exec kafka-client kafka-console-producer --broker-list kafka-proxy:9092 --topic example`

type a lovely `hello world` and a firm [Enter].

And then consume it back via 

```sh
docker-compose exec kafka-client kafka-console-producer --bootstrap-server kafka-proxy:9092 --topic example --from-beginning
```