---
title: 自用Docker项目部署分享
date: 2024-09-20T17:31:31.989Z
---

自用Docker项目部署分享​



常用参数

Docker容器的重启策略如下：





no，默认策略，在容器退出时不重启容器



on-failure，在容器非正常退出时（退出状态非0），才会重启容器



on-failure:3，在容器非正常退出时重启容器，最多重启3次



always，在容器退出时总是重启容器



unless-stopped，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器



 # 时区
 environment:
     - TZ=Asia/Shanghai

cloudflare穿透工具flaresolverr



 version: '3.3'
 services:
     flaresolverr:
         # DockerHub mirror flaresolverr/flaresolverr:latest
         image: ghcr.io/flaresolverr/flaresolverr:latest
         container_name: flaresolverr
         environment:
           - LOG_LEVEL=info
           - TZ=Asia/Shanghai
         ports:
           - 8191:8191
         restart: unless-stopped

思维导图mind-map



 version: '3.3'
 services:
   mindmap:
     container_name: mindmap
     image: shuiche/mind-map
     ports:
       - '6679:8080'
     environment:
       TZ: Asia/Shanghai
     restart: always

小雅阿里资源



 version: '3.3'
 services:
   xiaoya:
     container_name: xiaoya
     image: xiaoyaliu/alist
     ports:
       - '3579:80'
     volumes:
       - /home/zsg/docker/xiaoya:/data
       - /home/zsg/docker/xiaoya/alist:/opt/alist/data
     environment:
       TZ: Asia/Shanghai
     restart: always

Snapdrop网页文件传输



 version: "2.1"
 services:
   snapdrop:
     image: lscr.io/linuxserver/snapdrop:latest
     container_name: snapdrop
     environment:
       - TZ=Asia/Shanghai
     volumes:
       - /home/zsg/docker/snapdrop:/config
     ports:
       - 7635:80
     restart: unless-stopped

web-check网站测试



 version: '3.3'
 ​
 services:
     webcheck:
         container_name: webcheck
         restart: unless-stopped
         image: lissy93/web-check
         ports:
             - '3084:3000'
         environment:  
             - TZ=Asia/Shanghai

迅雷



 services:
   xunlei:
     image: cnk3x/xunlei:latest
     privileged: true
     container_name: xunlei
     hostname: nas
     network_mode: host
     environment:
       - XL_WEB_PORT=4321
       - XL_BA_USER=user
       - XL_BA_PASSWORD=password
     volumes:
       - /home/zsg/docker/xunlei/data:/xunlei/data
       - /home/zsg/docker/xunlei/downloads:/xunlei/downloads
     restart: unless-stopped

Fail2ban



 version: '3.3'
 services:
     fail2ban:
         container_name: fail2ban
         restart: no
         network_mode: host
         volumes:
             - /home/zsg/docker/fail2ban/data:/data
             - /var/log:/var/log:ro
         image: crazymax/fail2ban:latest
         environment:
             - TZ=Asia/Shanghai
             - F2B_LOG_TARGET=STDOUT
             - F2B_LOG_LEVEL=INFO
             - F2B_DB_PURGE_AGE=1d
         cap_add:
             - NET_ADMIN 
             - NET_RAW

群晖DSM



 docker network create -d macvlan \
     --subnet=10.0.0.0/24 \
     --gateway=10.0.0.1 \
     --ip-range=10.0.0.160/28 \
     -o parent=eth0 vdsm
 # 先建立macvlan网络，eth0要改成你的网卡地址
 # 以上按照你的局域网ip改



 version: '3.3'
 services:
     synology-dsm:
         container_name: dsm
         restart: always
         environment:
             TZ: Asia/Shanghai
             CPU_CORES: "2"
             RAM_SIZE: "1024M"
             DISK_SIZE: "16G"
             ALLOCATE: "N"
             DHCP: "Y"
         devices:
             - /dev/kvm
             - /dev/vhost-net   
         device_cgroup_rules:
             - 'c *:* rwm' 
         cap_add:
             - NET_ADMIN            
         ports:
             - '5000:5000'
         volumes:
             - /home/zsg/docker/dsm:/storage
         stop_grace_period: 1m
         image: kroese/virtual-dsm:latest
         networks:
             vdsm:             
                 ipv4_address: 10.0.0.160  #依据上面设置的网段选择一个ip
 networks:
     vdsm:
         external: true
 #详细 https://hub.docker.com/r/kroese/virtual-dsm

