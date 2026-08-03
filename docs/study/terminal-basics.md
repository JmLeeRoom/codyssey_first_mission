# 터미널 기초 조작 실습 기록

> 대응 체크리스트: [`CheckList.md`](../../CheckList.md) 섹션 2 "터미널 기초 조작 로그" / [`README.md`](../../README.md) 4.1
> 실습 위치: `~/Project/codyssey/codyssey_first_mission`

## 실습 로그

### 현재 위치 확인

```bash
$ pwd
/home/jmlee/Project/codyssey/codyssey_first_mission
```

### 목록 확인 (숨김 파일 포함)

```bash
$ ls -la
total 24
drwxrwxr-x 4 jmlee jmlee    87 Aug  3 00:56 .
drwxrwxr-x 4 jmlee jmlee    82 Aug  3 00:29 ..
-rw-rw-r-- 1 jmlee jmlee 15783 Aug  3 00:51 CheckList.md
drwxrwxr-x 3 jmlee jmlee    51 Aug  3 00:50 docs
drwxrwxr-x 8 jmlee jmlee   214 Aug  3 00:51 .git
-rw-rw-r-- 1 jmlee jmlee  5799 Aug  3 00:56 README.md

$ ls -al   # -la 와 옵션 순서만 다르며 동일하게 동작함을 확인
total 24
drwxrwxr-x 4 jmlee jmlee    87 Aug  3 00:56 .
drwxrwxr-x 4 jmlee jmlee    82 Aug  3 00:29 ..
-rw-rw-r-- 1 jmlee jmlee 15783 Aug  3 00:51 CheckList.md
drwxrwxr-x 3 jmlee jmlee    51 Aug  3 00:50 docs
drwxrwxr-x 8 jmlee jmlee   214 Aug  3 00:51 .git
-rw-rw-r-- 1 jmlee jmlee  5799 Aug  3 00:56 README.md
```

### 디렉토리 생성

```bash
$ mkdir docs/result
```

### 파일 복사 (+ 삭제 후 재복사로 동작 확인)

```bash
$ cp docs/study/image.png docs/result/
$ rm docs/result/image.png
$ ls docs/result/
              # 삭제되어 목록이 비어있음을 확인

$ cp docs/study/image.png docs/result/
$ ls docs/result/
image.png     # 재복사 후 목록에 다시 나타남을 확인
```

### 파일 이동

```bash
$ mv docs/result/image.png ./
$ ls
CheckList.md  docs  image.png  README.md
```

### 빈 파일 생성 및 내용 확인

```bash
$ touch test.txt
$ cat test.txt
```

## 스크린샷 증거

![터미널 기초 조작 실습](../result/image-1.png)

## 디렉토리 이동 (`cd`)

```bash
$ cd sub_dir && pwd
/home/jmlee/Project/codyssey/codyssey_first_mission/practice/cli-demo/sub_dir

$ cd .. && pwd
/home/jmlee/Project/codyssey/codyssey_first_mission/practice/cli-demo
```

`mv`가 "파일의 위치"를 옮기는 것과 달리, `cd`는 "내가 서 있는 현재 작업 디렉토리 자체"를 바꾼다는 점이 다르다. 이동 전/후 `pwd` 출력이 실제로 바뀐 것으로 확인된다.

![터미널 기초 조작 실습](../result/image-2.png)

## 관찰 메모

- `cp`로 복사 → `rm`으로 삭제 → 다시 `cp`로 복사하는 과정에서, `ls docs/result/`의 출력이 "없음 → `image.png`"로 바뀌는 것을 통해 두 명령의 효과를 직접 대조할 수 있었다.
- `touch`로 만든 빈 파일에 내용을 채운 뒤 `cat`으로 확인했을 때, 파일 끝에 개행 문자가 없으면 출력이 다음 프롬프트와 한 줄로 이어붙는 것을 확인했다 (스크린샷의 `테스트 문서입니다.jmlee@swhs-lab:...` 부분).
