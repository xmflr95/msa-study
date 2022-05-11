# Podman Network
ELK 구성 중 podman에서 host와 pod(하나 이상의 컨테이너 모음)간의 네트워크 연결이 필요해짐
pod에 네트워크를 연결하는 방식으로 해결
pod 내부 container의 통신은 localhost(127.0.0.1)로 할 수 있고 
외부 통신은 container host의 network(virtual network)가 pod에 연결함으로 container host ip주소로 접근 가능해짐

- 구성 순서
  1. Dockerfile 생성(+ shell 구성), container 이미지 생성
  2. podman network 생성
  3. podman pod 생성(+ 네트워크 연결)
  4. podman container 생성(+ pod에 연결)

## 1. Dockerfile 생성

> Elasticsearch
```shell
FROM docker.elastic.co/elasticsearch/elasticsearch:7.17.0  
COPY ./config/elasticsearch.yml  /usr/share/elasticsearch/config/elasticsearch.yml
USER root
RUN set -x && apt-get -y update
RUN set -x && apt-get -y upgrade
RUN set -x && apt-get install -y apt-utils
RUN set -x && apt-get install -y vim
RUN set -x && apt-get install -y iputils-ping
RUN set -x && apt-get install -y net-tools

RUN /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch analysis-nori
```
+) elasticsearch container 실행 시 `ES_JAVA_OPTS`로 메모리 설정이 필요(추천은 시스템의 50%정도이나 적절하게 맞추면 될듯함)
```shell
podman run -idt --pod pod-elk --name elk-e -e "discovery.type=single-node" -e TZ=Asia/Seoul -e "ES_JAVA_OPTS=-Xms2g -Xmx2g" elk-e:latest
```

> Logstash
```shell
FROM docker.elastic.co/logstash/logstash:7.17.0

USER root
RUN set -x && apt-get update && apt-get -y upgrade && apt-get install -y apt-utils
RUN set -x && apt-get update && apt-get install -y vim
RUN set -x && apt-get install -y iputils-ping

COPY ./logstash.conf /usr/share/logstash/config/logstash.conf
COPY ./logstash.yml /usr/share/logstash/config/logstash.yml
COPY ./logstash.conf /usr/share/logstash/pipeline/logstash.conf

RUN chmod 644 /usr/share/logstash/config/logstash.*
RUN chown -R logstash /usr/share/logstash/config/logstash.*
```

> Kibana
```shell
FROM kibana:7.17.0

COPY ./config/kibana.yml /usr/share/kibana/config/kibana.yml

USER root

RUN set -x && apt-get -y update
RUN set -x && apt-get -y upgrade
RUN set -x && apt-get install -y apt-utils
RUN set -x && apt-get install -y vim
RUN set -x && apt-get install -y iputils-ping
RUN set -x && apt-get install -y net-tools

USER kibana
```

## 2. podman network 생성
podman network 생성
```shell
# podman network create [options] ${network_name}
podman network create test-net
# /${USER}/.config/cni/net.d/test-net.conflist로 network 설정 생성
podman network inspect test-net	# inspect 명령어로 상세보기
# inspect 결과
[
    {
        "cniVersion": "0.4.0",
        "name": "test-net",
        "plugins": [
            {
                "bridge": "cni-podman2",
                "hairpinMode": true,
                "ipMasq": true,
                "ipam": {
                    "ranges": [
                        [
                            {
                                "gateway": "10.89.1.1",
                                "subnet": "10.89.1.0/24"
                            }
                        ]
                    ],
                    "routes": [
                        {
                            "dst": "0.0.0.0/0"
                        }
                    ],
                    "type": "host-local"
                },
                "isGateway": true,
                "type": "bridge"
            },
            {
                "capabilities": {
                    "portMappings": true
                },
                "type": "portmap"
            },
            {
                "backend": "",
                "type": "firewall"
            },
            {
                "type": "tuning"
            }
        ]
    }
]
```