单点登录Authelia



 version: "3.3"
 services:
   authelia:
     container_name: authelia
     image: authelia/authelia:latest
     restart: unless-stopped
     ports:
       - 19091:9091
     environment:
       - TZ=Asia/Shanghai
     volumes:
       - /home/zsg/docker/authelia/config:/config

内网穿透FRP

FRPS服务端



 version: '3.3'
 services:
     frps:
         restart: always
         network_mode: host
         volumes:
             - '/home/zsg/docker/frps/frps.ini:/etc/frp/frps.ini'
             - '/home/zsg/docker/frps/frps.log:/frps.log'
         container_name: frps
         environment:
             - TZ=Asia/Shanghai
         image: snowdreamtech/frps



 # 服务端配置文件frps.ini
 # [common] is integral section
 [common]
 # A literal address or host name for IPv6 must be enclosed
 # in square brackets, as in "[::1]:80", "[ipv6-host]:http" or "[ipv6-host%zone]:80"
 bind_addr = 0.0.0.0
 bind_port = 23456
 # udp port used for kcp protocol, it can be same with 'bind_port'
 # if not set, kcp is disabled in frps
 kcp_bind_port = 23456
 # if you want to configure or reload frps by dashboard, dashboard_port must be set
 dashboard_port = 9963
 # dashboard assets directory(only for debug mode)
 dashboard_user = user
 dashboard_pwd = password
 # console or real logFile path like ./frps.log
 log_file = console
 #./frps.log
 # debug, info, warn, error
 log_level = debug
 log_max_days = 3
 # auth token
 token = 复杂点
 # only allow frpc to bind ports you list, if you set nothing, there won't be any limit
 #allow_ports = 1-65535
 # pool_count in each proxy will change to max_pool_count if they exceed the maximum value
 max_pool_count = 50
 # if tcp stream multiplexing is used, default is true
 tcp_mux = true
 # vhost_http_port = 61233

FRPC客户端



 docker run --restart=always --network host -d -v /home/zsg/docker/frpc/frpc.ini:/etc/frp/frpc.ini --name frpc snowdreamtech/frpc



 version: '3.3'
 services:
     frpc:
         restart: always
         network_mode: host
         volumes:
             - '/home/zsg/docker/frpc/frpc.ini:/etc/frp/frpc.ini'
             - '/home/zsg/docker/frpc/frpc.log:/frpc.log'
         container_name: frpc
         environment:
             - TZ=Asia/Shanghai
         image: snowdreamtech/frpc



 # 客户端配置文件frpc.ini
 [common]
 server_addr = 88.88.88.88
 server_port = 23456
 token = 复杂点
 log_file = ./frpc.log

 [test]
 type = tcp
 local_ip = 10.0.0.5
 local_port = 5244
 remote_port = 5211

RustDesk远程桌面



 version: '3'
 services:
   hbbs:
     container_name: rustdesk-hbbs
     ports:
       - 21115:21115
       - 21116:21116
       - 21116:21116/udp
       - 21118:21118
     image: rustdesk/rustdesk-server
     command: hbbs
     volumes:
       - '/home/zsg/docker/rustdesk/hbbs:/root'
     restart: always
 ​
   hbbr:
     container_name: rustdesk-hbbr
     ports:
       - 21117:21117
       - 21119:21119
     image: rustdesk/rustdesk-server
     command: hbbr
     volumes:
       - '/home/zsg/docker/rustdesk/hbbr:/root'
     restart: always

Stirling PDF工具箱



 version: '3.3'
 services:
   stirling-pdf:
     container_name: PDF
     image: frooodle/s-pdf:latest
     restart: always
     ports:
       - '6180:8080'
     volumes:
       - /home/zsg/docker/PDF/trainingData:/usr/share/tesseract-ocr/4.00/tessdata
     environment:
       TZ: Asia/Shanghai
       APP_LOCALE: zh_CN
       APP_HOME_NAME: PDF
       APP_HOME_DESCRIPTION: "PDF百宝箱"
       APP_NAVBAR_NAME: PDF
       APP_ROOT_PATH: /
 ​
 教程来源于https://post.smzdm.com/p/a5o68zdl/
 OCR附件如下，放到trainingData内

