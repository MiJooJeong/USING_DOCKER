# Chapter 03. 새로운 시작
> 도커 컨테이너의 기본 구성 요소인 **도커파일(Dockerfiles)**과 **도커 레지스트리(DockerRegistries)**를 알아본다  

## 첫 번째 이미지 실행하기
> **컨테이너는 주 프로세스(main process)가 실행되는 동안에만 동작한다.** ::무슌말??::  

`docker run -i -t debian /bin/bash`
	- `docker run`: 컨테이너를 시작하는 역할
	- `-i -t`: `-i`와 `-t`플래그는 컨테이너와 ~tty 모드~ 와 대화형 세션을 사용
		- TTY(Teletypewriter) : 리눅스 환경에서 일반적인 CLI 콘솔
	- `debian`: 사용하고자 하는 이미지의 이름. 드비안(Debian)은 리눅스 배포판의 가장 기본적인 버전이다
	- `/bin/bash`: `/bin/bash` 명령은 bash 쉘을 반환

## 기본 명령어
1. `docker run -h CONTAINER -i -t debian /bin/bash`
	- `-h`플래그를 이용하여 컨테이너에 새로운 호스트 이름 부여
		- 도커는 임의의 형용사와 유명한 과학자, 엔지니어, 해커 등의 이름을 붙여서 호스트의 이름을 만들어낸다. `—name`인수를 이용하면 직접 이름을 지정할 수도 있다. (`docker run —name boris debian echo “Boo”`)
2. 위 명령어로 생성한 컨테이너를 `docker ps` 명령어로 확인
![](Chapter%2003.%20%EC%83%88%EB%A1%9C%EC%9A%B4%20%EC%8B%9C%EC%9E%91/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202018-10-29%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.15.07.png)
	* `docker ps -a`: **중지된(stopped)** 컨테이너(공식적으로는 **종료된(exited)** 컨테이너라고 한다)를 포함한 모든 컨테이너의 목록을 확인할 수 있다.
		* 종료된 컨테이너는 `docker start`명령을 실행하면 재시작
