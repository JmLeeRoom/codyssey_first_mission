# Dockerfile 기반 커스텀 이미지 + 웹 서버 컨테이너 — 완료 기록

> 대응 체크리스트: [`CheckList.md`](../../CheckList.md) 섹션 7 "Dockerfile 기반 커스텀 이미지 제작" / 섹션 8 "포트 매핑 검증" / [`README.md`](../../README.md) 4.5, 4.6

## 어디서 온 파일인가

`/home/jmlee/Project/codyssey/ia-codyssey/developer-workstation-mission` 폴더에서 동일한 미션을 먼저 작업한 이력이 있었고, 그 결과물(Dockerfile, nginx 설정, 정적 사이트, compose.yaml)을 이 저장소의 `app/`로 그대로 옮겨왔다. `docker images`에 원인 불명으로 있던 `workstation-web:1.0`이 바로 그 폴더에서 빌드된 이미지였음이 이번에 확인됐다 (같은 Dockerfile로 이 저장소 안에서 다시 빌드했을 때 레이어가 100% 캐시 히트된 것으로 교차 검증).

```
app/
├── Dockerfile
├── .dockerignore
├── compose.yaml
├── nginx/
│   └── default.conf
└── site/
    ├── index.html
    └── styles.css
```

## 선택한 베이스와 커스텀 포인트

- **선택 방식**: (A) 웹 서버 베이스 이미지 활용 + 정적 콘텐츠·설정 교체
- **베이스 이미지**: `nginx:1.30-alpine`

```dockerfile
FROM nginx:1.30-alpine

LABEL org.opencontainers.image.title="developer-workstation-web" \
      org.opencontainers.image.description="Static NGINX site for the developer workstation mission"

COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY site/ /usr/share/nginx/html/

EXPOSE 80

HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -q -O /dev/null http://127.0.0.1/health || exit 1
```

| 커스텀 포인트 | 지시어 | 목적 |
| --- | --- | --- |
| ① NGINX 설정 교체 | `COPY nginx/default.conf ...` | `/health` 엔드포인트 추가, `X-Workstation-Lab` 커스텀 헤더 주입, `server_tokens off`로 버전 노출 차단 |
| ② 정적 콘텐츠 교체 | `COPY site/ ...` | NGINX 기본 환영 페이지 대신 미션 전용 랜딩 페이지 제공 |
| ③ 헬스체크 추가 | `HEALTHCHECK ... /health` | 컨테이너가 스스로 상태를 판정 → `docker ps` STATUS에 `(healthy)` 표시 |
| ④ 포트 문서화 | `EXPOSE 80` | 이 이미지가 80 포트를 쓴다는 것을 이미지 메타데이터로 명시 |
| ⑤ 이미지 메타데이터 | `LABEL org.opencontainers.image.*` | 이미지 용도·설명을 표준 라벨로 기록 |

> 제출 전 개인화하고 싶다면 `site/index.html`의 문구를 본인 표현으로 바꾸거나, `LABEL`에 본인 이름/날짜를 추가하는 정도로 가볍게 커스텀해도 된다. 지금 상태로도 5가지 커스텀 포인트가 명확해 요구사항은 충족한다.

## 빌드

```bash
$ cd app
$ docker build -t workstation-web:1.0 .
...
#6 [2/3] COPY nginx/default.conf /etc/nginx/conf.d/default.conf
#6 CACHED
#7 [3/3] COPY site/ /usr/share/nginx/html/
#7 CACHED
#8 naming to docker.io/library/workstation-web:1.0 done
```

## 실행 — 포트 매핑 (서로 다른 호스트 포트 2회)

```bash
$ docker run -d --name web-demo -p 8080:80 workstation-web:1.0
$ docker run -d --name web-demo-2 -p 8081:80 workstation-web:1.0

$ docker ps --filter name=web-demo
CONTAINER ID   IMAGE                 ...   PORTS                                     NAMES
685724e1461b   workstation-web:1.0   ...   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   web-demo-2
0802ee050a7a   workstation-web:1.0   ...   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web-demo
```

## 접속 확인