Alist



 version: '3.3'
 services:
     alist:
         restart: always
         volumes:
             - './docker/alist:/opt/alist/data'
             - '/home:/home/alist'
         ports:
             - '5244:5244'
         environment:
             - TZ=Asia/Shanghai
         container_name: alist
         image: 'xhofe/alist:latest'
 ​
 #公开分享（无密码、无需登录访问）= 用户/guest的基本路径，
 # 把【无需密码访问】取消勾选才可以对它的下级路径加密
 #用密码的分享记得勾选【应用到子文件夹】

思维导图wiseapp



 mkdir ./wiseapp
 docker run --name wiseapp -d --mount type=bind,source=./wiseapp,target=/var/lib/wise-db wisemapping/wisemapping:latest
 docker run --name wiseapp -d -v ./wiseapp:/var/lib/wise-db wisemapping/wisemapping:latest
 docker cp wiseapp:/var/lib/wisemapping/db ./wiseapp
 docker stop wiseapp;docker rm wiseapp
 docker run --mount type=bind,source=./wiseapp/db,target=/var/lib/wisemapping/db -it --rm -p 6080:8080 wisemapping/wisemapping:latest
 docker run --name wiseapp -v ./wiseapp/db:/var/lib/wisemapping/db -d -p 6080:8080 wisemapping/wisemapping:latest
 ​
 默认账号：test@wisemapping.org密码test
 默认账号：admin@wisemapping.org密码test



 version: '3.3'
 services:
     wisemapping:
         container_name: wiseapp
         restart: always
         environment:
             - TZ=Asia/Shanghai
         volumes:
             - './wiseapp/db:/var/lib/wisemapping/db'
         ports:
             - '6080:8080'
         image: 'wisemapping/wisemapping:latest'

静态站点nginx-php



 version: '3.3'
 services:
     nginx: 
         container_name: nginx
         ports:
             - '3387:80'
         environment:
             - TZ=Asia/Shanghai
         restart: always
         volumes:
            - './zdj:/usr/share/nginx/html'
         image: gindex/nginx-php

网站访问统计Umami



 version: '3'
 services:
   umami:
     container_name: umami
     image: ghcr.io/umami-software/umami:postgresql-latest
     ports:
       - "3208:3000"
     environment:
       DATABASE_URL: postgresql://umami:12345678@db:5432/umami
       DATABASE_TYPE: postgresql
       APP_SECRET: a7ija2B7XHkd
     depends_on:
       - db
     restart: always
   db:
     container_name: umami-db
     image: postgres:15-alpine
     environment:
       POSTGRES_DB: umami
       POSTGRES_USER: umami
       POSTGRES_PASSWORD: 12345678
     volumes:
       - ./umami/sql/schema.postgresql.sql:/docker-entrypoint-initdb.d/schema.postgresql.sql:ro
       - ./umami/data:/var/lib/postgresql/data
     restart: always
     
     # 默认用户名admin和密码umami
     # https://github.com/umami-software/umami

WebSSH



 version: '3.3'
 services:
     webssh:
         container_name: webssh
         restart: no
         ports:
             - '5032:5032'
         environment:
             - PUID=0   
             - PGID=0  
             - TZ=Asia/Shanghai 
         image: jrohy/webssh

备忘录memos



 version: '3.3'
 services:
     memos:
         container_name: memos
         restart: always
         environment:  
             - TZ=Asia/Shangha        
         ports:
             - '5230:5230'
         volumes:
             - './memos/:/var/opt/memos'
         image: 'neosmemo/memos:latest'

Chatgpt-web



 version: '3.3'
 ​
 services:
     chatgpt-web:
         container_name: chatgpt
         restart: always
         image: yidadaa/chatgpt-next-web
         ports:
             - '3355:3000'
         environment:  
             - TZ=Asia/Shangha
             - OPENAI_API_KEY=sk-xxxxxxx
             - CODE=8888888
             - BASE_URL=https://api.openai.com

