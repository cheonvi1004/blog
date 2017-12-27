[Ubuntu에 MariaDB 설치 - 설치 가이드
====================================

#### ubuntu 저장소를 통한 maria DB 설치

```sh
$ sudo apt-get update
$ sudo apt-get install mariadb-server
```

끝!!! 이게 끝이다.

하지만 실제 DB를 사용하고 외부에서 붙을 수 있도록 설정하려면 조금 손을 봐야한다.

```sh
$ sudo mysql_secure_installation
```

#### MariaDB 연결

```sh
$ mysql -u root -p
```

root로 설정했던 비밀번호를 입력하면 접속된다.

#### 사용자 및 권한 설정

mariaDB를 root로 접속한 후에 database및 user정보를 생성한다.

로컬 사용자

```mysql
root#] create user 'user'@'localhost' identified by 'password';
```

로컬 및 원격 사용자가

```mysql
root#] create user 'user'@'%' identified by 'password';
```

권한 설정

```mysql
root#] grant all privileges on database_name.* to 'user'@'%';
root#] flush privileges;
```

참고로 전체 database에서 설정하려며 'on \*.\*' 로 한다.

#### 원격접속 설정

원격 접속을 허용하게 하려면 /etc/mysql/my.cnf파일에서 bind-address 부분을 주석처리한다.

설정 변경 후 에 서비스를 재시작 해준다.

```sh
$ sudo service mariadb restart
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
