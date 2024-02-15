# docker nginx vhost
![304601299-878eaf6a-18bc-4467-8b3f-5086de8ff3a1](https://github.com/papercrane55123/docker-nginx-vhost/assets/150432433/4e1b47ab-0550-4799-86ea-898f27cff012)

# 하나의 nginx 포트에만 접속할 수 있도록 제한하기(+ 네트워크 설정)
https://blog.naver.com/mini_crane_/223353642333

1) 환경설정
도커 이미지와 컨테이너가 아래와 같이 나타나도록 만들어놓자
```sh
$ sudo docker run -itd -p 8001:80 --name lb nginx
$ sudo docker run -itd -p 8002:80 --name serv-a nginx
$ sudo docker run -itd -p 8003:80 --name serv-b nginx
```

2) 작업하려는 디렉토리에 config 디렉토리를 만들고 default.conf 파일을 만들어 아래 내용을 입력한다.
```
upstream serv {
        server serv-a:80;
        server serv-b:80;
}

server {
        listen 80;

        location /
        {
                proxy_pass http://serv;
        }
}
```

3) 이를 cp 명령어 활용해 컨테이너 안의 default.conf 파일과 바꿔치기한다.
```sh
$ sudo docker cp config/default.conf lb:/etc/nginx/conf.d/
```
물론 컨테이터 안에서 직접 입력해도 좋다.

4) 네트워크 확인하기
현재 있는 네트워크를 확인해보자. 명령어는 아래와 같다.
```sh
$ sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
7e5a04fbd68d   bridge    bridge    local
84e8ee58bce2   host      host      local
1df93e512a5d   none      null      local
```

네트워크에 상태를 확인해보자. 명령어는 inspect 명령어를 사용
```sh

$ sudo docker network inspect abc
[
    {
        "Name": "abc",
        "Id": "f50e5abcce985126b5c5d7ba0aa91b26211c304451c03b90849da830a6312284",
        "Created": "2024-02-14T12:48:08.67870812+09:00",
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
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```
6) 네트워크 연결하기
```sh
$ sudo docker network connect abc(네트워크명) serv-a
$ sudo docker network connect abc(네트워크명) serv-b
// connect는 한번에 연결할 수 없다.
```

확인하기
```sh
$ sudo docker network inspect abc
[
    {
        "Name": "abc",
        "Id": "f50e5abcce985126b5c5d7ba0aa91b26211c304451c03b90849da830a6312284",
        "Created": "2024-02-14T12:48:08.67870812+09:00",
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
            "772a8271b64f3db5353ebbc27d5c5b64fceab0669f832ac8c71f27c5cbdbbaa0": {
                "Name": "lb",
                "EndpointID": "e76d7e0f91427724dc77c539cac20dd4bcfad714ae9da8a0f349de268e36c068",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "a6c0d5ea90f622121c6aa5c6432e0d5845c0f2ba2d4c5687c7371fc7d064059c": {
                "Name": "serv-b",
                "EndpointID": "7cea4b9df8d2042671ef2a1f8716709732e7cfaca0f57bb47770e32094c0ff85",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "e8b986b03de4a352d4c5e0593f9e3d5263903ad29c7d916671ed48c59ea6d8c1": {
                "Name": "serv-a",
                "EndpointID": "8b159e1f58cedc07c49bff30b2b523ade407afb4ea9cf505623fe9402377c049",
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
7) 서버 확인
```sh
$ curl http://localhost:8001/
<h1>B</h1>
$ curl http://localhost:8001/
<h1>A</h1>
```

# Dockerfile 생성 및 build하기
https://blog.naver.com/mini_crane_/223353541818
Dockerfile을 사용하는 이유는 컨테이너를 만든 후 작업하는 반복되는 번거로움을 줄이고, 이미지 생성시 사용하는 명령어들을 파일로 관리하기 위해 사용한다. 
즉, Dockerfile을 사용했을 때 몸으로 체감할 수 있는 장점은 따로 서버구성하기 위해 cp, -v 등을 사용할 필요가 없다는 것을 예로 들 수 있겠다.
1) nginx 기반으로 도커파일을 만들어보자
https://hub.docker.com/_/nginx
```
FROM nginx
COPY index.html /usr/share/nginx/html
```
2) 도커파일 실행(build)하기
```sh
$ sudo docker build -t 이미지명:TAG .
// $ sudo docker build -t ng-s-1:0.1.0 .

or

$ sudo docker build -t 이미지명:TAG -f 파일명 .
// $ sudo docker build -t ng-s-1:0.1.0 -f dockerfile .
```

실행결과
```sh
$ sudo docker build -t ng-s-1:0.1.0 .
[+] Building 1.1s (7/7) FINISHED                                                                                                                      docker:default
 => [internal] load build definition from Dockerfile                                                                                                            0.1s
 => => transferring dockerfile: 86B                                                                                                                             0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                                                                                 0.0s
 => [internal] load .dockerignore                                                                                                                               0.1s
 => => transferring context: 2B                                                                                                                                 0.0s
 => [1/2] FROM docker.io/library/nginx:latest                                                                                                                   0.4s
 => [internal] load build context                                                                                                                               0.2s
 => => transferring context: 49B                                                                                                                                0.0s
 => [2/2] COPY index.html /usr/share/nginx/html                                                                                                                 0.1s
 => exporting to image                                                                                                                                          0.1s
 => => exporting layers                                                                                                                                         0.1s
 => => writing image sha256:53a4be960e6c5bb6d1229e9dbbf33c5b2a463f1d427f566ff8bbacc51ca71356                                                                    0.0s
 => => naming to docker.io/library/ng-s-1:0.1.0                                                                                                                 0.0s
```

# load balancing(nginx 공식문서)
https://www.nginx.com/resources/glossary/load-balancing/