Traccar位置追踪

导出traccar.xml默认文件



 docker run \
 --rm \
 --entrypoint cat \
 traccar/traccar:latest \
 /opt/traccar/conf/traccar.xml > ./traccar/traccar.xml



 version: '3.3'
 services:
     traccar:
         container_name: traccar
         restart: always
         environment:
             - TZ=Asia/Shanghai
         ports:
             - '3697:8082'
             - '5000-5150:5000-5150'
             - '5000-5150:5000-5150/udp'
         volumes:
             - './traccar/data:/opt/traccar/data:rw'
             - './traccar/logs:/opt/traccar/logs:rw'
             - './traccar/traccar.xml:/opt/traccar/conf/traccar.xml:ro'
         image: 'traccar/traccar:latest'

反代服务器Traefik



 version: '3.3'
 services:
     traefik:
         container_name: traefik
         network_mode: host
         restart: always
         environment:
             - TZ=Asia/Shanghai
             - 'DNSPOD_API_KEY=123,66666
             - DNSPOD_HTTP_TIMEOUT=30
         volumes:
             - './traefik:/etc/traefik:rw'
             - '/var/run/docker.sock:/var/run/docker.sock:rw'
         image: 'traefik:latest'

Reader阅读



 version: '3.3'
 services:
     reader:
         restart: always
         container_name: reader
         environment:
             - SPRING_PROFILES_ACTIVE=prod
             - READER_APP_SECURE=true
             - READER_APP_SECUREKEY=282828   #管理密码
             - READER_APP_INVITECODE=33655 #邀请码
             - TZ=Asia/Shanghai
         volumes:
             - './reader/log:/log'
             - './reader/storage:/storage'
         ports:
             - '8769:8080'
         image: hectorqin/reader

绘图Draw



 version: '3.3'
 services:
     drawio:
         container_name: draw
         ports:
             - '8322:8080'
         environment:
             - TZ=Asia/Shanghai
         restart: always
         image: jgraph/drawio

RSS订阅FreshRSS



 version: '3.3'
 services:
     freshrss:
         container_name: freshrss
         environment:
             - CRON_MIN: '*/45'
             - TZ=Asia/Shanghai
         ports:
             - '5655:80'
         volumes:
             - './freshrss:/var/www/html/data'
         image: freshrss/freshrss

ESP32HOME



 version: '3.3'
 services:
     esphome:
         container_name: esphome
         ports:
             - '6052:6052'
         environment:
             - 'http_proxy=http://10.0.0.5:7890'
             - 'https_proxy=http://10.0.0.5:7890'
             - 'no_proxy=localhost,10.0.0.5,10.0.0.1,192.168.1.1,127.0.0.1,.local,localhost'
         image: esphome/esphome

手绘白板



 version: '3.3'
 services:
     excalidraw:
         container_name: excalidraw
         ports:
             - '8099:80' 
         environment:
             - TZ=Asia/Shanghai
         volumes:
             - './excalidraw:/app/web'
         restart: always
         image: 'excalidraw/excalidraw'

Nginx+PHP



 docker run -d --name nginx -p 6161:80 -v ./nginx:/usr/share/nginx/html:ro nginx



 version: '3.3'
 services:
     php80:
         container_name: php8.0
         restart: always
         image: php:8.0-fpm
         volumes:
             - ./nginx/site:/var/www/html
         ports:
             - "19080:9000"
         environment:
             - PHP_IDE_CONFIG=serverName=php
         networks:
             - web
     php73:
         container_name: php7.3
         restart: always
         image: php:7.3-fpm
         volumes:
             - ./nginx/site:/var/www/html
         ports:
             - "19073:9000"
         environment:
             - PHP_IDE_CONFIG=serverName=php
         networks:
             - web       
     php56:
         container_name: php5.6
         restart: always
         image: php:5.6-fpm
         volumes:
             - ./nginx/site:/var/www/html
         ports:
             - "19056:9000"
         environment:
             - PHP_IDE_CONFIG=serverName=php
         networks:
             - web               
     nginx:
         container_name: nginx
         restart: always
         environment:
             - TZ="Asia/Shanghai"
         ports:
             - '6161-6169:6161-6169'
         volumes:
             - './nginx/site:/usr/share/nginx/html:ro'
             - './nginx/conf.d:/etc/nginx/conf.d:ro'
             - './nginx/nginx.conf:/etc/nginx/nginx.conf:ro'
         image: nginx
         depends_on:
             - php80
             - php73
             - php56
         networks:
             - web
 networks:
      web:

