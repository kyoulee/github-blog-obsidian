---
title: Redhat 자격증
description : kyoulee blog
preview: /post/templates/images/default-og-image.jpg
created: 2026-02-03T08:02:36+09:00
updated: 2026-02-03T08:02:36+09:00
tags:
  - Redhat
  - RHCSA
status: public
id : 0
writer : kyoulee
---

# [자격증] RHCSA 합격 가이드: RHEL 9 실습 환경 구축부터 시험 꿀팁까지 🐧

안녕하세요! 오늘은 리눅스 엔지니어를 꿈꾸는 분들이라면 한 번쯤 들어보셨을 **RHCSA(Red Hat Certified System Administrator, EX200)** 자격증 취득 과정을 정리해 보려고 합니다.

이 시험은 이론만 빠삭하다고 딸 수 있는 게 아니라, 직접 터미널을 두드려 문제를 해결해야 하는 **100% 실기 시험**이라 준비 과정이 정말 중요해요. 제가 직접 공부하며 정리한 핵심 흐름을 공유합니다!

---

## RHCSA
레드햇(Red Hat)에서 주관하는 이 시험은 업계에서 가장 공신력 있는 리눅스 자격증 중 하나예요. 
* **실무 중심:** 실제 운영체제 위에서 트러블슈팅을 해야 합니다.
* **업계 표준:** 기업에서 가장 많이 사용하는 RHEL(Red Hat Enterprise Linux) 기반입니다.
* **합격 기준:** 300점 만점에 **210점 이상**이면 합격!

---

## 실습 환경 만들기 (RHEL 9 무료 설치)
유료 OS인 레드햇을 돈 내고 연습하기엔 부담스럽죠? 개발자 계정을 이용하면 무료로 실습할 수 있습니다.

### 준비물
1. **VMware Workstation Player** (무료 가상화 툴)
2. **Red Hat Developer 계정** (가입 필수)

### 🛠️ 세팅 순서
1. **계정 생성:** [Red Hat Developers](https://developers.redhat.com/) 사이트에서 가입합니다.
2. **ISO 다운로드:** 로그인 후 `RHEL 9 DVD ISO` 파일을 받습니다.
3. **VM 생성:** * 메모리는 **2GB~4GB**, 디스크는 **20GB** 정도면 충분합니다.
   * 설치 시 'Software Selection'에서 **Server with GUI**를 선택해 주세요.
4. **서브스크립션 등록:** 설치 완료 후 터미널에서 아래 명령어를 쳐야 패키지 설치(dnf)가 가능해집니다.
```bash
   sudo subscription-manager register --username [내계정ID] --password [내비밀번호]
```




