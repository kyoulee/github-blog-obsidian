---
title: config
description: kyoulee blog
preview: https://kyoulee.com/post/templates/images/default-og-image.jpg
created: 2026-03-08T04:03:25+09:00
updated: 2026-03-08T04:03:25+09:00
tags:
  - config
status: public
id: 0
writer: kyoulee
---

다음은 큰 분류의 `config` 목록이다.

|**설정 그룹**|**주요 역할**|**한 줄 요약**|
|---|---|---|
|**`config.vm`**|**가상 머신 자체** 설정|OS 선택, 네트워크, 공유 폴더, 하드웨어 사양 등 가장 많이 건드리는 곳.|
|**`config.ssh`**|**SSH 접속** 설정|Linux 계열 VM에 접속할 때 쓰는 사용자명, 패스워드, 키 경로 등을 설정.|
|**`config.winrm`**|**Windows 원격** 설정|Windows VM을 제어하기 위한 통신 규격(WinRM) 관련 설정.|
|**`config.winssh`**|**Windows SSH** 설정|최신 Windows에서 WinRM 대신 SSH를 사용해 접속할 때 쓰는 설정.|
|**`config.vagrant`**|**Vagrant 자체** 설정|Vagrant 프로그램의 동작(플러그인 확인, 호스트 환경 체크 등) 설정.|

필자는 window를 별로 안좋아하기 때문에 window에 관한건 스킵하도록 하겠다.

## config.vm
### Box

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`box`**|String|사용할 박스 이름 (예: `"ubuntu/focal64"`)|
|**`box_version`**|String|박스 버전 지정 (예: `">= 1.0, < 2.0"`)|
|**`box_url`**|String/Array|박스가 없을 경우 다운로드할 주소 (직접 링크나 파일 경로 가능)|
|**`box_check_update`**|Boolean|`up` 시마다 업데이트 확인 여부 (기본값: `true`)|
|**`box_architecture`**|String|아키텍처 지정 (`amd64`, `arm64` 등 / 기본값: `:auto`)|
|**`ignore_box_vagrantfile`**|Boolean|**박스 내부에 저장된 Vagrantfile 설정을 무시**할지 여부|

### Hardware

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`guest`**|Symbol|게스트 OS 종류 지정 (`:linux`, `:windows` 등 / 자동 감지 가능)|
|**`hostname`**|String|VM의 호스트 이름 설정. 설정 시 `/etc/hosts`도 자동 갱신됨.|
|**`boot_timeout`**|Integer|부팅 대기 제한 시간(초) (기본값: 300초)|
|**`graceful_halt_timeout`**|Integer|종료(`halt`) 시 정상 종료를 기다려주는 시간 (기본값: 60초)|
|**`communicator`**|String|연결 방식 (`"ssh"` 또는 Windows용 `"winrm"`)|
|**`disk`** / **`cloud_init`**|Block|추가 디스크 구성이나 `cloud-init` 자동화 설정|

## Network

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`network`**|Block|포트 포워딩, 고정 IP, 브리지 네트워크 설정|
|**`synced_folder`**|Block|호스트와 VM 간의 폴더 공유 설정|
|**`usable_port_range`**|Range|포트 충돌 시 Vagrant가 사용할 포트 범위 (기본: `2200..2250`)|
|**`allow_hosts_modification`**|Boolean|Vagrant가 호스트의 `/etc/hosts` 파일을 건드려도 되는지 여부|
|**`allow_fstab_modification`**|Boolean|공유 폴더를 위해 `/etc/fstab`에 등록할지 여부|

## Provider

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`provision`**|Block|쉘 스크립트, 앤서블 등을 이용한 **자동 소프트웨어 설치** 설정|
|**`provider`**|Block|**VirtualBox, VMware** 등 특정 플랫폼 전용 설정 (메모리, CPU 등)|
|**`post_up_message`**|String|머신이 다 뜨고 나서 사용자에게 보여줄 안내 메시지|

## config.ssh

### connect

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`host`**|String|접속할 IP/호스트명 (보통 프로바이더가 자동 설정)|
|**`port`**|Integer|호스트 측 SSH 포트 (기본값: 22)|
|**`connect_timeout`**|Integer|연결 시도 대기 시간 (기본값: 15초)|
|**`connect_retries`**|Integer|연결 실패 시 재시도 횟수 (기본값: 5회)|
|**`keep_alive`**|Boolean|연결 유지를 위해 5초마다 패킷 전송 여부 (`true`)|

## Authentication

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`username`**|String|접속 계정 (기본값: `"vagrant"`)|
|**`password`**|String|접속 비밀번호 (키 인증을 더 권장함)|
|**`private_key_path`**|String/Array|사용할 개인 키(`.pem` 등) 경로|
|**`insert_key`**|Boolean|보안을 위해 기본 공용 키를 새 키로 자동 교체할지 여부 (`true`)|
|**`keys_only`**|Boolean|Vagrant가 제공한 키만 사용하고 `ssh-agent` 키는 무시할지 여부|
|**`verify_host_key`**|Symbol|호스트 키 검증 수준 (`:never`, `:always` 등)|

## Enviroment

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`shell`**|String|실행할 기본 쉘 (기본값: `bash -l`)|
|**`sudo_command`**|String|`sudo` 실행 시 사용할 템플릿 (기본값: `sudo -E -H %c`)|
|**`forward_agent`**|Boolean|호스트의 SSH Agent를 머신 안에서도 쓸지 여부 (Github 연동 시 유용)|
|**`forward_x11`**|Boolean|가상 머신의 GUI 프로그램 화면을 호스트로 보낼지 여부|
|**`pty`**|Boolean|터미널(Pseudo-TTY) 사용 여부 (**주의: 꼭 필요할 때만 사용**)|

## Advance

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`extra_args`**|Array|SSH 실행 시 직접 전달할 추가 인자 (예: `["-L", "80:80"]`)|
|**`proxy_command`**|String|프록시를 거쳐 접속해야 할 때 사용하는 명령어|
|**`forward_env`**|Array|호스트의 환경 변수를 머신으로 넘겨줄 리스트|

## config.vagrant

|**설정 항목**|**타입**|**설명**|
|---|---|---|
|**`host`**|String/Symbol|내 컴퓨터(Host)의 OS 종류. 보통 `:detect`로 자동 감지하지만, NFS 설정 등이 꼬일 때만 수동으로 지정 (`:windows`, `:linux` 등).|
|**`plugins`**|String/Array/Hash|**가장 핵심.** 프로젝트 실행에 필요한 **플러그인을 정의**함. 설치되어 있지 않으면 Vagrant가 자동으로 설치를 시도함.|
|**`sensitive`**|String/Array|터미널 로그나 출력 화면에서 **가려야 할 비밀 정보** 지정. 비밀번호나 API 토큰 등이 노출되는 것을 방지.|

예시 코드

```
# 1. 단일 플러그인
config.vagrant.plugins = "vagrant-disksize"

# 2. 여러 개일 때
config.vagrant.plugins = ["vagrant-disksize", "vagrant-vbguest"]

# 3. 특정 버전이 필요할 때 (Hash 방식)
config.vagrant.plugins = {
  "vagrant-scp" => { "version" => "1.0.0" },
  "vagrant-vbguest" => { "version" => ">= 0.20.0" }
}
```



## Reference 

🔗 Link : [vagrant docs](https://developer.hashicorp.com/vagrant/docs)