phpmyadmin



 # 任意服务器
 docker run --name phpmyadmin -d -e PMA_ARBITRARY=1 -p 18080:80 phpmyadmin
 # 指定服务器及端口
 docker run --name phpmyadmin -d -e PMA_HOST=127.0.0.1 -e PMA_PORT=3306 -p 18080:80 phpmyadmin

MySQL



 version: '3.3'
 services:
     mysql:
         container_name: mysql
         restart: always
         ports:
             - '3306:3306'
         volumes:
             - './mysql/data:/var/lib/mysql'
         environment:
             - MYSQL_ROOT_PASSWORD=AxUr6aTg8u5pv8
             - TZ=Asia/Shanghai
         image: mysql

Linux命令查询



 version: "3.3"
 services: 
   web:
     image: stilleshan/linux-command
     container_name: linux-command
     ports:
       - 7326:80
     restart: always

docker-proxy



 version: '3.3'
 services:
   dockerproxy:
     image: tecnativa/docker-socket-proxy
     container_name: dockerproxy
     privileged: true
     volumes:
       - /var/run/docker.sock:/var/run/docker.sock
     ports:
       - 2375:2375
     environment:
       - BUILD=1
       - COMMIT=1
       - CONFIGS=1
       - CONTAINERS=1
       - DISTRIBUTION=1
       - EXEC=1
       - IMAGES=1
       - INFO=1
       - NETWORKS=1
       - NODES=1
       - PLUGINS=1
       - SERVICES=1
       - SESSSION=1
       - SWARM=1
       - POST=1


测试连通命令



 docker -H tcp://10.0.0.9:2375 ps
 ​
 docker -H tcp://10.0.0.9:2375 ps

文件快递柜



 version: '3.3'
 services:
     filecodebox:
         container_name: filecodebox
         restart: always
         ports:
             - '12345:12345'
         volumes:
             - './FileCodeBox/:/app/data'
         image: 'lanol/filecodebox:latest'

MQTT



 version: '3.3'
 services:
     eclipse-mosquitto:
         container_name: MQTT
         restart: always
         ports:
             - '1883:1883'
             - '9001:9001'
         volumes:
             - './mqtt/conf:/mosquitto/config'
             - './mqtt/data:/mosquitto/data'
             - './mqtt/log:/mosquitto/log'
         image: eclipse-mosquitto

测速speedtest



 version: '3.3'
 services:
     speedtest:
         container_name: speedtest
         restart: always
         ports:
             - '12345:80'
         image: adolfintel/speedtest

订阅转换



 version: '3.3'
 services:
     subweb: #前端
         container_name: subweb
         restart: always
         ports:
             - '58080:80'
         image: 'careywong/subweb:latest'
     subconverter: #后端
         container_name: sub
         restart: always
         ports:
             - '25500:25500'
         image: 'tindy2013/subconverter:latest'

Docker命令转堆栈



 version: "3.9"
 services:
   composerize:
     image: alcapone1933/composerize
     container_name: composerize
     restart: always
     ports:
       - 9080:80
     environment:
       - TZ="Asia/Shanghai"
     volumes:
       - './composerize:/var/www/

Docker更新通知



 version: '3.3'
 services:
     diun:
         container_name: diun
         restart: always
         environment:
             - TZ=Asia/Shanghai
             - LOG_LEVEL=info
             - LOG_JSON=false
             - DIUN_PROVIDERS_DOCKER=true
             - DIUN_PROVIDERS_FILE_FILENAME=/custom-images.yml
         volumes:
             - './diun:/data'
             - './diun/custom-images.yml:/custom-images.yml:ro'
             - './diun/diun.yml:/diun.yml:ro'
             - '/var/run/docker.sock:/var/run/docker.sock'
         image: 'crazymax/diun:latest'

