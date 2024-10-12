---
title: MongoDB의 time series collection
description: MongoDB 5.x 에 정식으로 추가된 time series collection 조사
aliases: 
tags:
  - MongoDB
  - OLAP
draft: true
date: 2024-09-08
---
OLAP 관련하여 조사하다가 MongoDB의 새로운 기능 time series collection에 대해서 알게 되었습니다. 해당 기능을 사용할 경우에는 기존 collection 과 어떤 차이가 있는지 확인해보고 간단하게 쿼리를 테스트 해봤습니다.

---
# 개념

Time Series Collections은 효과적으로 시계열 데이터를 저장하기 위해  MongoDB 5.0 에서 도입된 새로운 Collection 입니다. 

**기본적인 동작 원리는 비슷한 시간대의 데이터를 묶어 Paritioning 하는 것이며 추가로 Metafield 를 사용하여 추가적인 bucket 분리가 이루어집니다.** 시간 순서대로 데이터가 들어와도 저장은 반드시 시간 순으로 이루어집니다. 

![[Pasted image 20240908145110.png]]


아래는 time series collection 생성 쿼리 예시입니다.
``` json
db.createCollection( 
	"collection_name", {
		timeseries: {
			timeField: "timestamp",
			metaField: "metadata",
			granularity: "seconds",
			bucketMaxSpanSeconds : 60,
			bucketRoundingSeconds : 60
		},
		expireAfterSeconds: value_in_seconds 
	} 
)
```

`timeseries` 키워드로 time series collection 으로 만든다는 것을 명시합니다.

나머지 필드의 뜻은 아래와 같습니다.  

```
`timeField` : 시간 데이터로 사용할 필드 이름
`metaField` : bucket 분리에 사용될 필드 이름 
`granularity` : 시간 데이터를 어떤 단위로 분리할 것인지 \[ seconds | minutes | hours ] 

`bucketMaxSpanSeconds` : 최대 시간 범위 
`bucketRoundingSeconds` : 새롭게 생성되는 bucket의 timestamp 의 반올림 범위
	- `bucketMaxSpanSeconds`, `bucketMaxSpanSeconds` 는 동일한 값을 가지고 있어야 하며 `granularity` 와 동시에 사용할 수 없음
`expireAfterSeconds` : TTL, 일반 collection 에서도 사용 가능한 기능과 비슷함
```


아래의 명령어를 통해서 Timeseries Collection 을  생성해봅시다.

```mongodb
db.createCollection(
	"ts_collection", {
		timeseries: {
			timeField: "timestamp",
			metaField: "metadata",
			bucketMaxSpanSeconds: 86400,
			bucketRoundingSeconds: 86400 
		} 
	} 
)
```

생성에 성공하면 아래와 같은 collection 목록이 생성됩니다.

| collection name              | badge       |
| ---------------------------- | ----------- |
| ts_collection                | time-series |
| system.buckets.ts_collection |             |
| system.views                 |             |

사용자는 ts_collection 을 만들고 해당 collection 을  통해서 동작하지만 실제로 내부에서는 View와 Buckets 를  시계열데이터에 최적화 시켜서 Timeseries Collection 으로 만든 것입니다. 
![[Pasted image 20241013000356.png]]


여기서는 내부에서 사용되는 View 와 Buckets 에 대해서 자세히 설명하진 않겠습니다. 
자세한 설명이 필요하신 분은 아래의 문서 링크를 확인해보시길 바랍니다.

[[https://www.mongodb.com/ko-kr/docs/manual/core/views/#std-label-views-landing-page|Views - MongoDB Manual]]
[[https://www.mongodb.com/ko-kr/docs/manual/reference/system-collections/#mongodb-data--database-.system.buckets|System Collections - MongoDB Manual]]

---
# 사용 시 주의 할 점








---
# 출처

>Time Series - MongoDB Manual
>https://www.mongodb.com/docs/manual/core/timeseries-collections/

> Time Series Data Introduction 
> https://www.mongodb.com/resources/basics/time-series-data-analysis

