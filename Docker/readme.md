### Docker
- 클라우드 환경의 급속한 성장
- 클라우드 환경에서의 설치 및 배포의 어려움
- 서비스 운영 환경을 이미지로 생성해서 배포한다

### Virtual Machine vs Doceker
- 항상 guest OS를 설치해야 하는 VM &rightarrow; 이미지 용량이 커짐
- Docker는?
  - guest OS 필요 x
  - Docker Image에 서버 운영을 위해 필요한 것만 따로 설치
  - System Call 같은 OS 자원은 host system과 공유
  - 이미지 용량 줄어듬
  - 하드웨어 가상화 계층 x (mem, file system, network 속도 VM에 비해 매우 빠름)

### Docker Image, Container?
- base image ; 리눅스 배포판에 기본적인 것(유저랜드)만 설치된 이미지
  - OS에는 **kernel / user** space 존재
  - 유저랜드란, user space에서 실행되는 파일과 라이브러리
    - (부팅에 필요한 최소 실행 파일 및 라이브러리, 패키징 시스템)
- **docker image** 라고 하면, 베이스 이미지에 필요한 것들을 설치한 뒤 파일 하나로 만든 것
- docker image는 베이스 이미지에서 *바뀐 부분만 이미지로 생성*하고, 실행할 때는 각각을 합쳐 실행
  - 즉, 바뀐 부분만 이미지로 생성한 뒤, 부모 이미지를 계속 참조
- **docker container** 라고 하면, 이미지를 실행한 상태를 의미. 
  - 하나의 이미지로 여러개의 컨테이너 생성 가능

### 설치
- Ubuntu
  - 설치 명령어
    ```
     sudo apt-get update
     sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```
  - 잘 설치되었는지 확인하기 위해 hello-world 이미지 실행
    ```
      sudo docker run hello-world
    ```
  - 한방에 설치가 안된다면, 아래의 절차를 차근차근!
    - 혹시 old version이 있으면 삭제
      ```bash
      sudo apt-get remove docker docker-engine docker.io containerd runc
      ```
    - apt 패키지 색인을 업데이트하고, apt가 https를 통해 저장소를 사용할 수 있도록 관련 패키지 설치
      ```
      sudo apt-get update

      sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
      ```
    - 도커의 공식 GPG키 추가
      ```
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
      ```
    - 안정적인 저장소를 설치하려면 nightly, test 대신 stable을 사용
      ```
      sudo add-apt-repository \
         "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
         $(lsb_release -cs) \
         stable"
      ```
    - 도커 설치
      ```
       sudo apt-get update
       sudo apt-get install docker-ce docker-ce-cli containerd.io
      ```

