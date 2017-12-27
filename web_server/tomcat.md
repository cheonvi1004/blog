Micro Service Architecture 서버 구축(2/4) - tomact8 설치
========================================================

[앞장](../web_server/README.md)에 이어...

### 2. Tomcat8 설치

#### Tomcat 설치

```sh
mkdir ~/download
cd ~/download
wget http://mirror.apache-kr.org/tomcat/tomcat-8/v8.0.48/bin/apache-tomcat-8.0.48.tar.gz
tar -xvf apache-tomcat-8.0.48.tar.gz
mv ~/download/apache-tomcat-8.0.48 ../tomcat8

cd ../tomcat8
```

서비스별 디렉토리를 만들고 톰켓구성요소를 복사한다.

```sh
mkdir /home/vagrant/apps/api /home/vagrant/apps/web-user /home/vagrant/apps/web-admin

cp -r ./{conf,logs,temp,webapps,work} /home/vagrant/apps/api/
cp -r ./{conf,logs,temp,webapps,work} /home/vagrant/apps/web-user/
cp -r ./{conf,logs,temp,webapps,work} /home/vagrant/apps/web-admin/
```

각 프로젝트 디렉토리의 conf/server.xml의 포트를 수정한다.

```sh
$ vi api/conf/server.xml
 <Server port="8005" shutdown="SHUTDOWN">
 ...
 <Connector port="9022" protocol="HTTP/1.1"
               connectionTimeout="20000"
              redirectPort="8443" />
 <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
 ...

$ vi web-user/conf/server.xml
 <Server port="8006" shutdown="SHUTDOWN">
 ...
 <Connector port="9080" protocol="HTTP/1.1"
               connectionTimeout="20000"
              redirectPort="8444" />
 <Connector port="8010" protocol="AJP/1.3" redirectPort="8444" />
 ...

$ vi web-admin/conf/server.xml
 <Server port="8007" shutdown="SHUTDOWN">
 ...
 <Connector port="9090" protocol="HTTP/1.1"
               connectionTimeout="20000"
              redirectPort="8445" />
 <Connector port="8011" protocol="AJP/1.3" redirectPort="8445" />
 ...
```

#### tomcat user 설정

\<tomat-users\> 안에 다음과 같은 role을 입력해준다.

```sh
$ vi /home/vagrant/apps/api/conf/tomcat-users.xml
$ vi /home/vagrant/apps/web-user/conf/tomcat-users.xml
$ vi /home/vagrant/apps/web-admin/conf/tomcat-users.xml

<tomcat-users>
...
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<role rolename="manager-script"/>
<user username="vagrant" password="vagrant" roles="manager-gui,manager-script,admin-gui"/>
</tomcat-users>
```

#### Tomcat manager 접근 설정

Tomcat Manager 접속 IP를 제한하기 위해 아래와 같은 설정을 작성하고 저장한다. allow의 작성시 여러 IP를 나열할 경우 | 를 구분자로 사용한다. 또한 '.'은 '.'로 입력한다. 각각의 CATALINA_BASE에 다음과 같이 작성해준다.

```sh
$ vi /home/vagrant/apps/api/conf/Catalina/localhost/manager.xml
$ vi /home/vagrant/apps/api/conf/Catalina/localhost/manager.xml
$ vi /home/vagrant/apps/api/conf/Catalina/localhost/manager.xml

<Context docBase="${catalina.base}/webapps/manager" antiReourceLocking="false" privileged="true">
        <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="192\.168\.105\.177|127\.0\.0\.1|192\.168\.105\.233" />
</Context>
```

jenkins를 통한 배포를 하기 위해서는 jenkins서버 IP도 등록해준다.

#### 각 프로젝트의 시작/종료 shell을 생성

각서비스 디렉토리에 CATALINA_HOME과 CATALINA_BASE를 환경에 맞게 수정한다.

```sh
$ vi ./startup.sh
    #!/usr/bin/env bash
    export CATALINA_HOME=/home/vagrant/tomcat8
    export CATALINA_BASE=/home/vagrant/apps/api
    echo $CATALINA_HOME
    cd $CATALINA_HOME/bin
    ./startup.sh
$ vi ./shutdown.sh
    #!/usr/bin/env bash
    export CATALINA_HOME=/home/vagrant/tomcat8
    export CATALINA_BASE=/home/vagrant/apps/api
    echo $CATALINA_HOME
    cd $CATALINA_HOME/bin
    ./shutdown.sh
```

메모리를 설정하려면 다음과 같은 옵션을 추가해준다.

```sh
export JAVA_OPTS="-Djava.awt.headless=true -server -Xms512m -Xmx1024m -XX:NewSize=256m -XX:MaxNewSize=256m -XX:PermSize=256m -XX:MaxPermSize=512m -XX:+DisableExplicitGC"
export CATALINA_OPTS="-Denv=product -Denv.servername=msa"

```

Tomcat을 구동하여 정상적으로 포트가 구동되었으면 성공!!!

##### 만약 Tomcat 구동시간이 지나치게 길어진다면,

CATALINA_BASE/logs/catalina.out 로그를 확인한다. 첫번 째

```sh
 org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory /xxx/xxx/xx
```

와 같은 로그 후에 멈춘듯이 진행이 안되거나 이후에 로그에 아래의 로그가 있다면

```sh
org.apache.catalina.util.SessionIdGeneratorBase.createSecureRandom Creation of SecureRandom instance for session ID generation using [SHA1PRNG] took [xxx,xxx] milliseconds.
```

session ID generation 시간이 위와같이 지나치게 길어지기 때문인데 이는 난수를 생성할 때 Entropy pool에 default size만큼의 bit가 충분하지 않다면 /dev/random은 블로킹이 되기 때문이다.

이에 대한 해결방법으로 startup.sh 에 아래의 문구를 추가한다.

```sh
export JAVA_OPTS="$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom"
```

참고 URL http://lng1982.tistory.com/261

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
