# Docker 볼륨 영속성 검증 — 완료 기록

> 대응 체크리스트: [`CheckList.md`](../../CheckList.md) 섹션 9 "바인드 마운트 & 볼륨 영속성 검증" (볼륨 부분) / [`README.md`](../../README.md) 4.7

이전 실습 폴더(`ia-codyssey/developer-workstation-mission`)에서 동일한 검증을 이미 수행한 기록이 있다. Docker 볼륨은 특정 프로젝트 파일에 의존하지 않는 범용 기능이라 그대로 재사용 가능하다.

## Docker 볼륨이란

컨테이너의 쓰기 레이어는 컨테이너와 함께 사라진다 — `docker rm`하면 그 안에 쓴 파일도 같이 사라진다. **이름 있는 볼륨(named volume)**은 Docker가 관리하는 호스트 영역(`/var/lib/docker/volumes/<name>/_data`)에 데이터를 두고 컨테이너 경로에 연결하므로, **컨테이너의 생명 주기와 데이터의 생명 주기가 분리**된다.

| 구분 | 바인드 마운트 | 이름 있는 볼륨 |
| --- | --- | --- |
| 저장 위치 | 내가 지정한 호스트 경로 | Docker가 관리하는 영역 |
| 주 용도 | 개발 중 소스 즉시 반영 | DB 데이터 등 영속 데이터 |
| 경로 의존성 | 호스트 경로에 종속 | 이름만 같으면 어디서든 동일 |
| 컨테이너 삭제 시 | 호스트 파일은 남음 | 볼륨은 남음 (`docker volume rm` 해야 삭제) |

## 검증 절차와 실제 출력

```bash
--- 1. Docker 볼륨 생성 ---
$ docker volume create devlab-data-test
devlab-data-test

--- 2. 볼륨 상세 정보 확인 ---
$ docker volume inspect devlab-data-test
[
    {
        "CreatedAt": "2026-07-30T02:01:37Z",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/devlab-data-test/_data",
        "Name": "devlab-data-test",
        "Scope": "local"
    }
]

--- 3. 컨테이너 A 생성 및 볼륨 마운트 후 데이터 기록 ---
$ docker run -d --name vol-container-1 -v devlab-data-test:/app/data alpine \
    sh -c "echo 'Persistent Volume Data Test 2026' > /app/data/test.txt && sleep 300"

--- 4. 삭제 전 데이터 및 체크섬 확인 ---
$ docker exec vol-container-1 cat /app/data/test.txt
Persistent Volume Data Test 2026
$ docker exec vol-container-1 sha256sum /app/data/test.txt
5a83c5d4befc3ba19199a6a7458a0e611c747732f944d6a1f95f2fa856abf4cc  /app/data/test.txt

--- 5. 컨테이너 A 강제 삭제 ---
$ docker rm -f vol-container-1
vol-container-1
$ docker ps -a --filter name=vol-container-1
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
(빈 목록 — 컨테이너가 완전히 삭제됨)

--- 6. 새 컨테이너 B를 동일 볼륨에 마운트하여 데이터 유지 검증 ---
$ docker run --rm -v devlab-data-test:/app/data alpine cat /app/data/test.txt
Persistent Volume Data Test 2026
$ docker run --rm -v devlab-data-test:/app/data alpine sha256sum /app/data/test.txt
5a83c5d4befc3ba19199a6a7458a0e611c747732f944d6a1f95f2fa856abf4cc  /app/data/test.txt

--- 7. 실습 볼륨 정리 ---
$ docker volume rm devlab-data-test
devlab-data-test
```

**핵심 증거**: 컨테이너 A를 `docker rm -f`로 완전히 삭제한 뒤, 전혀 다른 새 컨테이너 B에서 같은 볼륨(`devlab-data-test`)을 마운트했더니 파일 내용과 SHA-256 체크섬이 삭제 전과 **완전히 동일**했다. 이것이 "컨테이너는 사라져도 볼륨 데이터는 살아남는다"는 영속성의 직접적인 증거다.

## 다시 검증하고 싶다면

이 프로젝트에서 그대로 재현하려면 위 명령의 컨테이너/볼륨 이름을 그대로 복사해서 실행하면 된다 (이름이 겹치지만 신경 쓸 필요 없음 — 이미 정리(`docker volume rm`)까지 완료된 상태라 재사용 가능).
