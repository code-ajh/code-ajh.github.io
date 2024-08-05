---
title: 런레벨과 rc.local
description: rc.local 관련 오류가 생겨 관련 개념 정리
aliases: 
tags:
  - Linux
  - systemd
draft: false
date: 2024-08-03
---

최근 Rocky Linux 9 환경에서 작업을 하면서 rc.local 오류를 접하면서 관련 정보를 찾아 볼 일이 있었습니다. ~~단순 문법 문제였지만~~ rc.local 및 run level 관련하여 알게된 기본적인 개념을 기록해둡니다.

  > [!note]
> 제가 RHEL 기반의 리눅스(CentOS, RockyLinux)에 익숙하기 때문에 해당 계열의 리눅스를 기준으로 설명하며 이름은 RHEL 로 통일합니다.


---

### 런레벨(run level)

이전 버전 (RHEL 6 이하)의 리눅스에서는 런레벨이라고 하는 개념으로 부팅할 때의 동작을 제어 했습니다. 

런레벨은 해당 서버를 어떠한 방식으로 사용할지 정할 수 있습니다. ( 예시 : CLI -> GUI )
만약 서버 작업에 익숙하신 분들은 init 0 을 통해 서버를 종료하거나 init 6 등으로 재부팅 해본 적 있을수도 있습니다. 해당 명령어는 해당 런레벨을 시작한다는 뜻입니다. 

아래는 RHEL 6 기준의 각 런레벨의 뜻입니다. 

| level | 설명                      | 유저 인터페이스 | 특징              |
| ----- | ----------------------- | -------- | --------------- |
| 0     | 시스템 정지                  |          |                 |
| 1     | 단독 사용자 모드               | CLI      | root 만 사용 가능    |
| 2     | 사용자 정의                  | CLI      |                 |
| 3     | 다중 사용자                  | CLI      | 일반적인 서버 사용 레벨   |
| 4     | 사용자 정의                  |          |                 |
| 5     | 다중 사용자 모드 (X Window 기반) | GUI      | 일반적인  GUI 사용 레벨 |
| 6     | 리부트                     |          |                 |

### rc.local

분명 rc.local에서 오류가 발생했다 그랬는데 위에서는 런레벨만 말하고 있습니다.

왜냐하면 rc.local은 런레벨의 동작과 관계가 있기 때문입니다. 

리눅스에서는 /etc/rc[0~6].d 라고 하는 디렉토리가 있는데 해당 런레벨을 실행 할 경우 맞는 동작 스크립트를 모아둔 디렉토리입니다. 

런레벨 0을 실행하면 rc0.d 디렉토리의 스크립트들을 실행하는 방식이죠. 

---
### 출처

> SysV init 런레벨 - Red Hat Documentation 
> https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/6/html/installation_guide/s1-boot-init-shutdown-sysv

