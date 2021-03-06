---
title: 누구나 만들 수 있는 IoT 방범 장비 만들기
description: 기기 분석하고 남은 IoT 기기 엮어서 나만의 특색있는 방범 도구 만들기
categories:
 - iot
 - security
 - cloud
tags: iot
---

# 누구나 만들 수 있는 IoT 방범 장비 만들기

*해당 글은 추후에 계속 업데이트 될 예정입니다. (2020-01-26)*

최근에 여러 일들이 있으면서 방범 장비를 만들어야 할 일이 생겼었다. 업체에 구독하면 최소 계약기간이 있어 이를 지키긴 어려운 상황이라 집에서 분석하고 남아 돌던 IoT 기기들을 이용해 나만의 방범 장비를 만들어 보기로 하였다.

누구나 쉽게(?) 만들 수 있도록 하기 위해 최대한 쉽게 설명하도록 노력까진 해 보았다.

나중에 이 삽질을 다른 사람들이 하지 않기를 바라면서..



## 시작

뭐부터 시작할까 하다가 기기 구매법부터 설명 해보고자 한다.

### 기기 매수법

주로 아마존에서 구하거나 아니면 `중고로운 평화나라` 에서 구할 수 있다. 생각보다 호기심에 샀다가 맘에 안들어서 파는 사람들이 많으니 잘 구해 보도록 하자.

아니면 외국 나가서 직구를 하는 방법도 있다. Google Home Mini를 그런 식으로 이시국에서 작년 초에 샀다 :)

### AWS

여기서는 아래와 같은 AWS 기술을 사용 하였다.

* Simple Notification Service
* AWS Lambda
* API Gateway
* DynamoDB
* Simple Secure Storage (S3)

내가 서버를 구축하긴 귀찮고, SMS 발송과 같은 귀찮은 작업들을 알아서 다 해주니 너무 편했다 :)

최초 가입하면 100달러 1년 Free-Tier 크레딧 주니 가입 안한 사람들은 한번씩 해보도록 하자.



## 패턴

아래와 같은 조건으로 방범 알림 패턴을 구성했다.

* 창문 열림
* 정문 열림
* 비명 소리
* 모션 센서 작동



## 알림

방범 장비들의 핵심은 바로 `알림` 기능이다. 알림 받기 제일 편하고 개발하기 쉬운게 SMS라..

외국 나가서 인터넷이 되지 않는 상황이여도 자동으로 로밍 통해서 SMS은 수신되니 이걸 선택했다.



## 앱과의 연동

내가 현재 집에 있는지 없는지를 판단하기 위해 휴대폰에 내가 만든 앱을 설치했다.

앱의 간단한 기능은 아래와 같은데..

* 5분마다 한번씩 내 위치 변동 시 발송
* 기기 연동 상태 조회

정도로 비교적 간단한 기능들로만 구성되어 있다.



여기서 특정 조건문을 통해 위치 정보 발송 횟수를 최소화 하였다.

* Home Wi-Fi에 연결되어 있는 경우
* 수동으로 Disarm 명령을 override 한 경우



## 테스트

집에서 미친놈처럼 비명도 질러보고 창문도 강제로 열려고 해봤다.

잘 되는 것 확인했다. 생각보다 AWS 인프라가 너무 빨라서 놀랐을 뿐..



## 결과

도둑이 들어본 적이 없어서 잘 모르겠는데 일단 오탐이 발생하진 않았다.



## 마지막으로

이걸 실제 서비스로 할려면 사실 고려해야 할 부분들이 너무 많다.

* 위치정보서비스 관련 사업자
* 개인정보보호법

그러나 혼자 쓰고 버릴 것이기에 그냥 이러한 법률은 크게 고려하지 않았다. 

그러니 만약에 이걸 가지고 실제 사업자를 하고 싶은 사람이 있다면 극구 말리고 싶다. 법이 너무 빡세다

사실 더 많이 쓰고 싶은데 코드 정리도 해야 하고 다른 공부 할 것들이 많아서 그냥 시간 나면 더 쓰도록 노력..



최근에 아이폰을 사서 NFC 태그를 이용한 Automation을 연동해서 자동으로 Arm, Disarm 하는 부분과 이를 응용해서 다양한 것들을 해볼려고 한다.

하고 재밌으면 공유 하는걸로..