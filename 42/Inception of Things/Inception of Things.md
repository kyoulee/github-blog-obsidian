---
title: Inception of Things
description : kyoulee blog
preview: https://kyoulee.com/post/templates/images/default-og-image.jpg
created: 2026-03-08T03:03:23+09:00
updated: 2026-03-08T03:03:23+09:00
tags:
  - Inception-of-Things
status: public
id : 0
writer : kyoulee
---

다음은 인셉션 과제를 하기위한 과정을 정리하여 나열하여본다

## Part 1

다음은 k3s and Vagrant 로 가상환경을 local에서 여러개를 제어할 수 있는 과정을 제공하는 것을 목표로 한다.

- 2개의 가상 머신을 구동한다
- 가상 머신은 1 cpu 와 512MB 크기의 RAM 로 정한다
- Vagrant를 이용하여 구동하도록 한다.
- 다음은 구성되어야 하는 네트워크 인터페이스이다.
    - first : 192.168.56.110 server , controller mode로 만들어 져야한다.
    - second : 192.168.56.111  server worker, agent mode 로 만들어 져야한다.
- ssh로 비밀번호가 없이 접근이 가능하도록 해야한다

과제에서 제공하는 [[42/Inception of Things/Vagrant]]에 대한 셈플 예제이다

```vg
Vagrant.configure(2) do |config|
    [...]
    config.vm.box = REDACTED
    config.vm.box_url = REDACTED
    
    config.vm.define "wilS" do |control|
        control.vm.hostname = "wilS"
        control.vm.network REDACTED, ip: "192.168.56.110"
        control.vm.provider REDACTED do |v|
            v.customize ["modifyvm", :id, "--name", "wilS"]
            [...]
        end
        config.vm.provision :shell, :inline => SHELL
            [...]
        SHELL
            control.vm.provision "shell", path: REDACTED
    end
    config.vm.define "wilSW" do |control|
        control.vm.hostname = "wilSW"
        control.vm.network REDACTED, ip: "192.168.56.111"
        control.vm.provider REDACTED do |v|
            v.customize ["modifyvm", :id, "--name", "wilSW"]
            [...]
        end
        config.vm.provision "shell", inline: <<-SHELL
            [..]
        SHELL
        control.vm.provision "shell", path: REDACTED
    end
end
```

다음 설정은 [[42/Inception of Things/box|box]] 와 [[42/Inception of Things/config|config]]를 참조하기 바란다.

### 설정 확인

다음 ssh 로 접속하여 실제 구성되어있는 내부 네트워크를 확인해 볼 차례이다.

```sh
vagrant up

vagrant ssh wilS
```

접속 후 네트워크 연결 상태를 보면 

```sh
k get nodes -o wide
```

2개의 네트워크 192.168.56.110 과 192.168.56.111이 있는것을 확인할 수있을 것이다.


## Part 2 

K3s and three simple applications  다음 말과 같이 3개의 간단한 어플리케이션을 구동하는것으로 k3s 구성을 하는것을 목표로한다.
 
## Part 3 

K3d and Argo CD 다음은 실제 배포된 서버와 repository 사이에 변화된 업대이트를 감지하여 변화된 사항을 모니터링하여 업데이트하는 것을 목표로 한다.


상위 3개의 파트를 기준으로 설계를 하고 만들어 보는것이 이번 과제의 목표이다

제출 목록으로는 

```sh
$> find -maxdepth 2 -ls
# 424242 4 drwxr-xr-x 6 wandre wil42 4096 sept. 17 23:42 .
# 424242 4 drwxr-xr-x 3 wandre wil42 4096 sept. 17 23:42 ./p1
# 424242 4 -rw-r--r-- 1 wandre wil42 XXXX sept. 17 23:42 ./p1/Vagrantfile
# 424242 4 drwxr-xr-x 2 wandre wil42 4096 sept. 17 23:42 ./p1/scripts
# 424242 4 drwxr-xr-x 2 wandre wil42 4096 sept. 17 23:42 ./p1/confs
# 424242 4 drwxr-xr-x 3 wandre wil42 4096 sept. 17 23:42 ./p2
# 424242 4 -rw-r--r-- 1 wandre wil42 XXXX sept. 17 23:42 ./p2/Vagrantfile
# 424242 4 drwxr-xr-x 2 wandre wil42 4096 sept. 17 23:42 ./p2/script
# 424242 4 drwxr-xr-x 2 wandre wil42 4096 sept. 17 23:42 ./p1/confs
# 424242 4 drwxr-xr-x 3 wandre wil42 4096 sept. 17 23:42 ./p3
# 424242 4 drwxr-xr-x 2 wandre wil42 4096 sept. 17 23:42 ./p3/scripts
# 424242 4 drwxr-xr-x 2 wandre wil42 4096 sept. 17 23:42 ./p3/confs
# 424242 4 drwxr-xr-x 3 wandre wil42 4096 sept. 17 23:42 ./bonus
# 424242 4 -rw-r--r-- 1 wandre wil42 XXXX sept. 17 23:42./bonus/Vagrantfile
# 424242 4 drwxr-xr-x 2 wandre wil42 4096 sept. 17 23:42 ./bonus/scripts
# 424242 4 drwxr-xr-x 2 wandre wil42 4096 sept. 17 23:42 ./bonus/confs
```

다음과 같은 파일 목록을 만드는 것으로 분할하여 파트별로 스크립트를 짜는것으로 볼 수 있다.

