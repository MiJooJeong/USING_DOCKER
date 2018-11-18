# Chapter 4. 도커의 기초

## 도커 아키텍처
![](https://www.aquasec.com/wiki/download/attachments/2854889/Docker_Architecture.png?version=1&modificationDate=1520172700553&api=v2)

- **도커 데몬(Docker Daemon)**: 컨테이너를 생성하고, 실행하고, 모니터링뿐만 아니라 이미지를 생성하고 저장하는 역할까지 담당. `docker daemon`명령을 실행하여 시작되며 해당 명령은 일반적으로 호스트 OS에 의해서 관리
- **도커 클라이언트(Docker Client)**: HTTP를 통하여 도커 데몬과 통신하는데 사용된다. 기본적으로 통신은 Unix domain socket을 통하여 이루어진다.
- **도커 레지스트리(Docker Registry)**: 이미지들을 저장하고 배포. 기본 레지스트리는 도커 허브로, “공식” 이미지라고 하는 수천 개의 공용 이미지들을 운영.

### 기반 기술
> 도커 데몬은 “실행 드라이버(execution driver)”를 이용하여 컨테이너를 생성.
> 실행 드라이버는 기본적으로 도커의 독자 **runc** 드라이버를 사용한다.
>
> - **runc**와 관련된 커널 기능
>   - **cgroups**는 컨테이너에 의해서 사용되는 리소스(ex. CPU, 메모리 사용량)를 관리하는 역할을 담당. 컨테이너를 일시중지(freezing), 일시중지 해제(unfreezing)하는 역할도 담당
>   - **namespaces**는 컨테이너를 격리시키는 역할 수행. 컨테이너의 파일 시스템, 호스트 이름, 사용자, 네트워킹, 프로세스 등을 나머지 부분과 분리시켜줌.
> 도커의 기반을 이루는 또 다른 주요 기술에는 UFS(Union File System)이 있다.
> UFS는 컨테이너의 계층들을 저장하는 데 사용된다.

## 이미지가 만들어지는 과정
### 빌드 컨텍스트
> `docker build`명령을 실행하려면 도커파일과 **빌드 컨텍스트(build context)**가 있어야한다. 
> 빌드 컨텍스트는 도커 파일 안에 명시된 `ADD`나 `COPY`같은 설정에서 참조할 수 있는 일련의 로컬 파일들과 디렉터리들을 말한다.
> - `docker build -t test/cowsay-dockerfile .`이라는 `build`명령에서, 컨텍스트를 현재 작업 중인 디렉터리로 설정하겠다는 의미로 `’.’`을 사용.
>   해당 경로에 있는 모든 파일들과 디렉터리들은 `build` 컨텍스트를 만들고, 		
>   `build`과정의 일부로 도커 데몬에 전달된다.
### 이미지 계층
> 도커파일에 있는 각 설정들은 컨테이너를 시작하는데 사용될 수 있는 새로운 이미지 **계층(layer)**을 만든다. 새로운 계층은 이전 계층의 이미지를 이용하는 컨테이너를 시작함으로써 생성된다. 도커파일 설정을 실행하고 새로운 이미지를 저장하는 방식으로 새로운 계층이 만들어지게 된다.
### 캐싱
> 또한 이미지 생성 속도를 높이기 위해서 도커는 모든 계층을 캐시에 저장하고 있다. 
> `RUN`설정이 여러번 호출되더라도 같은 결과를 반환한다는 보장이 없어도 **캐시에는 저장됨**을 의미한다.
> 캐시를 무효화할 필요가 있다면, `no-cache`인자를 이용하여 `docker build`명령을 실행하면 된다. 
### 기본 이미지
> 일반적으로 나만의 이미지를 만드는 것보다는 공식 이미지를 이용하는 것이 훨씬 낫다. 다른 사람들이 작업한 것과 컨테이너 안에서 응용프로그램이 동작하는데 가장 적합한 방법을 찾아낸 경험 등을 그대로 적용할 수 있다는 장점이 있다.
>
> - 이미지 다시 빌드하기 : `docker build`명령이 실행되면, 도커는 `FROM`설정을 읽어 이미지가 로컬에 없으면 가져오기를 시도하게 된다. 만약 로컬에 이미지가 있다면 도커는 새로운 버전이 있는지 확인하지 않고 바로 해당 이미지를 사용한다. 즉, `docker build`명령만 실행한다고 해서 이미지를 최신의 상태로 유지해 주는 것은 아니다. 따라서 모든 상위 이미지들에 대해서 `docker pull`명령을 명시적으로 실행해 주어야 하거나, 최신의 버전을 다운로드하기 위한 빌드 명령을 강제로 실행하도록 하기 위해 상위 이미지들을 삭제해야 한다..!
### 도커파일 설정
> 도커파일의 주석은 ‘#’으로 시작할 수 있다.
> - 실행 방식 vs 쉘 방식
>   : 여러 설정들(`RUN, CMD, ENTRYPOINT`)이 **실행**방식과 **쉘**방식을 모두 	지원한다. 
>   실행 방식은 JSON 배열(예를 들면 [“executable”, “param1”, 	“param2”])을 	받게 되는데 배열이 첫 번째 항목은 실행 파일의 이름이, 나머지 항목은 실행 파일이 
>   실행될 때 사용되는 매개 변수로 사용된다. 
>   쉘 방식은 자유로운 형식의 문자열을 사용하는데 해당 문자열을 `/bin/sh -c`로 	전달하여 처리가 된다.
>   쉘 변조 문자를 피하고자 하거나 이미지에 `/bin/sh`가 없다면 실행 방식을 사용.
> - 참고문서 : [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

## 컨테이너를 외부와 연결하기
> 컨테이너 안에서 웹 서버를 실행하고 있을 경우, 외부에서 웹 서버로 접근할 수 있도록 하려면 `-p` 또는 `-P` 명령으로 포트를 “게시”해야한다.
```
$ docker run -d -p 8000:80 nginx
c3b21572f6211258add89ae9831a723fdbd1e60aa4f0e10d8f16b067fb4df64d
$ curl localhost:8000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```
	- `-p 8000:80`인자는 호스트의 8000번 포트를 컨테이너의 80 포트로 전달하도록 도커에게 알려준다. 이 방법 외에도 `-P`인자를 이용하면 호스트로 전달할 여분의 포트를 도커가 자동으로 선택하도록 할 수 있다.
```
$ ID=$(docker run -d -P nginx)
$ docker port $ID 80
0.0.0.0:32771
$ curl localhost:32771
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>	
```
	- `-P`명령의 가장 큰 장점은 할당된 포트들을 계속해서 파악하고 있어야 할 필요가 없다는 데 있다. 특히 여러 개의 컨테이너들이 포트를 게시하고 있다면 더더욱 중요하다! 이런 경우 `docker port`명령을 이용하면 도커가 할당한 포트들을 확인할 수 있다.

## 컨테이너 연결하기
> **도커 링크(links)**는 같은 호스트에 있는 컨테이너들끼리 통신할 수 있는 가장 간단한 방법이다. 도커의 기본 네트워킹 모델을 이용하는 경우 컨테이너 간의 통신은 내부 도커 네트워크를 통해서 이루어지게 된다. ~즉, 통신은 호스트의 네트워크로는 노출되지 않는다. ::무슨말이지??::~

	- `docker run`명령을 실행하면서 `--link CONTAINER:ALIAS`를 인자로 줌
		- `CONTAINER`: 링크 컨테이너의 이름
		- `ALIAS`: 마스터 컨테이너 내부에서 링크 컨테이너를 참조할 때 사용하는 로컬 이름
```
$ docker run -d --name myredis redis
ceb91755725a013035a3fb8dc45fcb0340baf1f18448692b6190ae9b58e046a9
$ docker run --link myredis:redis debian env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=7fa2a3bb09f9
REDIS_PORT=tcp://172.17.0.4:6379
REDIS_PORT_6379_TCP=tcp://172.17.0.4:6379
REDIS_PORT_6379_TCP_ADDR=172.17.0.4
REDIS_PORT_6379_TCP_PORT=6379
REDIS_PORT_6379_TCP_PROTO=tcp
REDIS_NAME=/gracious_keldysh/redis
REDIS_ENV_GOSU_VERSION=1.10
REDIS_ENV_REDIS_VERSION=5.0.0
REDIS_ENV_REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-5.0.0.tar.gz
REDIS_ENV_REDIS_DOWNLOAD_SHA=70c98b2d0640b2b73c9d8adb4df63bcb62bad34b788fe46d1634b6cf87dc99a4
HOME=/root
```
	- 도커는 링크 컨테이너로부터 환경 변수를 가져오게 되는데(`REDIS_ENV`), 유용하게 사용될 수 있지만 API 토큰이나 데이터베이스 비밀번호 같은 민감한 정보를 저장하기 위해 환경 변수를 사용하는 경우에는 주의해야 한다.
	- 도커 링크의 가장 큰 단점은 **정적**이라는 것이다. 링크는 컨테이너가 재시작되더라도 그대로 유지되어야 하는데, 링크 컨테이너가 대체되면 갱신되지 않는다. 
  또한 마스터 컨테이너가 시작되기 전에 링크 컨테이너가 시작되어 있어야한다. 
	즉, 양방향 링크는 불가능하다.

## 볼륨과 데이터 컨테이너를 이용한 데이터 관리하기
> 도커 볼륨은 컨테이너의 UFS가 아닌 디렉터리로, 컨테이너에 **바인드 마운팅된(bind mounted)**호스트의 일반 디렉터리를 말한다.
> - 볼륨 초기화의 세가지 방법
>   1. `-v`플래그를 이용하여 볼륨을 선언 : 
>     `$ docker run -it —name container-test -h CONTAINER -v /			data Debian /bin/bash`
>     - 해당 방법을 이용하면, 컨테이너 내부의 */data*디렉터리가 볼륨으로 생성되며, */data* 디렉터리 내부에 있는 모든 파일은 볼륨으로 복사.
>     - `docker inspect`명령을 실행하면, 볼륨이 호스트의 어디에 위치하는지 확인 가능하다.
>       `docker inspect -f {{.Mounts}} container-test`
>   2. 도커파일에서 `VOLUME` 설정을 이용 :
```
FROM Debian:wheezy
VOLUME / data		
```
>	3. `docker run`명령을 `-v`인자와 함께 사용하면서, 
>	`-v HOST_DIR:CONTAINER_DIR`로  명시하여 호스트에서 바인드하려는 디렉터리를 명시적으로 지정
>	- 도커파일로는 불가능
>	- `$ docker run -v /home/Adrian/data:/data debian ls /data`
>	- 위 두 가지 방법과는 달리 이미지에 있는 파일들은 볼륨으로 복사되지 않으며, 볼륨은 도커에 의해서 삭제되지도 않는다.
### 데이터 공유하기
> `docker run`을 실행하면서 `—-volumes-from CONTAINER`인자를 사용하면, 컨테이너 간에 데이터를 공유할 수 있다.
```
$ docker run -it -h NEWCONTAINER —-volumes-from container-test debian /bin/bash
```
### 데이터 컨테이너
> 보통 컨테이너들 간의 데이터를 공유하기 위하여, 일반적으로 해당 용도로만 사용되는 컨테이너인 **데이터 컨테이너(data containers)**를 생성한다.
> - PostgreSQL 데이터베이스 용도의 데이터 컨테이너를 생성
>   `$ docker run —name dbdata posters echo “Data-only container 	for Postgres”`
>   - `postgres`이미지를 이용하여 컨테이너를 생성하고 `echo`명령이 실행되기 전에 이미지에 정의된 모든 볼륨을 초기화하고 종료하게 된다. 실행 상태로 두면 리소스 낭비를 초래하기 때문에 데이터 컨테이너를 실행된 상태로 둘 필요는 없다.
#### 볼륨 삭제하기
> 볼륨은 다음과 같은 경우에만 삭제된다
> - `docker rm -v`를 이용하여 컨테이너가 삭제되는 경우
> - `docker run`에 `—-rm`플래그가 같이 사용되는 경우
> - 볼륨에 연결된 컨테이너가 없는 경우
> - 볼륨에 명시된 호스트 디렉터리가 없는 경우

## 일반적인 도커 명령어
	- 참고자료 : [도커 공식 웹사이트 문서](https://docs.docker.com/)
### run 명령어
> 새로운 컨테이너를 시작할 때 사용되는 가장 일반적인 명령어인 `docker run`명령어는 컨테이너의 시작과 관련된 명령어인 만큼 가장 복잡하면서 상당히 많은 인자를 지원.
> 인자에서 지원되는 대표적인 기능에는 이미지를 실행하는 방법, 도커파일 설정, 네트워크 설정, 컨테이너의 권한과 리소스 설정 등이 있다.

