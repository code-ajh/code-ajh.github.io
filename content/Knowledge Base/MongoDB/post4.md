---
title: MongoDB의 time series collection
description: MongoDB 5.x 에 정식으로 추가된 time series collection 조사
aliases: 
tags:
  - MongoDB
  - OLAP
draft: false
date: 2024-10-13
---
OLAP 관련하여 조사하다가 MongoDB의 새로운 기능 time series collection에 대해서 알게 되었습니다. 해당 기능을 사용할 경우에는 기존 collection 과 어떤 차이가 있는지 확인해보고 간단하게 개념과 주의점만 알아봅니다. 

---
# 개념

Time Series Collections은 효과적으로 시계열 데이터를 저장하기 위해  MongoDB 5.0 에서 도입된 새로운 Collection 입니다. 

**기본적인 동작 원리는 비슷한 시간대의 데이터를 묶어 Paritioning 하는 것이며 추가로 Metafield 를 사용하여 추가적인 bucket 분리가 이루어집니다.** 시간 순서대로 데이터가 들어와도 저장은 반드시 시간 순으로 이루어집니다. 

![[Pasted image 20240908145110.png]]


아래는 timeseries collection 생성 쿼리 예시입니다.
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

`timeseries` 키워드로 timeseries collection 으로 만든다는 것을 명시합니다.

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

사용자는 ts_collection 을 만들고 해당 collection 을  통해서 동작하지만 실제로 내부에서는 View와 Buckets 를  시계열 데이터에 최적화 시켜서 Timeseries Collection 으로 만든 것입니다. 
![[Pasted image 20241013000356.png]]


여기서는 내부에서 사용되는 View 와 Buckets 에 대해서 자세히 설명하진 않겠습니다. 
자세한 설명이 필요하신 분은 아래의 문서 링크를 확인해보시길 바랍니다.

[[https://www.mongodb.com/ko-kr/docs/manual/core/views/#std-label-views-landing-page|Views - MongoDB Manual]]
[[https://www.mongodb.com/ko-kr/docs/manual/reference/system-collections/#mongodb-data--database-.system.buckets|System Collections - MongoDB Manual]]

---
# 사용

사용에 관해서는 기존과 크게 다른 주의점은 없지만 문제가 생겨서 수정이 필요할 때는 일반 collection 에 비해서 더욱 큰 비용이 필요하다는 것입니다. 

timeseries collection 을 만들고 데이터를 넣는 순간 metaField 와 timeField 기준으로 복합 index가 생성됩니다.  예를 들어 위에서 만든 ts_collection 에서 index를 조회해보면 아래와 같이 나옵니다. 

```mongodb
sample> db.getCollection("ts_collection").getIndexes()

[
  {
    v: 2,
    key: { metadata: 1, timestamp: 1 },
    name: 'metadata_1_timestamp_1'
  }
]
```

timeseries collection은  index 와 bucket 두 번의 처리가 필요하기에 상대적으로 Insert 의 비용이 커지고 수정도 어렵습니다. 쿼리의 속도와 저장 효율 등이 올라갈 수 있지만 빠른 처리가 필요한 빅데이터 시스템에서는 느려진 데이터 삽입 속도로 인해 서비스 불가능한 정도일 수도 있습니다. 

추가로 지속적으로 데이터를 넣을 수 있는 만큼 삭제 쿼리를 통해 데이터를 지우기보다는 TTL을 사용하여 삭제하는 것을 추천합니다. 

---
# 결론

timeseries collection 은 일반 collection 에 비해 더욱 많은 고민이 필요합니다.

1. 추후 변경이 힘들기 때문에 현재와 미래의 요구 사항을 고려한 설계가 필요합니다.
2. 효율적인 리소스 사용을 위해 필요 없는 필드는 제거하고 MetaField를 조정하는 등의 테스트와 설계가 요구됩니다.
3. 단순히 시계열 데이터라고 해서 무조건적으로 사용하는 것은 바람직하지 않습니다.

결론적으로, timeseries collection 도입 시에는 다각도의 분석과 고려가 선행되어야 합니다. 이를 통해 데이터의 특성을 최대한 활용하는 효율적인 시스템을 구성할 수 있습니다.


---
# 읽을만한 글

> \[MongoDB] Bucket 패턴으로 대용량 데이터 처리 — 스키마편
>https://medium.com/@seungwooyu2000kr/mongodb-bucket-패턴으로-대용량-데이터-처리-스키마편-a9241c7ed918

> MongoDB Timeseries를 활용기 - BBROS 기술블로그
> https://boostbrothers.github.io/2024-02-20-mongo-timeseries/

---
# 참고

>Time Series - MongoDB Manual
>https://www.mongodb.com/docs/manual/core/timeseries-collections/

> Time Series Data Introduction 
> https://www.mongodb.com/resources/basics/time-series-data-analysis