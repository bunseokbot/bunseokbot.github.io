---
title: 모델을 훔치는 완벽한 방법
description: 안드로이드 애플리케이션에 embedded 되어 있는 머신러닝 모델 탈취하기
categories:
 - research
tags: opensource, research
---

# 모델을 훔치는 완벽한 방법

계절학기 미적분 시험을 보고 나서 한 편의 논문을 보게 되었다.

**Mind Your Weight(s): A Large-scale Study on Insufficient Machine Learning Model Protection in Mobile Apps**

내용을 읽기 싫으신 분들을 위해 간단히 요약하자면, 모바일 앱 내에 삽입되어 있는 머신러닝 모델을 leak 하는 방법에 대해 설명하고 있다고만 알면 된다.

논문을 읽으면서.. 오 그럼 이걸 이용하면 남들이 만든 모델을 맘대로 가져다 쓸 수 있는 것인가? 라는 생각이 들었다. 실제로 관련 연구들도 슬슬 이뤄지고 있는 걸로 봐서.. 뭔가 금맥이 흐르는 기운이 들었다.

![](https://coinpan.com/files/attach/images/198/641/030/078/45145c693c5c9260888ad9fdcd0efcef.png)

떡... 상각?

그래서 한번 만들어 보기로 했다. 간단히 머신러닝 모델을 추출해 추출해 주는 도구를..



## 애플리케이션이 머신러닝 모델을 보관하는 방법

참고한 논문에서 머신러닝 모델을 보관하는 방법에 대해 크게 세 가지로 명시하고 있다.

* assets 에 직접 모델을 보관하는 방법
* 라이브러리 (libs) 에서 불러오는 방법
* 암호화된 파일에서 복호화 한 뒤 불러오는 방법

여기에서 "assets 에 직접 모델을 보관하는 방법" 이 제일 쉬워 보여서 직접 해보기로 했다.



## Robster

자동으로 APK 파일을 넣으면 머신러닝 모델을 추출해 주는 도구의 이름을 Robster라고 지었다.

주소: https://github.com/bunseokbot/robster

원래는 frontend까지 해서 완벽하게 웹 서비스 형태로 제작 하고자 했는데 부족한 frontend 실력으로 인해 API 서버만 만들어진 반쪽짜리가 되어버렸다.

이 부분은 나중에 시간이 되게 된다면 꼭 보완 하도록 하겠다. (이래놓고 절대 안하는건..)



일단, Robster에서 지원하는 Machine Learning Framework는 아래와 같다.

| 종류            | 제작사               | 범위       |
| --------------- | -------------------- | ---------- |
| Tensorflow Lite | Tensorflow           | 탐지, 추출 |
| Tesseract       | Google               | 탐지       |
| ncnn            | Tencent              | 탐지       |
| MNN             | alibaba              | 탐지       |
| Caffe           | Berkeley AI Research | 탐지       |

우선 대표적으로 사용되는 5개의 머신러닝 프레임워크에 대해 모델 추출과 DEX 파싱을 통한 method call trace를 지원한다.

기타 분류되지 않는 파일에 대해서는 키워드 검색 기능 (머신러닝 모델에서 주로 사용되었던 키워드) 를 이용하여 suspicious model로 분류할 수 있도록 하였다.

그리고, 탈취 가능한 머신러닝 모델에 대해서 구조에 대한 정보를 파싱하여 어떠하게 설계 되었는지 그 구조를 파악할 수 있도록 구조 정보를 추가로 제공하고 있다. (예시를 보여주고 싶지만 너무 위험하므로..)



## Experiment

개발한 Robster를 이용하여 Google Play Store에 올라와 있는 앱 중 6,569개의 앱에 대해서 검사를 실시하였고, 총 73개의 앱에서 80개의 모델을 추출할 수 있었다.



### Framework

확인된 머신러닝 모델을 프레임워크 별로 분류하면 아래와 같다.

| 종류            | 갯수 |
| --------------- | ---- |
| Tensorflow Lite | 53   |
| Caffe           | 9    |
| Tesseract       | 8    |
| ncnn            | 8    |
| MNN             | 2    |



## Conclusion

이러한 탈취 방법(?) 을 통해 실제 Play Store에 올라 있는 앱들 중 많은 수의 앱은 아니였지만, 그래도 몇몇 앱에서머신러닝 모델이 저장되어 있음을 확인할 수 있었다.

논문에서도 말하지만, 이러한 과정을 통해 기업에서 투자한 R&D나 컴퓨팅 리소스를 무단으로 탈취하여 맘대로 사용할 수 있는 만큼 위험할 수 있다고 설명하고 있다.

일단 새 학기가 시작되서 당분간은 추가 개발을 하긴 어려울 수 있지만, 앞으로 시간이 난다면 고도화 작업을 해보는 것도 좋을 것 같다.

Special thanks to Andy Ahn.