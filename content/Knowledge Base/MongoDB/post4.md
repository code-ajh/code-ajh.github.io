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

MongoDB의 공식문서에 설명되어 있으나 한국어 문서는 번역기를 사용한 듯 합니다. 영어 문서에서 직접 읽으시거나 다른 번역 툴을 사용하시는 걸 추천드립니다. 


---
# 개념

Time Series Collections은 효과적으로 시계열 데이터를 저장하기 위해  MongoDB 5.0 에서 도입된 새로운 Collection 입니다. 

**기본적인 동작 원리는 비슷한 시간대의 데이터를 묶어 분리하는 것이며 추가로 Metafield 를 사용하여 추가적인 bucket 분리가 이루어집니다.** 시간 순서대로 데이터가 들어와도 저장은 반드시 시간 순으로 이루어집니다. 

![[mongodb_timeseries_collection.jpg]]

시간대 별로  저장하는 것은 다른 OLAP 시스템의 데이터 저장소들과 크게 다르지 않지만 지정한 metafield 별로 분리된다는 것이 다른 점으로 보입니다. 


metafield는 하나의 field 만 사용하는 것이 아닌 여러 개를 사용할 수 있습니다. 
해당 필드들을 기준으로 나누게 되므로 Cardinality 가 높아질수록 분리된 파일이 더 많이 생성되므로 Insert 성능도 느려지게 됩니다. 

이러한 특징 덕분에 MongoDB의 timeseries collection은 




---
# 생성

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

timeField : 시간 데이터로 사용할 필드 이름
metaField : bucket 분리에 사용될 필드 이름 
granularity : 시간 데이터를 어떤 단위로 분리할 것인지 \[ seconds | minutes | hours ] 
- bucketMaxSpanSeconds : 최대 시간 범위 
- bucketRoundingSeconds : 새롭게 생성되는 bucket의 timestamp 의 반올림 범위
	- bucketMaxSpanSeconds, bucketMaxSpanSeconds 는 동일한 값을 가지고 있어야 합니다.
expireAfterSeconds : TTL, 일반 collection 에서도 사용 가능한 기능





---
# 출처

>Time Series - MongoDB Manual
>https://www.mongodb.com/docs/manual/core/timeseries-collections/

> Time Series Data Introduction 
> https://www.mongodb.com/resources/basics/time-series-data-analysis