3.  `docker inspect '컨테이너 이름'`
	- `docker inspect`를 실행하면서 컨테이너의 이름 또는 ID를 주면, 컨테이너에 대한 자세한 정보를 얻을 수 있다. (Inspect 명령에 대한 자세한 정보 : [docker inspect | Docker Documentation](https://docs.docker.com/engine/reference/commandline/inspect/)
4.  `docker logs xenodochial_bassi`: 현재 실행된 컨테이너의 log를 볼 수 있다.
5.  `docker rm '컨테이너 이름'`: 컨테이너 삭제
6.  **중지된 컨테이너 정리하기**(*자주사용됨*)
	- `docker rm -v $(docker ps -aq -f status=exited)`
	
7. 도커는 컨테이너에 UFS(Union file system)을 사용한다. UFS는 여러 개의 파일 시스템이 계층 구조로 마운트되어 하나의 파일 시스템처럼 보일 수 있도록 해 준다. 이미지의 파일 시스템은 읽기 전용 계층으로 마운트되고, 실행 중인 컨테이너에서 변경된 모든 내용들은 컨테이너에 마운트된 읽기-쓰기 계층에 쓰여진다 ::1도 모르겠당::

8.  도커화된 cowboy 웅용프로그램 생성
```
$ docker run -it --name cowsay --hostname cowsay debian bash
root@cowsqy:/# apt-get update
...
Reading package lists... done
root@cowsay:/# apt-get install -y cowsay fortune
...
root@cowsay:/# /usr/games/fortune | /usr/games/cowsay 
```
컨테이너를 이미지로 바꾸려면 `docker commit`명령을 사용하면 된다.
컨테이너가 실행 상태이건 중지 상태이건 상관 없이 동작한다. 명령을 실행할 때, 컨테이너의 이름(“cowsay”), 이미지 이름(“cowsayimage”), 이미지를 저장할 저장소(“test”)를 명시해 주어야 한다.
	- `docker commit cowsay test/cowsayimage`: 이미지 생성 후 ID 반환
	- `docker run test/cowsayimage /usr/games/cowsay “Moo”`
```
 _____
< Moo >
 -----
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||	
```
	* 위와 같은 과정을 수동으로 반복하지 않기 위해서는 **도커파일**을 이용하여 이미지 생성을 자동화해야한다.

## 도커파일로 이미지 만들기
> 도커파일은 도커 이미지를 생성하기 위해서 사용될 수 있는 일련의 절차들을 담고 있는 텍스트 파일이다.  

1. 새로운 폴더와 파일을 생성
```
mkdir cowsay
cd cowsay
touch Dockerfile
vi Dockerfile
```

2.  Dockerfile에 아래의 내용을 추가
```
FROM debian:wheezy

RUN apt-get update && apt-get install -y cowsay fortune
```

3. `docker build -t test/cowsay-dockerfile .`: `docker build`명령을 실행하면 이미지 생성
4.  `docker run test/cowsayimage /usr/games/cowsay “Moo”`: 앞선 예제와 같은 방법으로 이미지 실행 가능!

### 이미지, 컨테이너 그리고 UFS(Union File System or Union Mount)
- UFS는 여러 개의 파일 시스템들을 겹칠 수 있도록 해 주는데, 사용자에게는 하나의 파일 시스템처럼 보이게 된다. 폴더는 여러 개의 파일 시스템에 있는 파일들을 포함할 수 있지만 두개의 파일이 동일한 경로를 가지고 있게되면 마지막으로 마운트된 파일이 보여지고 이전에 마운트된 파일은 숨겨진다.
- 도커 이미지는 여러 계층(layers)으로 구성되어 있다. 각 계층은 읽기전용 파일 시스템에 있다. 계층은 도커파일에 있는 명령마다 생성되며 이전 계층 위에 위치하게 된다. 이미지가 **컨테이너**로 만들어지면, 도커 엔진은 이미지를 받아서 읽기-쓰기가 가능한 파일 시스템 위에 추가한다.
- 컨테이너의 상태는 **생성(created), 재시작(restarting), 실행 중(running), 일시 중지(paused), 종료(exited)**등으로 구분된다
	- 컨테이너가 “생성된” 상태라는 의미는 `docker create`명령으로 시작되었지만, 아직 완전하게 시작되지 않은 상태
	- 종료 상태는 일반적으로 “중지된(stopped)” 상태를 나타내며, 컨테이너 내부에서 실행되는 프로세스가 없음을 의미한다(이는 “생성된”상태의  컨테이너에서도 마찬가지이지만, 종료된 컨테이너는 적어도 한 번은 이미 실행이 되었다는 점이 다르다.)

- ENTRYPOINT 도커파일 설정은 `docker run`명령으로 전달되는 모든 인자를 처리하기 위해 사용되는 실행 파일을 명시할 수 있도록 해준다.
	- entrypoint.sh라는 파일을 만들고 내용을 추가한 다음 도커파일과 같은 디렉토리에 저장한다.
```
#!/bin/bash
if [ $# -eq 0 ]; then
    /usr/games/fortune | /usr/games/cowsay
  else
    /usr/games/cowsay "$@"
fi	
```
	`chmod +x entrypoint.sh`를 실행하여 파일을 실행할 수 있도록 설정.
		- 인자 없이 호출되는 경우, 위의 스크립트는 입력값을 fortune에서 cosway로 전달하게되고, 인자가 전달된 경우에는 주어진 인자를 이용하여 cowsay 응용프로그램을 호출
	- Dockerfile도 아래와 같이 수정.
```
FROM debian

RUN apt-get update && apt-get install -y cowsay fortune
COPY entrypoint.sh /

ENTRYPOINT ["/entrypoint.sh"]	
```
	- 이미지를 빌드한 후에 컨테이너를 실행

## 레지스트리를 이용한 작업
### 레지스트리, 저장소, 이미지, 태그
> 이미지는 계층적인 구조로 저장된다.  

	- **레지스트리(Registry)**
	이미지를 운영하고 배포하는 역할을 담당하는 서비스. 기본 레지스트리는 도커 허브.
	- **저장소(Repository)**
	관련된 이미지들(일반적으로 같은 응용프로그램 또는 서비스의 각기 다른 버전)의 집합.
	- **태그(Tag)**
	저장소에 있는 이미지에 붙여진 알파벳과 숫자로 된 구분자(예를 들면 14.04, stable)
	
	따라서 `docker pull amouat/revealjs:latest`명령은 도커 허브 레지스트리의 amouat/revealjs 저장소에 있는 latest 태그가 붙어 있는 이미지를 다운로드 한다!

- MAINTAINER 설정을 추가하여 이미지의 제작자와 연락처 설정.
`MAINTAINER John Smith <john@smith.com>`
- 이미지를 다시 빌드하고, 빌드한 이미지를 도커 허브에 업로드.
`docker build -t uuuuio/cowsay .`
`docker push uuuuio/cowsay`: 자동으로 latest 라는 태그가 붙음. 태그를 지정하고싶을 경우 `docker push uuuuio/cowsay:stable`과 같이 명시

## 개인 저장소
### 이미지 네임스페이스
	1. **user** 네임스페이스
		- uuuuio/cowsay와 같이 문자로 된 접두어와 / 로 이름이 구성되면, user 네임스페이스에 소속된다. 해당 사용자에 의해서 도커 허브에 업로드된 이미지.
	2. **root** 네임스페이스
		- 접두어나 / 가 없는 debian, ubuntu와 같은 이름은 root 네임스페이스에 속한다. 해당 이미지들은 도커 사에 의해 관리되며 일반적인 소프트웨어와 배포판의 공식 이미지들이다.
	3. 호스트 이름이나 IP가 접두어로 사용된 이름은 (도커 허브가 아닌)서드-파티 레지스트리에서 운영되는 이미지이다.

## Redis 공식 이미지 이용하기
`docker pull redis`

- Redis 컨테이너를 시작
	- `docker run --name myredis -d redis`: -d 인자는 컨테이너를 백그라운드에서 실행
	- 데이터베이스와 연결해야하는데 응용프로그램이 없기 때문에 `redis-cli`도구를 이용. 호스트에 `redis-cli`를 설치할 수도 있겠지만 `redis-cli`를 실행하기 위한 새로운 컨테이너를 시작하고 두 컨테이너를 연결하는 것이 좀 더 간편하다.
```
❯ docker run --rm -it --link myredis:redis redis /bin/bash
root@5f7fa8f6bd4d:/data# redis-cli -h redis -p 6379
redis:6379> ping
PONG
redis:6379> set "abc" 123
OK
redis:6379> get "abc"
"123"
redis:6379> exit
root@5f7fa8f6bd4d:/data# exit
exit		
```
		- `docker run`명령에 `—-link myredis:redis` 인자를 같이 사용하면 새로운 컨테이너와 기존의 “myredis” 컨테이너를 연결하고, 새로운 컨테이너 안에서 “myredis” 컨테이너를 “redis” 라는 이름으로 참조하는 작업을 수행.
		- 그 다음에는 Redis `ping` 명령을 실행하여 Redis 서버로 연결되었는지를 확인하고 `set`과 `put`을 이용하여 데이터를 추가하고 반환.

- **볼륨(volumes)**: 일반적인 UFS의 일부가 아닌, 호스트에 직접 마운트된 파일 또는 디렉토리들로 다른 컨테이너와 공유될 수 있으며, 모든 변경 사항들은 호스트 파일 시스템에 직접 쓰이게 된다.
	- 디렉터리를 볼륨으로 선언하는 방법
		1. Dockerfile 안에 VOLUME 설정 (`VOLUME /data`)
		2. `-v`플래그를 명시하여 `docker run`명령을 실행 
		(`docker run -v /data test/webserver`)
- Redis 컨테이너에서 백업 수행하기
```
$ docker run --rm -it --link myredis:redis redis /bin/bash
root@a3abcc15b486:/data# redis-cli -h redis -p 6379
redis:6379> set "persistence" "test"
OK
redis:6379> save
OK
redis:6379> exit
root@a3abcc15b486:/data# exit
exit
$ docker run --rm --volumes-from myredis -v $(pwd)/backup:/backup \debian cp /data/dump.rdb /backup/
$ ls backup
dump.rdb
```
		- `-v`: 호스트에 마운트하여 공유하게 되는 디렉터리
		- `--voluems-from`: Redis 데이터베이스 폴더를 공유할 새로운 컨테이


#Using Docker/Part 1. 배경 및 기초#