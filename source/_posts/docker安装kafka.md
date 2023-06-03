## 下载镜像

```sh
docker pull wurstmeister/zookeeper
```

```sh
docker pull wurstmeister/kafka
```

## 启动镜像

### 启动Zookeeper

```sh
docker run -d --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime wurstmeister/zookeeper
```



```sh

docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.118.128:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.118.128:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka    

docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=10.0.16.11:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.0.16.11:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka  

docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=172.17.0.7:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.0.16.11:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka  


10.0.16.11

```

```sh
docker run -d \
    -p 9001:8080 \
    -v /opt/kafka-map/data:/usr/local/kafka-map/data \
    -e DEFAULT_USERNAME=admin \
    -e DEFAULT_PASSWORD=admin \
    --name kafka-map \
    --restart always dushixiang/kafka-map:latest
```

