---
title: MongoDB의 time series collection
description: MongoDB 5.x 에 정식으로 추가된 time series collection에 대해서 조사합니다.
aliases: 
tags:
  - MongoDB
  - OLAP
draft: true
date: 2024-08-22
---
OLAP 관련하여 조사하다가 MongoDB의 새로운 기능 time series collection에 대해서 알게 되었습니다. 해당 기능을 사용할 경우에는 기존 collection 과 어떤 차이가 있는지 확인해보고 간단하게 쿼리를 테스트 해봤습니다.

MongoDB의 공식문서에 설명되어 있으나 한국어 문서는 신경망 번역을 사용한 듯 합니다. 영어 문서에서 직접 읽으시거나 다른 번역 툴을 사용하시는 걸 추천드립니다. 


---
# 개념

time series collection에 대해서 공식 문서를 정리한 내용입니다. 




---
# 출처

>Time Series - MongoDB Manual
>https://www.mongodb.com/docs/manual/core/timeseries-collections/