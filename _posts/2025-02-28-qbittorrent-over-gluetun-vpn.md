---
title: Docker에서 qBittorrent를 Wireguard VPN위에서 실행하기
date: 2025-02-28
categories: [홈서버]
tags: [docker, qbittorrent, wireguard, gluetun, nginx]
---

## Introduction
Docker를 사용하여 qBittorrent를 Wireguard VPN을 통한 네트워크 상에서 실행하고자 함. Wireguard 클라이언트로 Gluetun을 사용함.
  
## Docker Compose 구성
* `docker-compose.yml`{: .filepath} 에서 다음과 같이 서비스 구성

``` yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    volumes:
      - ./gluetun:/gluetun
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - TZ=Asia/Seoul
      - VPN_SERVICE_PROVIDER=custom
      - VPN_TYPE=wireguard
      - FIREWALL_VPN_INPUT_PORTS=6881
      - WIREGUARD_ENDPOINT_PORT=
      - WIREGUARD_ADDRESSES=
      - WIREGUARD_ENDPOINT_IP=
      - WIREGUARD_PUBLIC_KEY=
      - WIREGUARD_PRIVATE_KEY=
      - WIREGUARD_PRESHARED_KEY=
    ports:
      - 10000:10000
      - 6881:6881/tcp
      - 6881:6881/udp
  qbittorrent:
    image:  qbittorrentofficial/qbittorrent-nox:latest
    container_name: qbittorrent
    restart: unless-stopped
    network_mode: service:gluetun
    depends_on:
      - gluetun
    environment:
      - TZ=Asia/Seoul
      - QBT_WEBUI_PORT=10000
      - QBT_LEGAL_NOTICE=
    volumes:
      - ./qbittorrent:/config
      - ./downloads:/downloads #optional
```
* Gluetun에서 `FIREWALL_VPN_INPUT_PORTS`를 설정하지 않았더니 qBittorrent에서 트래커 정보를 불러오지 못했음. 위 경우에서는 기본 토렌트 포트인 6881을 사용함. 또한 해당 포트 tcp, udp 바인딩.
* qBittorrent의 웹 UI 포트를 gluetun 서비스에서 바인딩해줘야 웹에서 접근 가능함. 위 경우에서는 10000번 포트로 설정했음.
* Wireguard 서버 측 방화벽 또는 NAT에서 6881포트에 대한 포트포워딩은 필요 없음.

## nginx를 통한 다운로드된 파일 접근
* qBittorrent 웹 UI 에서는 다운로드된 파일에 접근할 수 없기 때문에 해당 파일들을 nginx를 통해 다른 클라이언트에서 직접 다운로드 할 수 있도록 구성할 수 있음.
* 위에서 구성한 `docker-compose.yml`{: .filepath} 에 다음과 같은 nginx 서비스를 추가

``` yaml
nginx:
  image: nginx:alpine
  restart: unless-stopped
  ports:
    - 10001:80
  environment:
    - TZ=Asia/Seoul
  volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf
    - ./downloads:/usr/share/nginx/html:ro
```
* nginx 기본 포트 80번을 사용할 포트 10001번에 바인딩
* nginx에서 파일을 서빙하기 위해 다음과 같이 설정파일 `nginx.conf`{: .filepath} 구성

``` conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    access_log off;
    sendfile        on;
    keepalive_timeout  65;
    gzip  on;
    include /etc/nginx/conf.d/*.conf;
    autoindex on;
}
```
* `nginx:alpine` 이미지와 간단한 설정 파일로 단순한 파일 조회만 가능하도록 설정함. 

## 최종 docker-compose.yml
``` yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    volumes:
      - ./gluetun:/gluetun
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - TZ=Asia/Seoul
      - VPN_SERVICE_PROVIDER=custom
      - VPN_TYPE=wireguard
      - FIREWALL_VPN_INPUT_PORTS=6881
      - WIREGUARD_ENDPOINT_PORT=
      - WIREGUARD_ADDRESSES=
      - WIREGUARD_ENDPOINT_IP=
      - WIREGUARD_PUBLIC_KEY=
      - WIREGUARD_PRIVATE_KEY=
      - WIREGUARD_PRESHARED_KEY=
    ports:
      - 10000:10000
      - 6881:6881/tcp
      - 6881:6881/udp
  qbittorrent:
    image:  qbittorrentofficial/qbittorrent-nox:latest
    container_name: qbittorrent
    restart: unless-stopped
    network_mode: service:gluetun
    depends_on:
      - gluetun
    environment:
      - TZ=Asia/Seoul
      - QBT_WEBUI_PORT=10000
      - QBT_LEGAL_NOTICE=
    volumes:
      - ./qbittorrent:/config
      - ./downloads:/downloads #optional
  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - 10001:80
    environment:
      - TZ=Asia/Seoul
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./downloads:/usr/share/nginx/html:ro
```
## References
1. [[nginx] 간단 파일리스팅 서버](https://blog.leocat.kr/notes/2018/01/08/nginx-simple-file-listing-server)
