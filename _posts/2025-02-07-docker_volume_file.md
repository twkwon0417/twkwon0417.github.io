---
title: Docker Mount, Dockerfile
date: 2025-02-07 14:23:52 +9000
categories: [Infra, Docker]
tags: [dockerfile, volume, infra, tmpfs]     # TAG names should always be lowercase
---

Docker Mounts
--
![docker_mounts](../assets/Docker/types-of-mounts-volume.png)

- volume: 도커 볼륨은 docker내에서 관리, 여러 컨테이너 간에 안전하게 공유할수 있다.
``` bash
docker volume create my-appvol-1
docker run -v mu-appvol-1:/var/log
```

- bind mount: 시용자가 파일 또느 디렉터리를 생성하면 해당 호스트 파일 시스템의 소유자 권한으로 연결이 되고, 존재하지 않는 경우 자동 생겅 된다. 이때 자동 생성된 디렉터리는 루트 사용자 소유가 된다.
```
docker run -d -it --name bind-test1 \
> --mount type=bind,source="$(pwd)",target=/var/log centos
```
둘이 같다.
```
docker run -d -it --name bind-test2 \
> -v "$(pwd)":/var/log centos
```

- tmpfs mount: 호스트 메모리에서만 지속되는 mount
```
docker run --tmpfs <mount-path>
```

Dockerfile
--
### IaC(infrastructure as a code): 탄력성, 반복성 good
application에 적용되는 새로운 환경을 사용자가 직접 정의해서 아이디어를 실현 할수 있는 것 code 로서 infra 환경 provisioning => dockerfile

### 효율적인 dockerfile이란? - 빌드 시간, 이미지 크기, 재사용성, 보안, 유지보수성 등등
- 경량의 컨테이너 서비스를 제공: 최소한의 설정과 구성
- dockerfile에 담기는 layer 최소화: dockerfile 명령어 수 == docker image layer 수 => layer수 줄이자
- 하나의 application 하나의 컨테이너에: else 결합성 나뻐, 확장성 안좋아
- cache 기능 사용: 이미지를 빌드하면 자동으로 각 명령어 단위로 캐싱된다.
- IaC환경은 디렉터리 단위로: Dockerfile로 이미지 빌드 시 현재 위치로부터 하위 경로의 모든 디렉터리와 파일을 도커 컨텍스트에 저장한뒤 작업이 진행되기 떄문
- 서버리스 환경으로 개발

### Buildkit을 사용하자
- 빌드 병렬처리
- 사용되지 않는 빌드 단계 비활성화
- secret구축 가능
- 빌드 중 빌드 정보에 따라 변경된 파일만 전송
- 자동 빌드 시 캐시의 우선순의를 정한다.

```Dockerfile
# base image 설정
# tag를 넣지 않으면 자동으로 latest로 설정.
FROM ubuntu:latest

# Build시점 인자 전달 받기
# 보안 문제 https://vsupalov.com/better-docker-build-secrets/
ARG ARG_EG

# Dockerfile 작성자에 대한 정보
MAINTAINER "Tony Kwon <twkwon0417@gmail.com>"
# 생성하는 이미지에 대한 설명 e.g. purpose, version, description
LABEL title="IaC, PHP application"

# 컨테이너의 기본 사용자는 root, application doesnt need authority, user를 통해 다른 사용자로 변경해 사용
USER tony

# Docker Image를 생성할때 실행되는 명령어
# Ubuntu의 경우 apt-get update는 필수
# apt cache를 삭제하여 image size 감소시킬수 있다.
# -y, --yes, --assume-yes의 경우 prompt에 assume "yes" as answer to all prompts and run non-interactively.
RUN apt-get update && apt-get -y install apache2 \
    php \
    git \
    curl \
    ssh \
    wget

# 볼룸을 이미지 빌드에 미리 설정, 볼룸의 기본 경로 /var/lib/docker와 자동으로 연결
VOLUME /var/log

# 환경변수 지정
ENV APACHE2_WEB_DIR /var/www/html


# 호스트 환경의 파일, 디렉터리를 이미지 안에 복사하는 경우 작성
COPY index.html /var/www/html

# COPY와 비슷하나 url주소에서 직접 다운로드 해 넣을 수도 있고, 지정한 경로에 압축을 풀어 추가한다.
ADD example.com.tar.gz /workspace/data/

RUN echo '<?php phpinfo(); ?>' > /var/www/html/index.php

# 80번 port 노출
# 컨테이너와 호스트 네트워크 사이의 port를 여는 것, host network에서도 열려면 run시 -p 해주자
EXPOSE 80

# RUN, CMD, ENTRYPOINT의 명령어가 실행되는 디렉터리 설정, 없으면 자동 설정
WORKDIR /var/www/html

# 생성된 이미지를 컨테이너로 실행할 때 실행되는 명령
# container 실행시 항상 수행해야하는 명령어
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D"]

# 생성된 이미지를 컨테이너로 실행할 때 실행되는 명령
# 여러개의 CMD가 있으면 마지막께 쓰이고, 외부에서 ocker이미지를 컨테이너에서 실행할 때 인자를 주면 그것을 CMD로 인식하여 override된다.
# Exec 실행되고 마지막에 실행 된다.
CMD ["FOREGROUND"]

# container실행시 인자가 따로 없으면 FOREGROUND로 실행됨

# 다루지 않은 것!!!!!: ONBUILD, STOPSIGNAL, HEALTHCHECK, SHELL
```

### Reference
- [Docker Mounts](https://docker-docs.uclv.cu/storage/volumes/)
- 도커, 컨테이너 빌드 업

