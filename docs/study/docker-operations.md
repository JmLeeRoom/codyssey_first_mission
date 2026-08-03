# Docker 기본 운영 명령 기록

> 대응 체크리스트: [`CheckList.md`](../../CheckList.md) 섹션 5 "Docker 기본 운영 명령 수행" / [`README.md`](../../README.md) 4.4

## 이미지 목록 확인

```bash
$ docker images
IMAGE                 ID             DISK USAGE   CONTENT SIZE   EXTRA
alpine:3.23           fd791d74b689         13MB         3.93MB
alpine:latest         28bd5fe8b56d         13MB         3.93MB
hello-world:latest    c3cbe1cc1aa5       25.9kB         9.49kB    U
ubuntu:latest         3131b4cc82a7        161MB         45.3MB
workstation-web:1.0   cb5ff185b423       92.7MB         26.1MB
```

## 컨테이너 목록 확인

```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
# 실행 중인 컨테이너 없음

$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED      STATUS                  PORTS     NAMES
9d4b13a3ce9f   hello-world   "/hello"   3 days ago   Exited (0) 3 days ago             compassionate_gates
```

`docker ps`는 실행 중인 것만, `docker ps -a`는 종료된 것까지 전부 보여준다는 차이가 실제로 확인된다 (hello-world는 실행 즉시 종료되는 이미지라 `-a`에만 나타남).

## 컨테이너 로그 확인 — `docker logs`

가장 기본형은 컨테이너 이름 또는 ID를 붙이는 것이다. **컨테이너가 이미 종료된 상태여도 로그는 그대로 남아있어 확인 가능**하다 (도커 데몬이 컨테이너별 stdout/stderr를 별도 보관하기 때문).

```bash
$ docker logs compassionate_gates
# 또는 이름 대신 ID로: docker logs 9d4b13a3ce9f

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

hello-world 이미지는 실행되면 안내 메시지를 stdout에 출력하고 바로 종료되므로, 위 명령으로 그 메시지 전체를 다시 볼 수 있었다.

자주 쓰는 옵션:

| 옵션 | 용도 |
| --- | --- |
| `-f`, `--follow` | 실시간으로 새 로그를 계속 스트리밍 (실행 중인 컨테이너에 유용, 종료하려면 `Ctrl+C`) |
| `--tail N` | 마지막 N줄만 출력 |
| `-t`, `--timestamps` | 각 줄 앞에 타임스탬프 표시 |
| `--since`, `--until` | 특정 시간 이후/이전 로그만 필터링 |

캡처용으로는 `-f` 없이 기본 명령(한 번 출력하고 끝)이 스크린샷/로그 기록에 더 적합하다.

## 컨테이너 리소스 사용량 확인 — `docker stats`

`docker stats`는 **현재 실행 중인 컨테이너**의 CPU/메모리/네트워크/디스크 I/O를 보여준다. 지금은 `docker ps` 결과가 비어있어(실행 중인 컨테이너 없음) 의미 있는 값이 나오지 않는다 — 먼저 뭔가 하나를 백그라운드로 띄워야 한다.

```bash
# 리소스 확인용으로 오래 떠있을 컨테이너 하나 실행
# (섹션 6 "컨테이너 실행 심화 실습"에서 쓸 ubuntu 컨테이너와 겸용해도 됨)
$ docker run -d --name stats-demo ubuntu sleep infinity
7aeb016abfab2737562bac45b4962622f8692de287a9eb6f52ffceeca538fa

$ docker stats --no-stream
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O        BLOCK I/O   PIDS
7aeb016abfab   stats-demo   0.00%     1.719MiB / 345.8GiB   0.00%     2.9kB / 126B   0B / 0B     1
```

- `docker stats`는 기본적으로 화면이 계속 갱신되는 대시보드 형태라 로그로 캡처하기 어렵다 — **`--no-stream`을 붙이면 현재 값을 한 번만 찍고 종료**되어 캡처에 적합하다.
- 특정 컨테이너만 보고 싶으면 `docker stats stats-demo --no-stream`처럼 이름을 지정한다.
- 확인이 끝나면 `docker stop stats-demo` / `docker rm stats-demo`로 정리하거나, 섹션 6 실습에 이어서 계속 사용해도 된다.

## 이미지 다운로드 — `docker pull`

```bash
$ docker pull alpine:latest
latest: Pulling from library/alpine
Digest: sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
Status: Image is up to date for alpine:latest
docker.io/library/alpine:latest
```

이미 로컬에 있는 이미지라 "Status: Image is up to date"가 뜬다. 이 메시지 자체가 "이미지 다운로드 명령을 정상적으로 실행했다"는 증거가 된다 (완전히 새 이미지를 받으면 레이어별 `Pulling fs layer` 진행률이 출력된다).

## 컨테이너 중지 — `docker stop`

```bash
$ docker stop stats-demo
stats-demo

$ docker ps -a --filter name=stats-demo
CONTAINER ID   IMAGE     COMMAND            CREATED      STATUS                     PORTS     NAMES
7aeb016abfab   ubuntu    "sleep infinity"   ...          Exited (137) ...                     stats-demo
```

`docker stop`은 컨테이너에 SIGTERM(안 되면 SIGKILL)을 보내 정상 종료시킨다. `STATUS`가 `Up ...`에서 `Exited (137) ...`로 바뀐 것이 중지 증거다 (137 = 128+9, SIGKILL로 종료됐다는 뜻 — `sleep infinity`는 SIGTERM을 무시해 결국 강제 종료된 것으로 보인다).
