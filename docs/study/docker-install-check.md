# Docker 설치 및 기본 점검 기록

> 대응 체크리스트: [`CheckList.md`](../../CheckList.md) 섹션 4 "Docker 설치 및 기본 점검" / [`README.md`](../../README.md) 4.3
> 환경: 네이티브 Linux에 Docker Engine이 설치되어 있어 OrbStack은 사용하지 않음 (sudo 제약 없는 환경)

## 버전 확인

```bash
$ docker --version
Docker version 29.6.2, build dfc4efb
```

## 데몬 동작 확인

```bash
$ docker info
Client: Docker Engine - Community
 Version:    29.6.2
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.36.0
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v5.3.1
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 1
  Running: 0
  Paused: 0
  Stopped: 1
 Images: 5
 Server Version: 29.6.2
 Storage Driver: overlayfs
  driver-type: io.containerd.snapshotter.v1
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 CDI spec directories:
  /etc/cdi
  /var/run/cdi
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 11ce9d5f3c68c941867e82890e93e815c1304f1b
 runc version: v1.3.6-0-g491b69ba
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.8.0-136-generic
 Operating System: Ubuntu 24.04.4 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 72
 Total Memory: 345.8GiB
 Name: swhs-lab
 ID: 72179aac-494c-4a82-b3c8-b9b4aea71ece
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  ::1/128
  127.0.0.0/8
 Live Restore Enabled: false
 Firewall Backend: iptables
  EnableUserlandProxy: true
  UserlandProxyPath: /usr/bin/docker-proxy
```

## 스크린샷 증거

![Docker 설치/점검 로그](../result/image-4.png)

## 관찰 메모

- `docker info`가 `Server:` 섹션까지 정상 출력됐다는 것 자체가 **데몬이 살아있고 클라이언트와 통신 가능하다**는 증거다 (데몬이 꺼져있으면 이 명령에서 연결 오류가 난다).
- 현재 이미지 5개, 컨테이너 1개(중지 상태)가 이미 존재 — 이후 이미지/컨테이너 운영 명령(섹션 5) 로그를 남길 때 이 숫자가 어떻게 변하는지 비교하면 좋다.
- `Cgroup Driver: systemd`, `Storage Driver: overlayfs`처럼 이후 트러블슈팅에서 원인 분석에 참고할 수 있는 정보도 함께 기록해뒀다.
