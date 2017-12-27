Micro Service Architecture 서버 구축(4/4) - NginX 설치
======================================================

### Nginx

Nginx는 haproxy와 같은 기능을 제공한다.

### 4. Ubuntu 14.04 LTS에 NginX 설치

#### 설치

현재 안정버전은 1.10.2

```sh
$ cd ~/download
$ wget http://nginx.org/download/nginx-1.10.2.tar.gz
$ tar zxvf nginx-1.10.2.tar.gz

$ useradd --shell /usr/sbin/nologin www-data
```

컴파일 옵션을 참조 (http://nginx.org/en/docs/install.html) –with 옵션에 ssl, pcre, zlib 가 등장하므로. 이것들을 미리 설치해주자.

```sh
$ sudo apt-get install openssl libssl-dev
$ sudo apt-get install libpcre3 libpcre3-dev
$ sudo apt-get install zlib1g-dev
```

아래 -path= 옵션들의 경로는 입력하지 않을 경우 nginx의 기본값이기도 하다.

```sh
$ cd ~/download/nginx-1.10.2
$ sudo ./configure --prefix=/usr/local/nginx \
--sbin-path=/usr/local/nginx/sbin/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--pid-path=/usr/local/nginx/logs/nginx.pid \
--error-log-path=/usr/local/nginx/logs/error.log \
--http-log-path=/usr/local/nginx/logs/access.log \
--with-http_ssl_module


$ sudo apt-get install build-essential
$ sudo make
$ sudo make install
```

####nginx 구동 방법

```sh
$ sudo /usr/local/nginx/sbin/nginx
```

별다른 메시지가 없다면 정상 구동이 된것이다. 브라우저에서 서버아이피를 검색하면 “Welcome to nginx!” 가 보이면 성공!!!

#### service 등록

```sh
$ sudo wget https://raw.github.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx;
$ sudo chmod +x /etc/init.d/nginx;
```

자동 실행 등록

```sh
$ sudo update-rc.d -f nginx defaults
```

현재 실행중인 NginX의 상태를 체크

```sh
$ service nginx status
```

서버 정지

```sh
$ sudo service nginx stop
```

서버 시작

```sh
$ sudo service nginx start
```

#### tomcat 연동

```sh
$ cd /usr/local/nginx/conf
$ mkdir msa_conf
$ cd ./msa_conf


$ vi web.conf
upstream user {
    ip_hash;
    server 127.0.0.1:9080;
}

upstream admin {
    ip_hash;
    server 127.0.0.1:9090;
}


server {
    listen       80;

    server_name  localhost;

    #로그위치 설정
    access_log   /usr/local/nginx/logs/user_access.log;

    location / {
        proxy_set_header        Host $http_host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto $scheme;
        proxy_set_header        X-NginX-Proxy true;
        #위에서 설정해준 upstream경로 복사
        proxy_pass http://user;
        proxy_redirect          off;
        charset utf-8;

       #이 밑으로는 옵션인데 필요없으면 안 넣어도 됩니다.
        if ($request_filename ~* ^.*?/([^/]*?)$)
        {
            set $filename $1;
        }

        if ($filename ~* ^.*?\.(eot)|(ttf)|(woff)$){
             add_header Access-Control-Allow-Origin *;
        }
    }

    location / {
        proxy_set_header        Host $http_host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto $scheme;
        proxy_set_header        X-NginX-Proxy true;
        #위에서 설정해준 upstream경로 복사
        proxy_pass http://admin;
        proxy_redirect          off;
        charset utf-8;

       #이 밑으로는 옵션인데 필요없으면 안 넣어도 됩니다.
        if ($request_filename ~* ^.*?/([^/]*?)$)
        {
            set $filename $1;
        }

        if ($filename ~* ^.*?\.(eot)|(ttf)|(woff)$){
             add_header Access-Control-Allow-Origin *;
        }
    }
}

server {




$ vi api.conf
upstream api {
    ip_hash;
    server 127.0.0.1:9022;
}


server {
    listen       8022;

    server_name  localhost;

    #로그위치 설정
    access_log   /usr/local/nginx/logs/api_access.log;

    location / {
        proxy_set_header        Host $http_host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto $scheme;
        proxy_set_header        X-NginX-Proxy true;
        #위에서 설정해준 upstream경로 복사
        proxy_pass http://api;
        proxy_redirect          off;
        charset utf-8;

        #이 밑으로는 옵션인데 필요없으면 안 넣어도 됩니다.
        if ($request_filename ~* ^.*?/([^/]*?)$)
        {
            set $filename $1;
        }

        if ($filename ~* ^.*?\.(eot)|(ttf)|(woff)$){
             add_header Access-Control-Allow-Origin *;
        }
    }
}

```

####nginx.conf 수정

위의 설정은 nginx.conf 의 기본 server 설정과 중복되므로 기존의 기본 server 블럭을 모두 주석처리 혹은 삭제한다.

주의! http 블럭을 지우는 것이 아님.

그 후에 아래의 문구를 추가하여 web_conf 디렉토리의 .conf 확장자의 파일이 모두 포함되도록 설정한다.

```sh
$ cd /usr/local/nginx/conf

$ vi nginx.conf
...
http {
    ...
    include     ./web_conf/*.conf;
    ...
}

# nginx를 리로드한다.
$ sudo service nginx reload

```

---

#####gitLab 설치

-	[Ubuntu에 GitLab 설치 - 설치 가이드](../gitlab/README.md)

#####jeinkins 설치

-	[Ubuntu에 Jenkins 설치 - 설치 가이드](../jenkins/README.md)

#####테스트 서버 설치

-	[Micro Service Architecture 서버 구축(1/4) - java8설치](../web_server/README.md)
-	[Micro Service Architecture 서버 구축(2/4) - tomcat8설치](../web_server/tomcat.md)
-	[Micro Service Architecture 서버 구축(3/4) - redis설치 ](../web_server/redis.md)
-	[Micro Service Architecture 서버 구축(4/4) - nginx설치 ](../web_server/nginx.md)

#####MariaDB 설치

-	[Ubuntu에 MariaDB 설치 - 설치 가이드](../mariadb/README.md)
