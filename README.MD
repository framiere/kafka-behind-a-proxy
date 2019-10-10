# Testing kafka-proxy

We are simulating 2 networks

* `ccloud` where Kafka resides
* `onpremise` where are the clients

`onpremise` can't reach `ccloud`, and thus needs a proxy with can connect to both.

The problem relies in the medata protocol, and thus a simple haproxy would not suffice. We need to proxy at Level7 to change the ip announced by Kafka by the ip of the proxy (kafka-proxy approach), or trick the metadata sent to go through the proxy (ha-proxy approach).


## Kafka-proxy 

Enter https://github.com/grepplabs/kafka-proxy.


The main command is the following:

```yaml
    command: server --bootstrap-server-mapping=broker-1:9092,0.0.0.0:9092,kafka-proxy:9092 --bootstrap-server-mapping=broker-2:9092,0.0.0.0:9093,kafka-proxy:9092 --bootstrap-server-mapping=broker-3:9092,0.0.0.0:9094,kafka-proxy:9092
```

Let's decompose the first argument

`--bootstrap-server-mapping=broker-1:9092,0.0.0.0:9092,kafka-proxy:9092`

1. We are bootstraping the `broker-1:9092` kafka server
2. We are asking to proxy `broker-1:9092` via `0.0.0.0:9092` 
3. We are announcing this proxy to clients as `kafka-proxy:9092`

To test, run the stack with `docker-compose up -d` then write something into a topic

```sh
docker-compose exec kafka-client kafka-console-producer --broker-list kafka-proxy:9092 --topic example`
```

type a lovely `hello world` and a firm [Enter].

And then consume it back via 

```sh
docker-compose exec kafka-client kafka-console-producer --bootstrap-server kafka-proxy:9092 --topic example --from-beginning
```

## HAProxy

To test, run the stack with `docker-compose up -d`

in `kafka-client` enter docker 

```
docker-compose exec kafka-client bash
echo `host haproxy-1 | awk '{print $4}'` broker-1 >> /etc/hosts
echo `host haproxy-2 | awk '{print $4}'` broker-2 >> /etc/hosts
echo `host haproxy-3 | awk '{print $4}'` broker-3 >> /etc/hosts
echo example | kafka-client kafka-console-producer --broker-list haproxy-1:9092 --topic example
kafka-client kafka-console-producer --bootstrap-server haproxy-1:9092 --topic example --from-beginning
```