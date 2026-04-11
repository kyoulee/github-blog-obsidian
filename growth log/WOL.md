---
title: 편한 WOL 기능
description : kyoulee blog
preview: /post/templates/images/default-og-image.jpg
created: 2026-04-11T06:04:48+09:00
updated: 2026-04-11T06:04:48+09:00
tags:
  - wol
status: public
id : 0
writer : kyoulee
---

이번에 서버를 포맷하면서 sleep mode(suspend)에 들어가버려 서버 컴퓨터가 멈처버린 경험을 하게 되었다. 😓
외부에 있다보니 바로 킬 수도 없고 조금 곤란한 상황이 일어나 wol 설정을 해보려고 한다.

## WOL (Wake On Lan)

> [!Note]
> 네트워크에 연결되있는 컴퓨터를 작동할 수 있도록 하며 **Magic Packet** 이라는 용어로 특정 프로토컬에 종속되지 않은 규약으로 보인다.

간단히  원격으로 네트워크가 연결되있다면 컴퓨터를 네트워크 요청으로 킬 수 있는 방법이다.
WoL은 1990년대 중반 **IBM과 Intel**이 주도한 'Advanced Manageability Alliance'에서 제안한 기술로, 인터넷 표준 기구(IETF)의 RFC가 아닌 업계 표준(De Facto Standard)으로 자리잡았다고 한다. 허나 RFC에 대한 규정은 없지만 amd에서 제공하는 문서가 표준처럼 사용된다고 한다.

결국 매직 패킷은 UDP 통신으로 9번 프로토컬에 102바이트의 데이터를 전달하는 것으로 상대에게 어떠한 작동을 할 지 알려 주는 레서피라고 생각하면 편하다 다음은 실제 데이터 전송 방법의 내용이다.

```md
If the IEEE address for a particular node on the network
was 11h 22h 33h 44h 55h 66h, then the LAN controller
would be scanning for the data sequence (assuming an
Ethernet Frame):
DESTINATION SOURCE MISC FF FF FF FF FF
FF 11 22 33 44 55 66 11 22 33 44 55 66 11 22 33 44
55 66 11 22 33 44 55 66 11 22 33 44 55 66 11 22 33
44 55 66 11 22 33 44 55 66 11 22 33 44 55 66 11 22
33 44 55 66 11 22 33 44 55 66 11 22 33 44 55 66 11
22 33 44 55 66 11 22 33 44 55 66 11 22 33 44 55 66
11 22 33 44 55 66 11 22 33 44 55 66 MISC CRC
```

이를 봤을때 ```FF FF FF FF FF FF``` 를 기점으로 이것은 magic packet 의 시작을 알리고 다음으로 요청하려는 MAC주소를 전달하는 것으로 원하는 컴퓨터의 lancard를 찾아 켜주는 것으로 보인다. 

## 설정

### Bios

실제 lancard에서 WOL를 지원하는지 확인이 필요하다 그리고 BIOS에서 설정을 해주어아야 하는데 이는 필자는 MSI 보드의 고급설정을 확인하여 BIOS를 사용한 WOL가 작동하도록 하였다. 이는 OS와 BIOS 설정이 있는데 시스템이 완전히 꺼졌을 경우 OS로는 킬 수 가 없기 때문에 BIOS 로 설정해주고 최신 랜카드들은 둘다 지원하는 모델들도 있는것으로 보인다.

## Iptime

필자는 iptime을 사용하기 있기때문에 다음과 같이 WOL 설정을 해주었다

![[growth log/images/Pasted image 20260411191538.png]]

> [!Note]
> 왜 mac주소만 전달할까? 이는 컴퓨터가 꺼지면 ip를 자동 할당 되는 경우가 많기 때문에(수동 설정을 하여도 OS가 작동하지 않기때문으로 보인다.) 고정적인 IP로 사용을 할 수 가 없다. 그로인해 boardcast와 같이 전체적으로 9port에 요청하는 것으로 mac주소를 찾고 깨워주는 방식을 택한것으로 보인다.

이로서 사용하고자 하는 컴터를 WOL설정으로 킬 요건이 되었고 외부에서 ```netcat```을 이용하여 켜보도록 한다

```sh
echo -ne $(echo "FFFFFFFFFFFF$(printf '047C16544249%.0s' {1..16})" | sed 's/../\\x&/g') | nc -u -b 192.168.0.255 9
```

이로서 내가 속한 네트워크는 요청이 되는것을 확인하였다.

> [!Warning]
> 하위 부분은 iptime의 부분적 오류 혹은 보안이거나 통신사의 의해 안되는것으로 추정된다.

추가적으로 port forwarding으로 만들어 주려 했지만 실패하였다. 이유는 외부의 ip 에서 접속하여 magic packet을 내가 원하는 lancard에 전달하지 않는것으로 보인다. 이유는 못찾고 추후에 iptime쪽에 문의를 해볼 예정이다.

### 공유기 접속

상위 방법이 안되어 어쩔 수없이 내부망을 접속하기 위한 gateway 서버를 만들던가 아님 router에 접속하여 내부망에서 해결하는 방법밖에 없게 되었다. 
그럼으로 iptime의 공유기를 접속하여 해결하는 쪽으로 선택하였다.

![[growth log/images/Pasted image 20260411192812.png]]

이는 공유기의 내부 접속 방법을 정의한 것으로 (1000이상의 port만 가능한것으로 보인다 필자도 알고 싶지 않았다.) ip를 이용하여 접속할 수 있게 열어두는 것으로 이 문제 또한 해결하여 현재는 외부에서도 편하게 서버 pc를 sleep mode 또는 power off 상태에서도 사용할 수 있게 되었다.

## Reference

wiki : https://en.wikipedia.org/wiki/Wake-on-LAN
amd : https://www.amd.com/content/dam/amd/en/documents/archived-tech-docs/white-papers/20213.pdf