看板homepage



version: '3.3'
services:
    diun:
        container_name: diun
        restart: always
        environment:
            - TZ=Asia/Shanghai
            - LOG_LEVEL=info
            - LOG_JSON=false
            - DIUN_PROVIDERS_DOCKER=true
            - DIUN_PROVIDERS_FILE_FILENAME=/custom-images.yml
        volumes:
            - './diun:/data'
            - './diun/custom-images.yml:/custom-images.yml:ro'
            - './diun/diun.yml:/diun.yml:ro'
            - '/var/run/docker.sock:/var/run/docker.sock'
        image: 'crazymax/diun:latest'

trilium笔记



version: '3.3'
services:
    trilium-cn:
        container_name: trilium
        ports:
            - '2992:8080'
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - './trilium:/home/node/trilium-data'
        image: nriver/trilium-cn

FileBrowser文件管理



version: '3.3'
services:
    filebrowser:
        container_name: filebrowser
        ports:
            - '9696:80'
        environment:
            - TZ=Asia/Shanghai
            - PUID=0
            - PGID=0
        volumes:
            - '.:/Disk'
            - './filebrowser/database.db:/database.db'
            - './filebrowser/filebrowser.json:/.filebrowser.json'
            - './filebrowser/cache:/cache'
        image: filebrowser/filebrowser

filebrowser.json

VSCode



version: '3.3'
services:
    code-server:
        container_name: vscode
        environment:
            - TZ=Asia/Shanghai
            - PUID=0
            - PGID=0
            - PASSWORD=KcsAPNb2F4rY
            - DEFAULT_WORKSPACE=/000
        ports:
            - '5797:8443'
        volumes:
            - './vscode:/config'
            - '.:/000'
        restart: always
        image: linuxserver/code-server

百度脑图



version: '3.3'
services:
    kityminder:
        container_name: mind
        restart: always
        ports:
            - '2333:80'
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - './mind:/data'
        image: xinyewdz/kityminder

RSSPush



version: '3.3'
services:
    rsspush:
        container_name: rsspush
        ports:
            - '6863:8000'
        volumes:
            - './rsspush:/rsspush/api/data'
        environment:
            - TZ=Asia/Shangha
            - 'RSS_BASE=http://10.0.0.9:8556'
            - ADMIN_KEY=123456
        image: easychen/rsspush

‍

OCR文字识别



version: '3.3'
services:
    trwebocr:
        ports:
            - '18089:8089'
        restart: always
        container_name: trwebocr
        image: mmmz/trwebocr

相册（MT Photo）



version: '3.3'
services:
    mt-photos:
        container_name: mt-photos
        volumes:
            - './mt_photos/config:/config'
            - './mt_photos/upload:/upload'
        ports:
            - '8063:8063'
        environment:
            - TZ=Asia/Shanghai
        restart: unless-stopped
        image: mtphotos/mt-photos/photosync
volumes:
  photo:
    name: photo
    driver: local
    driver_opts:
      type: 'cifs'
      device: '//10.0.0.5/photosync'
      o: 'addr=10.0.0.5,username=000,password=000000000,vers=3.0'

ADGuard Home



version: '3.3'
services:
    run:
        container_name: adguard
        hostname: adguard
        restart: always
        environment:
            - TZ=Asia/Shanghai
            - PGID=0
            - PUID=0
        volumes:
            - './adguard/workdir:/opt/adguardhome/work'
            - './adguard/confdir:/opt/adguardhome/conf'
        ports:
            - '53:53/tcp'
            - '53:53/udp'
            - '3000:3000/tcp'
        image: run

SmartDNS



version: '3.3'
services:
    smartdns:
        network_mode: host
        hostname: smartdns
        container_name: smartdns
        restart: always
        environment:
            - TZ=Asia/Shanghai
        ports:
            - '53:53/udp'
            - '53:53/tcp'
        volumes:
            - './smartdns:/smartdns
        image: ghostry/smartdns

xTeVe



