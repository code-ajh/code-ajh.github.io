---
title: Home
description: 
aliases: 
tags: 
draft: 
date: 2024-08-03
---
### ABOUT

어쩌다 보니 개발로 밥 벌어 먹고 있는 컴퓨터 공학 전공
그런데 어느 정도 적성에 맞아서 계속하고 있습니다.

평범한 중소기업에서 이것저것 하고 있지만...
팀 중간에서 윤활제 역할로 살아가고 있는 평범한 잡부입니다.

```dataview
TABLE
file.ctime as "Created Time", file.mtime as "Updated Time", file.tag as "Tag"
FROM "/"
WHERE file.folder != "이미지 저장" AND file.folder != "관리/템플릿" AND file.folder != "dashboard"
SORT file.mtime DESC
LIMIT 10
```
### 카테고리

- [[잡담/]]
- [[기록/]]
- [[오류/]]