## 3. podman pod 생성(+ 네트워크 연결)
container간의 통신을 위해 pod을 생성한다.
- `--network network` 명령어로 네트워크에 연결
```shell
# podman pod create [options]
# 기본 생성 + name 설정
podman pod create --name pod-elk
# pod을 network에 연결
podman pod create --name pod-elk --network network
# pod <-> container 포트 미리 설정
# ex) 9300,18000: elasticsearch, 18001: kibana, 18002: logstash
podman pod create --name pod-elk --network network -p 9300:9300 -p 18000:18000 -p 18001:18001 -p 18002:18002

# pod 상세 정보 (inspect)
podman pod inspect pod-elk # [pod_name]
# 실행 결과
{
     "Id": "513002097c9947cf7431de06cd86c6c98e4c93d9dc7db2667e37537e1242f52b",
     "Name": "pod-elk",
     "Created": "2022-05-11T10:52:53.74175547+09:00",
     "CreateCommand": [
          "podman",
          "pod",
          "create",
          "--name",
          "pod-elk",
          "--network",
          "network",
          "-p",
          "9300:9300",
          "-p",
          "18000:18000",
          "-p",
          "18001:18001",
          "-p",
          "18002:18002"
     ],
     "State": "Running",
     "Hostname": "pod-elk",
     "CreateCgroup": true,
     "CgroupParent": "/libpod_parent",
     "CgroupPath": "/libpod_parent/513002097c9947cf7431de06cd86c6c98e4c93d9dc7db2667e37537e1242f52b",
     "CreateInfra": true,
     "InfraContainerID": "1f993317df8a7b932b5c0be7d859c8a5b067b6a91fcec0067309bfbd6ae6c396",
     "InfraConfig": {
          "PortBindings": {
               "18000/tcp": [
     {
          "HostIp": "",
          "HostPort": "18000"
     }
],
               "18001/tcp": [
     {
          "HostIp": "",
          "HostPort": "18001"
     }
],
               "18002/tcp": [
     {
          "HostIp": "",
          "HostPort": "18002"
     }
],
               "9300/tcp": [
     {
          "HostIp": "",
          "HostPort": "9300"
     }
]
          },
          "HostNetwork": false,
          "StaticIP": "",
          "StaticMAC": "",
          "NoManageResolvConf": false,
          "DNSServer": null,
          "DNSSearch": null,
          "DNSOption": null,
          "NoManageHosts": false,
          "HostAdd": null,
          "Networks": [
               "network"
          ],
          "NetworkOptions": null
     },
     "SharedNamespaces": [
          "net",
          "uts",
          "ipc"
     ],
     "NumContainers": 1,
     "Containers": [
          {
							# infra 생성 설정을 건드리지 않으면 자동으로 생성되는 듯함
               "Id": "1f993317df8a7b932b5c0be7d859c8a5b067b6a91fcec0067309bfbd6ae6c396",
               "Name": "513002097c99-infra",
               "State": "running"
          }
     ]
}
```

## 4. podman container 생성(+ pod에 연결)
- podman run 명령어 실행(or shell 실행)
- `--pod [pod_name]` 명령어로 생성해둔 pod와 연결
- pod에 연결 시 pod 내부의 container는 localhost로 통신 가능
```shell
# Elasticsearch
podman run -idt --pod pod-elk --name elk-e -e "discovery.type=single-node" -e TZ=Asia/Seoul -e "ES_JAVA_OPTS=-Xms2g -Xmx2g" elk-e:latest
# Logstash
podman run -idt --pod pod-elk -e TZ=Asia/Seoul --name elk-l elk-l:latest
# Kibana
podman run -idt --pod pod-elk -e TZ=Asia/Seoul --name elk-k elk-k:latest
```

모두 정상적으로 network에 연결되면 아래처럼 pod에 연결된 컨테이너로 나타남
`podman pod inspect [pod_name]` 명령어 실행
```shell
# ...inspect content
"NumContainers": 4,
	"Containers": [
		{
			"Id": "1f993317df8a7b932b5c0be7d859c8a5b067b6a91fcec0067309bfbd6ae6c396",
			"Name": "513002097c99-infra",
			"State": "running"
		},
		{
			"Id": "260e136d060ca922f9436b92ac3cd8c7a81c96e6fc6a6047be77f239280047d9",
			"Name": "elk-l",
			"State": "running"
		},
		{
			"Id": "56e7c1437dc7561c8ab4dea16123fd17ee995fa9ba1b109be87e914d27a85788",
			"Name": "elk-k",
			"State": "running"
		},
		{
			"Id": "97128e7120e5d709eef558dbb6d01fa2d4d29f4f72ac9a1f7413ced360d9855d",
			"Name": "elk-e",
			"State": "running"
		}
]
```


##### ELK 컨테이너 설정
> elasticsearch.yml
```yml
network.host: 0.0.0.0
http.port: 18000	# elasticsearch 내부 포트 설정
```
> logstash.conf
```yml
input {
  tcp {
    port => 18002
     codec => json_lines
  }
}

filter {
  json {
    source => "message"
  }
}

output {
  elasticsearch {
    # hosts 설정 시 localhost:${container port}로 설정 가능해짐
    hosts => ["http://localhost:18000"]
    index => "logstash-%{+YYYY.MM.dd}"
    ecs_compatibility => disabled
  }
  stdout {
    codec => rubydebug
  }
}
```
> kibana.yml
```yml
server.host: "0.0.0.0"
server.port: 18001
server.shutdownTimeout: "5s"
#elasticsearch.hosts: [ "http://elasticsearch:9200" ]
# logstash와 동일하게 localhost로 설정
elasticsearch.hosts: [ "http://localhost:18000" ]
monitoring.ui.container.elasticsearch.enabled: true
```