version: '3.3'
services:
    xteve:
        container_name: xteve
        restart: always
        hostname: xteve
        ports:
            - '34400:34400'
        logging:
            options: 'max-size=10m,max-file=3'
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - './xteve/xteve:/root/.xteve:rw'
            - './xteve/config:/config:rw'
            - './xteve/tmp:/tmp/xteve:rw'
            - './xteve/tvheadend/data:/TVH'
        image: alturismo/xteve

思源笔记（siyuan）



version: '3.3'
services:
    siyuan:
        container_name: siyuan
        restart: always
        volumes:
            - './siyuan:/siyuan'
        ports:
            - '6806:6806'
        image: b3log/siyuan

网页监控（uptime-kuma）



version: '3.3'
services:
    uptime-kuma:
        restart: always
        environment:
            - TZ=Asia/Shanghai
        ports:
            - '3001:3001'
        volumes:
            - './uptime-kuma/data:/app/data'
        container_name: uptime-kuma
        image: 'louislam/uptime-kuma:latest'

影音机器人（movierobot）



version: '3.3'
services:
    movie-robot:
        restart: always
        container_name: movie-robot
        ports:
            - '1329:1329'
        volumes:
            - './movie-robot:/data'
            - '/path/to/media:/media'
        environment:
            - LICENSE_KEY=xxxxxxxxxx
        image: 'yipengfei/movie-robot'

overseerr



version: '3.3'
services:
    overseerr:
        restart: always
        container_name: overseerr
        environment:
            - LOG_LEVEL=debug
            - TZ=Asia/Shanghai
        ports:
            - '5055:5055'
        volumes:
            - './overseerr/config:/config'
        image: miniers/overseerr

docker可视化管理（portainer）



version: '3.3'
services:
    portainer-ee:
        restart: always
        container_name: portainer
        ports:
            - '9002:9000'
        volumes:
            - '/var/run/docker.sock:/var/run/docker.sock'
            - './portainer/data:/data'
        image: portainer/portainer-ce

RSS生成（rsshub）



version: '3.3'
services:
    rsshub:
        restart: always
        container_name: rsshub
        ports:
            - '8556:1200'
        image: diygod/rsshub

flexget



version: '3.7'
services:
  flexget:
    image: madwind/flexget
    container_name: flexget
    environment:
      #密码需保证复杂度
      FG_WEBUI_PASSWD: 65956232
      #日志级别
      FG_LOG_LEVEL: INFO
      TZ: Asia/Shanghai
      PUID: 1000
      PGID: 1000
    volumes:
      - ./flexget/config:/config
      - ./flexget/downloads:/downloads
    ports:
      - "8385:3539"
    restart: always

bark



version: '3.3'
services:
    bark-server:
        restart: always
        environment:
            - TZ=Asia/Shanghai
        container_name: bark
        ports:
            - '8383:8080'
        volumes:
            - './bark/data:/data'
        image: finab/bark-server

密码管理（vaultwarden/Bitwarden）



version: '3.3'
services:
    server:
        restart: always
        container_name: vaultwarden
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - './Bitwarden/data/:/data/'
        ports:
            - '12256:80'
        image: vaultwarden/server

docker容器自动更新（watchtower）



Bark通知部署（一定要https）

version: '3.3'
services:
    watchtower:
        container_name: watchtower
        restart: always
        environment:
            - TZ=Asia/Shanghai
            - WATCHTOWER_NOTIFICATION_URL=bark://:8888888@push.abc.com:8080/
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock

        image: containrrr/watchtower
        command: --cleanup --interval 1200

Bark通知部署（一定要https）

网址收藏夹（shaarli）



version: '3.3'
services:
    shaarli:
        container_name: Shaarli
        restart: always
        ports:
            - '8561:80'
        volumes:
            - './shaarli:/var/www/shaarli/data'
        image: shaarli/shaarli

智能家居（HASS）



version: '3.3'
services:
    home-assistant:
        restart: always
        container_name: hass
        privileged: true
        network_mode: host
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - './hass:/config'
            - '/run/dbus:/run/dbus'
            - '/var/run/dbus:/var/run/dbus'
            - '/dev:/dev'
            - '/var/run/docker.sock:/var/run/docker.sock'
            - '/dev/bus/usb:/dev/bus/usb'
        image: 'homeassistant/home-assistant:latest'

