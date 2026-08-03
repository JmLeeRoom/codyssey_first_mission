# 바인드 마운트 실습 — 완료 기록

> 대응 체크리스트: [`CheckList.md`](../../CheckList.md) 섹션 9 "바인드 마운트 & 볼륨 영속성 검증" (바인드 마운트 부분) / [`README.md`](../../README.md) 4.8
> 이전 실습 폴더에도 이 부분(`logs/05-bind-mount.txt`)은 만들어진 적이 없어 새로 진행했다. `app/site/`를 그대로 활용했다.

## 바인드 마운트란

**바인드 마운트**는 호스트의 특정 경로를 컨테이너 내부 경로에 그대로 연결하는 방식이다. [Docker 볼륨](./volume-guide.md)과 달리 Docker가 관리하는 별도 저장 공간이 아니라, **내가 지정한 호스트 폴더 자체**가 컨테이너 안에서 보인다 — 그래서 호스트에서 파일을 고치면 이미지를 다시 빌드하지 않아도 컨테이너 쪽에 즉시 반영된다.

## 실행 명령

`app/site/`를 nginx 컨테이너의 콘텐츠 경로에 바인드 마운트했다 ([`dockerfile-web-server-guide.md`](./dockerfile-web-server-guide.md)에서 빌드한 `workstation-web:1.0` 사용).

```bash
$ cd /home/jmlee/Project/codyssey/codyssey_first_mission/app
$ docker run -d --name bind-demo -p 8082:80 \
  -v "$(pwd)/site:/usr/share/nginx/html" \
  workstation-web:1.0
dbcabb297af55e1db9c7b21aa3f4dd22054ce48341ed5e4be325559aeeca3a08
```

- `-v 호스트경로:컨테이너경로` 형식이다. 컨테이너 경로 앞에 `/`가 있으면 바인드 마운트(또는 볼륨), 이름만 있으면(예: `devlab-data-test:/app/data`) 이름 있는 볼륨으로 해석된다는 차이를 기억해둔다.
- `$(pwd)/site`처럼 **절대 경로**를 써야 한다 (상대 경로를 쓰면 예상과 다른 곳이 마운트될 수 있다).

## BEFORE — 마운트 직후 상태 확인

```bash
$ curl -s http://localhost:8082/ | grep "<h1>"
      <h1>컨테이너 웹 서버가 정상 실행 중입니다.</h1>
```

이미지에 원래 `COPY`로 들어있던 것과 같은 내용이지만, 지금은 **호스트의 `app/site/index.html`이 보이고 있는 것**이다 (바인드 마운트가 이미지 안의 원본 콘텐츠를 덮어썼다).

## 호스트에서 파일 수정

```bash
$ sed -i 's/컨테이너 웹 서버가 정상 실행 중입니다./바인드 마운트 반영 테스트 - 재빌드 없이 바뀝니다!/' site/index.html
```

## AFTER — 재빌드·재시작 없이 반영되는지 확인

```bash
$ curl -s http://localhost:8082/ | grep "<h1>"
      <h1>바인드 마운트 반영 테스트 - 재빌드 없이 바뀝니다!</h1>
```

컨테이너를 재시작하거나 이미지를 다시 빌드하지 않았는데도 내용이 바뀐 것이 바인드 마운트의 핵심 증거다.

![바인드 마운트 실습 전체 터미널 로그](../result/image-8.png)

## 원복 및 정리

```bash
$ git -C /home/jmlee/Project/codyssey/codyssey_first_mission checkout -- app/site/index.html
error: pathspec 'app/site/index.html' did not match any file(s) known to git

$ docker rm -f bind-demo
bind-demo
```

`git checkout`이 실패한 이유는 `app/`가 아직 한 번도 커밋되지 않은(untracked) 상태라서다 — git이 되돌릴 "이전 버전"을 모른다. 그래서 `site/index.html`의 `<h1>`은 테스트 문구가 그대로 남아있었고, 이후 원래 문구(`컨테이너 웹 서버가 정상 실행 중입니다.`)로 직접 되돌려뒀다 — 포트 매핑 스크린샷(섹션 8)이 이 원래 문구를 기준으로 찍혀 있으므로 내용을 맞춰둔 것이다.

> **참고**: `app/`를 커밋해두면 다음부터는 `git checkout -- <파일>`로 안전하게 되돌릴 수 있다. 지금처럼 되돌릴 커밋이 없는 상태에서 실습용으로 파일을 고칠 때는, 고치기 전 원본을 별도로 복사해두거나 바뀐 내용을 기억해뒀다가 수동으로 되돌리는 방식이 필요하다.

## 기술 문서에 남길 것 (README.md 4.8)

- [x] 바인드 마운트 실행 명령 (`-v $(pwd)/site:/usr/share/nginx/html` 포함)
- [x] BEFORE `curl` 결과 (원본 `<h1>`)
- [x] 호스트 파일 수정 명령
- [x] AFTER `curl` 결과 (바뀐 `<h1>`) — 재빌드 없이 반영됨
- [x] 터미널 로그 스크린샷 (`image-8.png`)
