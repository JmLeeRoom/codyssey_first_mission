# 컨테이너 실행 심화 실습 — `exec` vs `attach`

> 대응 체크리스트: [`CheckList.md`](../../CheckList.md) 섹션 6 "컨테이너 실행 심화 실습" / [`README.md`](../../README.md) 4.6(예정)

## `hello-world` 실행

`docker logs compassionate_gates`(섹션 5 실습, [`docker-operations.md`](./docker-operations.md) 참고)로 이미 hello-world의 안내 메시지 전체를 확인했다 — 이 컨테이너가 hello-world 이미지 실행 증거다.

## `ubuntu` 컨테이너 내부 명령 수행 (`docker exec`)

```bash
$ docker exec stats-demo ls -la /
total 60
drwxr-xr-x    1 root root 4096 Aug  3 01:31 .
drwxr-xr-x    1 root root 4096 Aug  3 01:31 ..
-rwxr-xr-x    1 root root    0 Aug  3 01:31 .dockerenv
...
drwxr-xr-x   11 root root 4096 Jul 13 16:06 var

$ docker exec stats-demo echo "Hello from Inside Ubuntu Container!"
Hello from Inside Ubuntu Container!
```

`.dockerenv` 파일이 보이는 것 자체가 "지금 컨테이너 내부에 있다"는 표식이다 (호스트에는 이 파일이 없다).

## `exec` 종료 후에도 컨테이너가 유지되는지 확인

```bash
$ docker exec stats-demo bash -c "echo executing subshell; exit"
executing subshell

$ docker ps --filter name=stats-demo
CONTAINER ID   IMAGE     COMMAND            CREATED          STATUS          PORTS     NAMES
7aeb016abfab   ubuntu    "sleep infinity"   32 minutes ago   Up 32 minutes             stats-demo
```

`exec`로 만든 프로세스가 `exit`으로 끝나도 컨테이너 자체는 여전히 `Up` 상태다.

## `attach` vs `exec` — 구조로 이해하기

```
[docker attach]
 호스트 터미널 ──(직접 연결)──► 컨테이너 주 프로세스 (PID 1)
                                  └─ exit           → PID 1 종료 → 컨테이너 Exited
                                  └─ Ctrl+P, Ctrl+Q → 분리만     → 컨테이너 Up 유지

[docker exec]
 호스트 터미널 ──(새 프로세스)──► 컨테이너 내부 부 프로세스 (PID N)
                                  └─ exit → 그 프로세스만 종료 → PID 1 살아있음 → Up 유지
```

| 구분 | `docker attach` | `docker exec` |
| --- | --- | --- |
| 연결 대상 | 컨테이너 생성 시 지정된 **주 프로세스(PID 1)**의 stdin/stdout | 컨테이너 내부에 **새로 만든 부 프로세스** |
| `exit` 시 영향 | PID 1이 종료 → **컨테이너 정지(Exited)** | 그 프로세스만 종료 → **컨테이너 유지(Up)** |
| 안전하게 빠져나오기 | `Ctrl+P`, `Ctrl+Q` (detach, 컨테이너는 계속 실행) | `exit`로 충분 |
| 주요 용도 | 메인 프로세스의 출력을 실시간으로 붙어서 관찰 | 실행 중인 컨테이너 디버깅·점검 |

**컨테이너의 수명은 PID 1의 수명과 같다** — 이것이 두 명령의 차이가 생기는 근본 원인이다.

## 아직 실습하지 않은 것: `docker attach`로 직접 관찰하기

위 표는 개념 정리이고, 실제로 `attach` 후 `exit`했을 때 컨테이너가 `Exited`로 바뀌는 것과 `Ctrl+P, Ctrl+Q`로 분리했을 때 `Up`으로 남는 것을 **직접 눈으로 확인하는 실습**이 아직 남아있다. TTY가 필요해 자동화할 수 없고 손으로 해야 한다.

```bash
# 1) 대화형 TTY로 컨테이너 실행
docker run -dit --name attach-demo ubuntu bash

# 2) attach로 PID 1(bash)에 직접 연결
docker attach attach-demo
```
![alt text](../result/image-5.png)
- 이 상태에서 `exit`을 입력하면 → PID 1이 죽으므로 컨테이너가 즉시 `Exited`가 된다. `docker ps -a --filter name=attach-demo`로 확인.
- 대신 다시 `docker run -dit --name attach-demo-2 ubuntu bash` 후 `docker attach attach-demo-2`로 들어가 이번엔 `Ctrl+P`를 누르고 이어서 `Ctrl+Q`를 누르면(분리/detach) → 컨테이너는 `Up` 상태로 계속 살아있다. `docker ps --filter name=attach-demo-2`로 확인.

**주의**: `docker attach`한 셸 안에서 절대 `docker ...` 명령을 입력하지 않는다 — 그 순간 호스트가 아니라 **컨테이너 내부**에 있는 것이므로(프롬프트가 `root@<컨테이너ID>:/#` 형태로 바뀐다), 컨테이너 안에는 docker CLI가 없어 `command not found`가 난다. 두 실습(exit로 종료 / detach로 유지)을 각각 별도 컨테이너로 나눠서 진행하면 헷갈리지 않는다.