为知笔记（wiz）



version: '3.3'
services:
    wizserver:
        container_name: wiz
        volumes:
            - './wiz/data:/wiz/storage'
            - '/etc/localtime:/etc/localtime'
        ports:
            - '8357:80'
            - '9269:9269/udp'
        image: wiznote/wizserver

默认管理员账号：admin@wiz.cn，密码：123456

堡垒机（next-terminal）



version: '3.3'
services:
  guacd:
    container_name: guacd
    image: dushixiang/guacd:latest
    volumes:
      - ./next-terminal/data:/usr/local/next-terminal/data
    restart:
          always
  next-terminal:
    container_name:  next-terminal  
    image: dushixiang/next-terminal:latest
    environment:
      DB: sqlite
      GUACD_HOSTNAME: guacd
      GUACD_PORT: 4822
    ports:
      - "8068:8088"
    volumes:
      - /etc/localtime:/etc/localtime
      - ./next-terminal/data:/usr/local/next-terminal/data
    restart:
      always

相册（photoprism）



version: '3.3'
services:
    photoprism:
        container_name: photoprism
        ports:
            - '8568:2342'
        environment:
            - PHOTOPRISM_UPLOAD_NSFW=false
            - PHOTOPRISM_ADMIN_PASSWORD=6666666
        volumes:
            - './photoprism:/photoprism/storage'
            - './Pictures/Pictures:/photoprism/originals'
            - './Pictures/Example:/photoprism/originals/Example'
        image: photoprism/photoprism

node-red



version: '3.3'
services:
    node-red:
        container_name: nodered
        restart: always
        ports:
            - '1880:1880'
        environment:
            - TZ=Asia/Shanghai
        volumes:
            - './nodered/data:/data'
        image: nodered/node-red

PLEX监视器（Tautulli）



version: '3.3'
services:
    argparse:
        container_name: tautulli
        restart: always
        volumes:
            - './tautulli:/config'
            - './tautulli/data:/data'
        environment:
            - TZ=Asia/Shanghai
        ports:
            - '28181:8181'
        image: argparse

个人导航（onenav）



version: '3.3'
services:
    onenav:
        container_name: onenav
        ports:
            - '8252:80'
        environment:
            - USER=zsg
            - PASSWORD=6666666
        volumes:
            - './onenav:/data/wwwroot/default/data'
        image: helloz/onenav

网页剪切（wallabag）



version: '3.3'
services:
    wallabag:
        container_name: wallabag
        restart: always
        volumes:
            - './wallabag/data:/var/www/wallabag/data'
            - './wallabag/images:/var/www/wallabag/web/assets/images'
        ports:
            - '5631:80'
        environment:
            - 'SYMFONY__ENV__DOMAIN_NAME=https://wb.abc.com:8080'
        image: wallabag/wallabag

TTRSS



version: "3"
services:
  service.rss:
    image: wangqiru/ttrss:latest
    container_name: ttrss
    ports:
      - 18181:80
    environment:
      - SELF_URL_PATH=https://rss.abc.com:8080 
      - DB_PASS=hr5y15gqeX
      - ALLOW_PORTS=8080,8556
      - ENABLE_PLUGINS=auth_internal,fever,remove_iframe_sandbox
      - PUID=1000
      - PGID=1000
    volumes:
      - ./ttrss/feed-icons/:/var/www/feed-icons/
    networks:
      - public_access
      - service_only
      - database_only
    stdin_open: true
    tty: true
    restart: always

  service.mercury: 
    image: wangqiru/mercury-parser-api:latest
    container_name: mercury
    networks:
      - public_access
      - service_only
    restart: always

  service.opencc:
    image: wangqiru/opencc-api-server:latest
    container_name: opencc
    environment:
      - NODE_ENV=production
    networks:
      - service_only
    restart: always

  database.postgres:
    image: postgres:13-alpine
    container_name: postgres
    environment:
      - POSTGRES_PASSWORD=hr5y15gqeX
    volumes:
      - ./ttrss/data/:/var/lib/postgresql/data
    networks:
      - database_only
    restart: always

networks:
  public_access: 
  service_only: 
    internal: true
  database_only:
    internal: true

