# docker nginx vhost

![image](https://github.com/pySatellite/docker-nginx-vhost/assets/87309910/878eaf6a-18bc-4467-8b3f-5086de8ff3a1)

# load-balancing
- https://www.nginx.com/resources/glossary/load-balancing/
  
![image](https://github.com/Jaelinny/docker-nginx-vhost/assets/148875683/f1013b3f-9a5f-4de5-85d6-518347e8c230)


<br/>
<br/>

# Dockerfile

## 1. serv-a/serv-b/lb Dockerfile 생성
```
$ pwd
/home/nori/code/docker-nginx-vhost

$ cd ../serv-a, serv-b
$ cat Dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html

$ cd lb
$ cat Dockerfile
FROM nginx
COPY config/default.conf /etc/nginx/conf.d/
```

## 2. lb/config/default.conf 코드 수정
```
upstream serv {
        server n1-1:80;
        server n1-2:80;
}
server {
        listen 80;

        location /
        {
                proxy_pass http://serv;
        }
}
```

## 3. Build & RUN

### step 1
- docker rm * rmi
```
$ sudo docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### step 2
```
$ docker run -itd -p 8002:80 --name serv-a nginx
$ docker run -itd -p 8003:80 --name serv-b nginx
$ docker run -itd -p 8001:80 --name lb nginx:latest

$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                   NAMES
34fd7fbc0ba1   nginx:latest   "/docker-entrypoint.…"   16 seconds ago   Up 15 seconds   0.0.0.0:8001->80/tcp, :::8001->80/tcp   lb
efbdd20605f0   nginx          "/docker-entrypoint.…"   25 seconds ago   Up 24 seconds   0.0.0.0:8003->80/tcp, :::8003->80/tcp   serv-b
4d25a85c1c16   nginx          "/docker-entrypoint.…"   34 seconds ago   Up 33 seconds   0.0.0.0:8002->80/tcp, :::8002->80/tcp   serv-a
```

### step 1,2 by Dockerfile
- build & push
```
$ tree
.
├── README.md
├── lb
│   ├── Dockerfile
│   └── config
│       └── default.conf
├── serv-a
│   ├── Dockerfile
│   └── index.html
└── serv-b
    ├── Dockerfile
    └── index.html

$ cd lb
$ sudo docker build -t taengguu/lb:0.1.0 .
$ sudo docker push taengguu/lb:0.1.0

$ cd ../serv-a
$ sudo docker build -t taengguu/serv-a:0.1.0 .
$ sudo docker push taengguu/serv-a:0.1.0

$ cd ../serv-b
$ sudo docker build -t taengguu/serv-b:0.1.0 .
$ sudo docker push taengguu/serv-b:0.1.0
```

### step 3 
- run & 네트워크 생성하여 서버 묶기
```
$ sudo docker network create -d bridge lb-net

$ sudo docker images
$ sudo docker run -d --name serv-a --network lb-net taengguu/serv-a:0.1.0
$ sudo docker run -d --name serv-b --network lb-net taengguu/serv-b:0.1.0
$ sudo docker run -d -p 8001:80 --name lb --network lb-net taengguu/lb:0.1.0

$ docker network inspect lb-net
[
    {
        "Name": "lb-net",
        "Id": "440f5937e12798a7e82993af754bcb8ab11fe65d7b7df9e8493dce6aa5d17937",
        "Created": "2024-02-15T01:12:21.015312758+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0ce4371ccab6b420adb06c2f89aa879f77a291815329d0ee3e0c663e8c0ff874": {
                "Name": "serv-b",
                "EndpointID": "0eaa2447cf45a9fca4fe8be7ad7f3ee6a3f3ffe67a7dacecf2659d6cadae0dbd",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "c5a1a6ac44b4cb3ccd1780e8195efff4e98015b50b8a9ad6671961d975d45016": {
                "Name": "lb",
                "EndpointID": "93ecd256f3a3ba24505aff6d915e9bc0b4f93667ac824901ef830798a6c74670",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "d44b3688c8ccf7b3938f86c37c236acf214b9fae20dfd4ccdbcf51c6d580287f": {
                "Name": "serv-a",
                "EndpointID": "8af5a14c9f0db020ef28071a1041f19ed7f89ae25613eb4d1630c628d2d67702",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

## 4. 테스트

- test 1
```
$ sudo docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS                                   NAMES
1d50274416dd   taengguu/lb:0.1.0       "/docker-entrypoint.…"   6 seconds ago    Up 5 seconds    0.0.0.0:8001->80/tcp, :::8001->80/tcp   lb
9eb365f17ac2   taengguu/serv-b:0.1.0   "/docker-entrypoint.…"   30 seconds ago   Up 29 seconds   80/tcp                                  serv-b
bef7ae198520   taengguu/serv-a:0.1.0   "/docker-entrypoint.…"   52 seconds ago   Up 51 seconds   80/tcp                                  serv-a
```

- test 2
```
$ curl http://localhost:8001
<h1>A</h1>
$ curl http://localhost:8001
<h1>B</h1>
$ curl http://localhost:8001
<h1>A</h1>
```

## 출력 화면 <🔄f5>
![image](https://github.com/Jaelinny/docker-nginx-vhost/assets/148875683/e2d4ae4e-bfd7-4ebc-981c-3d02b9e57ace)


![image](https://github.com/Jaelinny/docker-nginx-vhost/assets/148875683/662d4aac-e0d9-4555-a586-fdf88bb7a9a4)

