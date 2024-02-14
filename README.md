# docker nginx vhost

![image](https://github.com/pySatellite/docker-nginx-vhost/assets/87309910/878eaf6a-18bc-4467-8b3f-5086de8ff3a1)

# load-balancing
https://www.nginx.com/resources/glossary/load-balancing/

# hello Dockerfile

## A. serv-a/serv-b/lb 각각 Dockerfile 을 생성
```
$ pwd
/home/nori/code/docker-nginx-vhost/serv-b
$ vi Dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html

$ pwd
/home/nori/code/docker-nginx-vhost/serv-a
$ vi Dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html

$ pwd
/home/nori/code/docker-nginx-vhost/lb
$ vi Dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html
```

## B~C. Build & RUN
```
$ sudo docker build -t n-s-1:0.1.0 .
$ sudo docker build -t n-s-2:0.1.0 .
$ sudo docker build -t n-s-3:0.1.0 .

$ sudo docker images
REPOSITORY        TAG       IMAGE ID       CREATED             SIZE
n-s-3             0.1.0     fb0f0a51c3da   16 minutes ago      187MB
n-s-2             0.1.0     f329679ae831   40 minutes ago      187MB
n-s-1             0.1.0     9c1976011eea   43 minutes ago      187MB

$ sudo docker run -d --name s1-1 -p 9001:80 n-s-1:0.1.0
$ sudo docker run -d -p 9002:80 n-s-2:0.1.0
$ sudo docker run -d -p 9003:80 n-s-3:0.1.0
```

## D. 네트워크 생성하여 serv-a, serv-b, lb 를 묶기
```
$ sudo docker network create docker-build  // 네트워크 생성

$  docker network inspect docker-build
[
    {
        "Name": "docker-build",
        "Id": "ffd8d4bed5551a210152a134456ae481acac6da9131a8e4cfbe8efbb9670fa00",
        "Created": "2024-02-13T16:51:40.524975693+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
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
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

// 생성 된 docker-build 네트워크에 serv-a, serv-b, lb 묶기!
$ sudo docker network connect docker-build serv-a
$ sudo docker network connect docker-build serv-b
$ sudo docker network connect docker-build lb

$ sudo docker inspect docker-build

[
    {
        "Name": "docker-build",
        "Id": "ffd8d4bed5551a210152a134456ae481acac6da9131a8e4cfbe8efbb9670fa00",
        "Created": "2024-02-13T16:51:40.524975693+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
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
            "510e9fc8cb026d95307540345d303201f588aec3dcd658a6c92823e48dddea0f": {
                "Name": "serv-b",
                "EndpointID": "76fbc533acabc13bbf69920b704324c251a7d947c60a374087bba58aad2e637c",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            },
            "6454337e4a48a6ac5f1d6d69e047ab7de0c20e0274c3f8bb8c4ab73d4cb41822": {
                "Name": "serv-a",
                "EndpointID": "b8eff8a444aa72840a21c562c915b1d7eb19f223d35ee46f55c919e8dfc8eff9",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "8c43dcb29c58dc1e6be39137c094e3fc914b6ab7b4c183120e7a691e3ddcb4cf": {
                "Name": "lb",
                "EndpointID": "2d2a380c9d73e948b7af53811b43d7b234a9cf882d6610f5153643654c9cad09",
                "MacAddress": "02:42:ac:13:00:04",
                "IPv4Address": "172.19.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

## E. 잘 동작하는지 확인 및 README.md 에 설명 작성


- ![image](https://github.com/Jaelinny/docker-nginx-vhost/assets/148875683/ab2c0540-56b1-48d3-a1cd-f851d6558cce)



