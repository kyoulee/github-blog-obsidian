---
title: Inception of Things
description : kyoulee blog
preview: /post/templates/images/default-og-image.jpg
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

- k3s를 설치하고 
- host에 따라 app을 접속할 수 있도록 하여야한다.
- 이는 192.168.0.56.110을 접속한다음 app1.com으로 접속하는것과 app2.com으로 접속 그리고 app3.com은 주소가 없을경우 접속 되도록 한다.


### host와 ip의 차이

```sh
curl -H “Host:app1.com” 192.168.56.110
```

`Host`는 이곳으로 요청을 한다고 표시한것으로 ip의 경우는 서버를 찾기위한 주소고 Host는 도착한다음 어떠한 자료를 원해서 온건지 검증하는 느낌이다.

[[Nginx]]와 같은 원리인것으로 보인다 port를 지정하는게 아니라 host를 지정함으로서 어느 app에 요청을 할건지 같은 느낌이다.

### Develop

그럼 요구사항을 분석해 만들어보자

| Host | Server | info |
| --- | --- | --- |  
| app1.com | 1 개의 서버 | app1.com으로 요청시 반환 |
| app2.com | 1 개의 서버 | app2.com으로 요청시 반환 |
| app3.com | 3 개의 서버 | 상위 2개 이외의 모든 요청 반환 |

요청된 사이트 모습은 kubernets 기본 사이트로 `Hello form appX.` 형식이 반환되도록 한다.

> [!Note]
> 다음 사이트를 확인하여 app을 만들것 [hello-kubernetes](https://github.com/paulbouwer/hello-kubernetes)


```yaml
# --- APP 1 (Replica: 1) ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: hello-kubernetes
        image: paulbouwer/hello-kubernetes:1.10
        ports:
        - containerPort: 8080
        env:
        - name: MESSAGE # 화면에 표시될 메시지
          value: "Hello from app1."
---
apiVersion: v1
kind: Service
metadata:
  name: app1-svc
spec:
  selector:
    app: app1
  ports:
    - protocol: TCP
      port: 80 # 외부에서 보이는 포트
      targetPort: 8080 # 컨테이너 내부 포트
---
# --- APP 2 (Replica: 1) ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: hello-kubernetes
        image: paulbouwer/hello-kubernetes:1.10
        ports:
        - containerPort: 8080
        env:
        - name: MESSAGE
          value: "Hello from app2."
---
apiVersion: v1
kind: Service
metadata:
  name: app2-svc
spec:
  selector:
    app: app2
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
# --- APP 3 (Replica: 3, Catch-all) ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app3
spec:
  replicas: 3 # <--- 3개의 서버!
  selector:
    matchLabels:
      app: app3
  template:
    metadata:
      labels:
        app: app3
    spec:
      containers:
      - name: hello-kubernetes
        image: paulbouwer/hello-kubernetes:1.10
        ports:
        - containerPort: 8080
        env:
        - name: MESSAGE
          value: "Hello from app3."
---
apiVersion: v1
kind: Service
metadata:
  name: app3-svc
spec:
  selector:
    app: app3
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
# --- Ingress (라우팅 규칙) ---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: main-ingress
spec:
  rules:
  - host: "app1.com" # app1.com -> app1-svc
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
  - host: "app2.com" # app2.com -> app2-svc
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-svc
            port:
              number: 80
  - http: # Host 없음 -> 모든 나머지 요청 -> app3-svc
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app3-svc
            port:
              number: 80
```



## Part 3 

[[42/Inception of Things/K3d|K3d]] and [[42/Inception of Things/Argo CD|Argo CD]] 다음은 실제 배포된 서버와 repository 사이에 변화된 업대이트를 감지하여 변화된 사항을 옵저브하여 업데이트하는 것을 목표로 한다.

상위 3개의 파트를 기준으로 설계를 하고 만들어 보는것이 이번 과제의 목표이다


2개의 namespaces 를 가지도록 한다

-  dedicated to Argo CD ( Argo CD 를 위한?? 거? )
- dev라는 이름을 가지는 어플이며 이는 Github repositiory를 참조하여 Argo CD 가 자동 배포를 하는것을 목적으로 한다.
- 
```sh
$> kubectl create namespace argocd
$> kubectl create namespace dev

$> kubectl get ns
NAME              STATUS   AGE
argocd            Active   5m    # <--- 우리가 만든 관리용 방
dev               Active   2m    # <--- 우리가 만든 서비스용 방
```
그리고  2개의 다른 버전을 배포하여야한다. (아마 dev라는 에 말하는거 같다.)

선택이 가능한 
- [https://hub.docker.com/r/wil42/playground](https://hub.docker.com/r/wil42/playground) 이것을 참조하여 이것으로 만들기
- 집접 docker hub에 배포 컨테이너를 만들어 해보기


이제 배포가 되면 다음과 같은 결과가 나올것이다

```sh

$> k get pods -n argocd
NAME                                           READY   STATUS
argocd-application-controller-0                1/1     Running
argocd-redis-74cb89f466-9vlnp                  1/1     Running
argocd-repo-server-6674446b58-shv8p            1/1     Running
argocd-server-58fb564889-l9p6s                 1/1     Running
...

$> k get pods -n dev
NAME                              READY   STATUS    RESTARTS   AGE
wil-playground-65f745fdf4-d2l2r   1/1     Running   0          8m9s

```


## result

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

## Reference

- hello-kubernetes : [https://github.com/paulbouwer/hello-kubernetes](https://github.com/paulbouwer/hello-kubernetes)

