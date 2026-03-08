---
title: Vagrant
description: kyoulee blog
preview: https://kyoulee.com/post/templates/images/default-og-image.jpg
created: 2026-03-08T04:03:59+09:00
updated: 2026-03-08T04:03:59+09:00
tags:
  - Vagrant
  - virtual
status: public
id: 0
writer: kyoulee
---

## Vagrantfile

Vagrantfile은 머신의 유형을 설명하고 해당머신을 구성하는 것을 목표로하는 파일이다. 
스크립트의 유형은 Ruby와 같으며 간단한 부분 수정을 구성하고 있으므로 Ruby에 대한큰 지식을 필요로하고 있지않다.

docker 와 비슷한 [[42/Inception of Things/box|box]] 가 있다. 그리고 그것을 구성하는 [[42/Inception of Things/config|config]]로 다양한 환경설정을 기록할 수 있다.

## Lookup Path

파일의 조회 경로는 다음과 같다. 그럼으로 프로젝트의 범위를 넘어가는 자동 할당을 원하지 않는다면 Lookup Path 목록을 주의하여 생성하도록 하자

> [!TIP]
> `VAGRANT_CWD` 를 이용하여 lookup 폴더를 지정할 수도 있다.


```
/home/mitchellh/projects/foo/Vagrantfile
/home/mitchellh/projects/Vagrantfile
/home/mitchellh/Vagrantfile
/home/Vagrantfile
/Vagrantfile
```

상위와 같은 위치에 구성되있는 Vagrantfile이 있을경우 vagrant 명령어로 실행을 할 수 있다.

## Load Order and Merging

> [!Warning]
> 다음과 같은 사항의 순서로 Vagrantfile의 내용이 추가되어 단계별 수정될 수 있음으로 전체적인 영향을 가질 기본 설정 파일을 만들경우 유희하여 만들도록 한다.

|**순서**|**단계 (Scope)**|**실제 예시 경로 (Linux/Mac 기준)**|**비고**|
|---|---|---|---|
|**1**|**Box 내장**|`~/.vagrant.d/boxes/[박스이름]/[버전]/[프로바이더]/Vagrantfile`|박스 제작자가 넣어둔 기본값 (수정 비권장)|
|**2**|**사용자 홈**|`~/.vagrant.d/Vagrantfile`|내 PC의 모든 프로젝트에 공통 적용할 때 사용|
|**3**|**프로젝트**|`/home/user/projects/my_app/Vagrantfile`|**가장 많이 쓰는 곳.** 현재 폴더(또는 상위 폴더)의 파일|
|**4**|**멀티 머신**|(프로젝트 Vagrantfile 내부의 `config.vm.define` 블록)|특정 VM 이름에만 주는 개별 설정|
|**5**|**프로바이더**|(프로젝트 Vagrantfile 내부의 `config.vm.provider` 블록)|VirtualBox, AWS 등 전용 설정|



## Reference

🔗 링크 : [Vagrant](https://developer.hashicorp.com/vagrant)