- [centos](http://www.kwangsiklee.com/2017/07/centos%EC%97%90%EC%84%9C-docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/)

### 주요 명령어들
- **[주의]** : root 권한 필요, 혹은 현재 계정을 docker 그룹에 포함
```
sudo usermod -aG docker ${USER}
sudo service docker restart
```
- docker hub로 부터 ubuntu **이미지 검색**
```
docker search ubuntu
```
- docker hub로 부터 **이미지(이미지이름:태그) 받기**
  - latest는 최신
  - 당연히 centos에서 ubuntu 이미지 실행 가능
```
docker pull ubuntu:latest
```
- **이미지 목록 출력**, 추가 옵션으로 이름 넣을 수 있음
```
docker images 
```
- **컨테이너 생성**
  - 이미지를 컨테이너화 한 이후에 bash셸 실행하기
  - docker run <옵션> <이미지 이름> <실행할 파일>
  - i, t 옵션 지정시 실행된 bash셸에 입출력 가능 (물론 -it 로 쓸 수 있음)
  - p, P 옵션을 이용해 포트 바인딩을 시킬 수 있다.
  - d 옵션으로 백그라운드 실행 가능하다.
    - 백그라운드로 실행하지 않으면 컨테이너 안에서 foreground로 실행되고 있는 프로세스의 출력화면이 보인다
      - 컨테이너 내부에 foreground로 실행되는 프로세스가 없다면 
        바로 exit된다
  - [run 옵션](http://pyrasis.com/book/DockerForTheReallyImpatient/Chapter20/28)
    ```
    docker run --name hello --cpus="4" -d -P myappImage:test
    ```
```
docker run -i -t --name hello ubuntu /bin/bash
```
- **컨테이너 목록 확인**
  - a옵션 ; 정지된 컨테이너까지 보여줌
```
docker ps -a
```
- **컨테이너 시작**
  - 재시작은 restart
```
docker start hello
```
- **컨테이너에 접속**
  - 생성시 /bin/bash를 실행해야만 쉘에 접속해 명령을 입력할 수 있고
  - 그렇지 않다면 서버의 출력만 볼 수 있다.
  - 접속 후 **ctrl + D** 를 입력하면 컨테이너가 정지되고
  - **ctrl + P, ctrl + Q**를 차례대로 입력함으로써 컨테이너를 정지하지 않고 빠져나올 수 있다.
```
docker attach hello
```
- /bin/bash를 통하지 않고, **외부에서 컨테이너 안의 명령 실행**
  - docker exec <컨테이너 이름> <명령> <매개 변수>
  - 외부에서 apt-get, yum을 이용한 패키지 설치 혹은 데몬 실행시 활용 가능!
```
docker exec hello echo "hello world"
```
- run 할 때, /bin/bash를 명시하지 않아도, 다음과 같이 접속할 수 있네
```
docker exec -it [CONTAINER ID] /bin/bash
```
- **컨테이너 정지**
  - docker stop <컨테이너 이름>
```
docker ps 후
docker stop hello
```
- **컨테이너 삭제**
```
docker rm hello
```
- **이미지 삭제**
  - docker rmi <이미지 이름>:<태그>
```
docker rmi ubuntu:latest
docker images
```

### 그 외 명령어
- 이미지 history 조회
```
docker history <이미지이름>:<태그>
```
- 컨테이너로 부터 호스트로 파일 꺼내기(복사)
```
docker cp <컨테이너이름>:/etc/nginx/nginx.conf ./
```
```
docker cp ./abc.txt containername:/abc.txt
docker containername:/abc.txt ./abc.txt
```
- 현재 컨테이너의 변경사항 이미지로 생성
  - 다른 명령어에서도 마찬가지이지만 컨테이너 이름엔 ID를 적어도 됨
```
docker commit <옵션> <컨테이너 이름> <이미지 이름>:<태그>
docker commit -a "Jein Song <jeinsong@zum.com>" -m "커밋 메시지" node-nginx node-nginx-image:0.2
```
- 컨테이너의 변경 부분 확인(생성 이미지 기준으로 실행 후 변경된 것들)
```
docker diff <컨테이너 이름>
```
- 이미지 혹은 컨테이너 세부 정보 출력
```
docker inspect node-nginx
```

## docker-compose로 여러 컨테이너 관리하기
  - docker-compose.yml을 정의하고 아래의 명령어로 명시된 컨테이너를 한방에 키고 끌 수 있는 기능!!
    ```
    docker-compose up -d
    docker-compose down
    ```
    - docker와 별도로 설치해줘야 한다. centos라서 curl을 통해 [설치](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-centos-7)했음
    - docker-compose 파일 버전은 [링크](https://docs.docker.com/compose/compose-file)에서 시스템의 도커 버전에 따라 선택
  - 실제 사용했던 docker-compose 구성
    ```
      version: "3.5"
      services:
        worker:
          image: facethumb:alpha

        proxy: 
          build:
            context: ./nginx/
          image: proxy:latest
          ports:
            - '22223:80'

    ```
    - 참고로 compose 하는 법을 모를 때는 각 이미지를 각각 빌드한 뒤 아래와 같은 과정을 거쳤음...  
      (워커가 늘어나면 늘어나는대로.. 일일히 하나하나 키고, 끄고, )
      - compose는 디폴트로 네트워크가 구성되서 서비스들끼리는 통신이 가능하다고 들었는데 실제로 그렇다.
      ```
      docker network create facethumb-network
      docker run --name facethumb1 -d -P --cpus="4" --network facethumb-network facethumb:alpha
      docker run --name facethumb2 -d -P --cpus="4" --network facethumb-network facethumb:alpha
      docker run --name proxy -d -p 22223:80 --network facethumb-network proxy:test
      ```
    - 이후에 docker-compose up --scale worker=3 -d 명령을 통해 proxy 컨테이너를 3대 띄우고
    - 모두 한방에 종료하는 방법은 docker-compose down
    - 매우 간편하다