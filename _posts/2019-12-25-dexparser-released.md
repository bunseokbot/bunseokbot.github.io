---
title: dexparser released!
description: dexparser 1.0.0 released
categories:
 - opensource
tags: opensource, dexparser
---

# dexparser

2015년 비오비 프로젝트의 사이드 작업 중에 일환으로 dexparser for Python 을 공개 하였었습니다.

당시에 제 코딩 실력이 많이 미천하다고 생각되어 정식 릴리즈를 하지 않고 0.0.1 알파 버전으로 공개 하였는데요.

그 이후로 놀랍게도 더 이상 안드로이드를 볼 일이 없게 되면서 업데이트를 진행하지 않았습니다.

그러던 중..



## Goodbye 2.7

지금으로부터 약 일주일 뒤, 2020년 1월 1일부로 은퇴한다는 소식을 듣게 되었습니다. (사실 원래 알았음)

그래서 제 개인 프로젝트들도 하나하나씩 Python 3.6 버전으로 포팅하는 작업을 하고 있는데요.

그 개인 프로젝트 중에 제일 마지막으로 dexparser 프로젝트를 Python 3 버전으로 포팅 하는데 성공 하였습니다.



## 프로젝트를 사용해주신 감사한 분들

직접적으로나, 간접적으로 dexparser 프로젝트를 많이 써 주시고 계셨습니다.

[soFrida]( https://github.com/june5079/soFrida/blob/master/dex/dexparser.py ) 의 dexparser.py 라는 파일에서도 dexparser 코드가 사용되었던 것을 보고 결정적으로 업데이트를 마음 먹게 되었습니다.



## 추가적으로

아래와 같은 작업을 업데이트 기간에 진행 하였습니다.



### 테스트 코드 추가

원활한 테스트와 코드 퀄리티 보장을 위해 간단한 테스트 코드를 추가 하였습니다.

[Travis CI]( https://travis-ci.com/bunseokbot/dexparser ) 도 붙여 두었으므로 master에 push되면 알아서 CI를 통해 pytest가 돌게 됩니다.



### Pythonic code

이전보다 좀 더 Pythonic한 naming으로 돌아왔습니다(?)

사실 정확하게 파이써닉한 네이밍이라는게 뭔진 모르겠지만 그래도 나름 다른 프로젝트를 기웃거리면서 비슷하게 따라는 해 보았습니다 :)



사실 기능적인 부분을 많이 추가하고 싶었는데 공부할 시간도 부족하고 제일 문제는 제가 잘 안쓰다 보니..

DEX 파일을 본지도 너무 오래되서 최근에 다시 안드로이드 스튜디오 깔아서 올리는데.. 너무 편한 것..



## 개선 사항

* Documentation



## 잡담

DEX 파일이 이제까지 업데이트 된 것들이 있는지 알기 위해 파서들을 검색했는데 왜 내 것만 나오지.. 흠..

PR은 항상 환영합니다 :)