```bash
$ curl -i http://localhost:8080/
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 1051
X-Workstation-Lab: ready
...

$ curl -i http://localhost:8081/
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 1051
X-Workstation-Lab: ready
...

$ curl http://localhost:8080/health
ok
$ curl http://localhost:8081/health
ok
```

두 포트 모두 같은 이미지에서 200 OK + 동일 콘텐츠 + 커스텀 헤더(`X-Workstation-Lab`)까지 확인됐다 — "같은 이미지를 다른 호스트 포트로 동시에 띄울 수 있다"는 포트 매핑의 핵심을 그대로 보여준다.

## 지금 컨테이너가 떠 있는 상태 — 남은 건 브라우저 스크린샷뿐

`web-demo`(8080), `web-demo-2`(8081) 두 컨테이너를 실행해뒀다. 지금 브라우저로 아래 두 주소에 각각 접속해서, **주소창(포트 포함)과 페이지 내용이 함께 보이도록** 스크린샷 2장을 찍어 `docs/result/`에 저장하면 이 섹션이 완전히 끝난다.

- `http://localhost:8080`
- `http://localhost:8081`
![alt text](../result/image-6.png)
![alt text](../result/image-7.png)
캡처가 끝나면 정리(선택):

```bash
docker rm -f web-demo web-demo-2
```

## Compose로도 동일하게 확인됨 (보너스 섹션 14와 연결)

```bash
$ cd app
$ docker compose up -d
 Container app-web-1 ... Healthy
 Container app-probe-1 ... Started

$ docker compose ps
NAME          IMAGE                 SERVICE   STATUS
app-probe-1   alpine:3.23           probe     Up 3 seconds
app-web-1     workstation-web:1.0   web       Up 9 seconds (healthy)

$ curl -i http://localhost:8080/     # compose.yaml은 127.0.0.1:8080으로 고정 매핑
HTTP/1.1 200 OK ...

$ docker compose logs probe --tail 5
probe-1  | ok

$ docker compose down
```

`probe` 서비스가 `web` 서비스명을 그대로 호스트명(`http://web/health`)으로 접근해 성공한 것이 컨테이너 간 서비스 디스커버리(DNS) 증거다. 이 부분은 [`CheckList.md`](../../CheckList.md) 섹션 14(보너스: Compose 멀티 컨테이너)로 그대로 쓸 수 있다.

## Dockerfile 지시어 참고

| 지시어 | 역할 | 이번 파일에서의 사용 |
| --- | --- | --- |
| `FROM` | 베이스 이미지 지정 | `nginx:1.30-alpine` |
| `COPY` | 호스트 파일을 이미지 안으로 복사 | 설정 파일·정적 콘텐츠 |
| `RUN` | 빌드 시점에 실행할 명령 | 미사용 (베이스 이미지에 NGINX 포함) |
| `ENV` | 환경 변수 설정 | 보너스: `compose.yaml`에서 `WORKSTATION_HOST_PORT`로 주입 |
| `EXPOSE` | 사용 포트 문서화 | `80` |
| `HEALTHCHECK` | 컨테이너 자체 상태 판정 | `/health` 폴링 |
| `CMD` / `ENTRYPOINT` | 시작 시 실행할 기본 명령 | 베이스 이미지의 `/docker-entrypoint.sh` 상속 |

**레이어 캐시 팁**: `RUN`·`COPY`는 각각 레이어 하나다. 자주 바뀌지 않는 것을 앞에, 자주 바뀌는 것을 뒤에 두면 캐시 재사용으로 빌드가 빨라진다. `.dockerignore`로 `.git`, 로그 등을 빌드 컨텍스트에서 제외해두었다.

## 기술 문서에 남길 것 (README.md 4.5 / 4.6)

- 선택한 베이스: `nginx:1.30-alpine`
- Dockerfile 경로: `app/Dockerfile`
- 커스텀 포인트 5가지 (위 표)
- `docker build` 성공 로그, `docker run`(8080/8081) + `docker ps` 결과
- `curl` 200 OK 결과 (8080, 8081 각각) — 완료
- 브라우저 스크린샷 (8080, 8081 각각) — **아직 필요**
