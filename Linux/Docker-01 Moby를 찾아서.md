---
title: Docker-01 모비를 찾아서
description : kyoulee blog
preview: /post/templates/images/default-og-image.jpg
created: 2026-02-09T11:02:13+09:00
updated: 2026-02-09T11:02:13+09:00
tags:
  - Docker
  - 01
status: public
id : 0
writer : kyoulee
---

docker 에는 Moby 라는 마스코트 🐋 고래가 있다.

귀여운 마스코드 고래에 대하여 알아보고 사용해보자

## Install

docker 는 여러방법으로 설치를 할 수 있다.

필자는 sh 을 위조로 설명하려한다.

```sh
curl -fsSL https://get.docker.com -o get-docker.sh

sh get-docker.sh
```

상위와 같이 docker 에서 제공하는 shell script로 설치도 가능하다 

혹은 docker desktop 을 사용하여 다양한 옵션과 설정을 하여 사용도 가능하다.

## Docker 

> [!Important]
> 필자는 mac os 를 사용하기 때문에 window 와 다를 수 있다는 것을 유의 바란다.


## run

docker 명령어 정리

docker container ls -a 

76 docker container ls -a 
77 docker container rm 04a
docker container ls -a docker container --help
78 docker container run -publish 80:80--detach --name webhost nainx
79 docker container logs webhost
80 docker container Is -a
81 curl 0.0.0.0:80
82 docker container rm f8d
83 docker container stop f8d
84 docker container rm f8d
macbook@Lee-MacBook-Air ~ % 

docker container top

docker container state
docker container inspect


docker network ls
docker network inspect

docker network create 

docker network connect [network] [container]




