---
title: box
description : kyoulee blog
preview: /post/templates/images/default-og-image.jpg
created: 2026-03-08T04:03:18+09:00
updated: 2026-03-08T04:03:18+09:00
tags:
  - box
  - vagrant
status: public
id : 0
writer : kyoulee
---

box는 vagrant 환경설정과 운영체제를 구성하는것을 말한다. 
필자의 느낌으로는 `docker image`와 비슷한것으로 보인다.

![[42/Inception of Things/images/Pasted image 20260308163509.png]]

다음 사이트 [discover](https://portal.cloud.hashicorp.com/vagrant/discover)를 이용하여 독자가 필요한 box를 찾아 사용할 수 있다.
 
```sh
vagrant init hashicorp/bionic64
```

다음 명령어로 구성에 대하여 볼수 있으며

```sh
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
end
```
다음과 같은 형식으로 사용하며


|**코드 구성 요소**|**의미 (뜻)**|**실제 역할**|
|---|---|---|
|**`Vagrant.configure("2")`**|**버전 선언**|Vagrant의 **API 버전 2**를 사용하겠다는 선언. (호환성 유지용)|
|**`do|config|... end`**|
|**`config`**|**설정 변수**|하위 옵션들을 담는 **장바구니(변수)** 이름. 이후 모든 설정은 `config.`으로 시작.|
|**`vm.box`**|**대상 옵션**|가상 머신(`vm`)의 템플릿 이미지(`box`)를 지정하는 속성.|
|**`"hashicorp/bionic64"`**|**설정값 (Box 이름)**|**Ubuntu 18.04 LTS** 버전의 공식 이미지 이름. (도커 이미지명과 유사)|

추가로 설정을 하는방법은 다음과 같다

```sh
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  
  # 메모리를 2GB (추가 주문)
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
  end
end
```

[[42/Inception of Things/config|config]] 에 대하여는 다음을 확인하여 자세한 설정방법을 알아보기 바란다.

## Reference 

🔗 Link : [discover](https://portal.cloud.hashicorp.com/vagrant/discover)