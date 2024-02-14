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

# Dockerfile 생성 및 build하기
https://blog.naver.com/mini_crane_/223353541818

# load balancing
