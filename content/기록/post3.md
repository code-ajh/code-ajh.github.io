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
만약 서버 작업에 익숙하신 분들은 init 0 을 통해 서버를 종료해본 기억이 있을수도 있겠네요. 
해당 명령어는 0 런레벨로 동작하겠다는 뜻입니다. 

아래는 RHEL 6 기준의 각 런레벨 설명입니다.

| level | 설명                      | 유저 인터페이스 | 특징              |
| ----- | ----------------------- | -------- | --------------- |
| 0     | 시스템 정지                  |          |                 |
| 1     | 단독 사용자 모드               | CLI      | root 만 사용 가능    |
| 2     | 사용자 정의                  | CLI      |                 |
| 3     | 다중 사용자                  | CLI      | 일반적인 서버 사용 레벨   |
| 4     | 사용자 정의                  |          |                 |
| 5     | 다중 사용자 모드 (X Window 기반) | GUI      | 일반적인  GUI 사용 레벨 |
| 6     | 리부트                     |          |                 |

**다만 런레벨은 RHEL 7 에서부터 사용되지 않으며 호환성을 가지고 systemd 의 target 으로 변경 되었습니다.**

---
### rc.local

분명 rc.local에서 오류가 발생했다 그랬는데 위에서는 런레벨만 말하고 있는 이유는 rc.local은 런레벨의 동작과 관계가 있었기 때문입니다. 

리눅스에서는 /etc/rc[0~6].d 라고 하는 실행 수준에 맞는 각 디렉토리의 스크립트들을 실행하였으며 모두 끝난 뒤에는 rc.local 을 모든 런레벨에서 공통적으로 실행합니다.

|     RHEL 6      |     RHEL 9      |
| :-------------: | :-------------: |
| ![[RHEL 6.png]] | ![[RHEL 9.png]] |

다만 위에서 말한 것처럼 런레벨을 더 이상 사용되지 않으며 RHEL 9 에서 호환성을 제외한 나머지 부분은 제거되었습니다.  rc.local은 호환을 위해서 유지되고 있는 상태입니다.  

기존에는 rc.local이 init 에서 실행되었지만 systemd target 으로 변경된 이후에는 rc-local.service 를 통해서 실행됩니다. 

system file은 이렇게 구성되어 있습니다.
```bash
#  SPDX-License-Identifier: LGPL-2.1-or-later
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

# This unit gets pulled automatically into multi-user.target by
# systemd-rc-local-generator if /etc/rc.d/rc.local is executable.
[Unit]
Description=/etc/rc.d/rc.local Compatibility
Documentation=man:systemd-rc-local-generator(8)
ConditionFileIsExecutable=/etc/rc.d/rc.local
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
ExecStart=/etc/rc.d/rc.local start
TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no
```
systemd 파일에 대한 설명은 따로 진행하지 않습니다.

해당 서비스는 `rc.local start` 명령어를 통해서 실행되므로 `rc.local` 파일에는 실행 권한이 필요합니다. 추가로 당연하지만 스크립트 실행 중 실패 발생시 오류가 나면서 실행이 되지 않으므로 그 부분만 고려하며 사용하면 됩니다.

---
### 출처

> SysV init 런레벨 - Red Hat Documentation 
> https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/6/html/installation_guide/s1-boot-init-shutdown-sysv

> systemd 대상 작업 - Red Hat Documentation
> https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-Managing_Services_with_systemd-Targets#tabl-Managing_Services_with_systemd-Targets-Runlevels

