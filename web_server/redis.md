Micro Service Architecture 서버 구축(3/4) - Redis 설치
======================================================

### Redis

Micro Service Architecture에서는 각각의 서비스별로 단독의 주소를 가지고 동작을 하기때문에 사용에있어 일관성을 보여야하는 데이터를 공유해야만 한다. 예로 session정보를 A서비스를 위한 서버에 저장하고 B서비스또한 A서비스에 session을 사용해야하는 경우가 생긴다. 즉, 사용자가 웹포탈에 들어와서 A, B서비스 둘다 사용하는데 동일한 세션정보를 사용해야하는 것이다. A서비를 위해 로그인을 하고 또 B서비스를 위해 다시 로그인을 또하지는 않는다.

Micro Service Architecture에서는 각 서비스가 돌아가는 메모리에 session을 저장하지 않고 외부의 cache service를 이용하여 공유할 수 있다.

즉 A, B서비스가 외부에 동일한 저장소를 바라보고 있으면 되는것이다.

[Redis](https://redis.io/)가 이런 역할을 한다. Map과 같이 key, value 쌍으로 데이터를 저장할 수 있다.

### 3. Ubuntu 14.04 LTS에 Redis 설치

#### 설치:

```sh
$ sudo apt-get install redis-server
```

#### 설치 확인:

```sh
$ redis-server --version
Redis server v=2.8.4 sha=00000000:0 malloc=jemalloc-3.4.1 bits=64 build=a44a05d76f06a5d9
```

#### password 설정 및 bind 설정:

아래 항목을 작성 한다.

```sh
$ sudo vi /etc/redis/redis.conf
bind 0.0.0.0 # bind
requirepass redisadmin # password
```

#### 재시작:

```sh
$ redis-cli -p 9078 shutdown
$ sudo redis-server /etc/redis/redis.conf
```

#### Redis 여러대 띄우기

redis.conf 를 여러개 만들어서 다음 항목들이 겹치지 않게 설정한다.

-	pidfile
-	port
-	logfile

```sh
$ sudo redis-server /etc/redis/redis-9078.conf
$ sudo redis-server /etc/redis/redis-9079.conf
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
