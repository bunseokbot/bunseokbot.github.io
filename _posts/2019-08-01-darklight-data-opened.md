---
title: 다크웹 데이터를 공개합니다
description: DarkLight를 통해 다크웹에서 수집한 데이터를 여러분들께 공개합니다.
categories:
 - darkweb
tags: darkweb, data mining, opensource
---
# 다크웹 데이터를 공개합니다
## 서론

제목을 보고 많은 분들이 이미 아시겠지만, 다크웹에서 수집한 데이터를 공개하게 되었습니다.

이 프로젝트를 시작하게 된 계기는 **프로젝트를 통해서 해외 발표를 다녀보자** 와 **다크웹이 궁금해서** 로 시작하게 되었지만 그 과정에서 뜻하지 않게 많은 것들을 배우게 되었습니다. (물론, 제출한 CFP는 다 떨어져서 한번도 가본 적은 없습니다 ㅋㅋ)

원래는 REST API 형태로 해서 다양한 기능을 구상하고 실제로 일부 부분을 개발 하였지만, 더 관심있고 하고싶은 프로젝트가 생겨 시작하게 되면서 이 프로젝트에 더 이상 신경쓰지 못할 것 같아 이러한 결정을 내리게 되었습니다.

제 분수에 맞지 않게 많은 분들이 이 프로젝트에 관심을 가져 주셨습니다. 이 부분에 대해서는 진짜 엄청나게 감사드립니다 :bow:



## 공개

**제가 다크웹에서 크롤링 한 데이터를 세상에 공개합니다.**

단, 제공된 데이터를 사용함에 있어 아래와 같은 조건을 명시합니다.

1. 타인에게 공유하는 것을 제한하지 않지만, 왠만하면 신청 과정을 통해 데이터를 받을 수 있도록 부탁드립니다.
2. 스크린샷 데이터는 제공되지 않습니다.
3. 상업적 목적으로 사용하는 부분까지 제한하지는 않습니다만, 데이터를 사용한 경우 출처를 **정확히** 밝혀 주시기 바랍니다.



추가적으로, 만약 출처를 어떻게 밝힐 지 잘 모르겠다면 아래와 같이 밝혀주시면 됩니다.

```
Namjun Kim (bunseokbot@gmail.com)
Sejong University, Wellbia.com Co., Ltd.
```



## 신청하기

bunseokbot@gmail.com 으로 아래와 같은 내용을 담아 보내주시면 데이터가 담긴 링크를 보내 드립니다.

만약, 48시간 이내에 답장을 못 받으셨다면 주저하지 마시고 연락해 주시기 바랍니다. (누락될 수 있습니다)

* 이름 (선택사항)
* 이메일
* 소속
* 사용 목적



## 사용법

도커 이미지를 백업한 tar 파일 형태로 제공됩니다. 아래 명령어를 통해 도커 환경에서 container를 구동할 수 있습니다.

```
docker load < darklight.tar
docker run -d --name darklight-elastic -p 9200:9200 bunseokbot/darklight-data:latest
```

load에 성공하여 컨테이너를 실행시키고, 9200번 포트를 통해 접속하면 elasticsearch 가 정상적으로 떠 있는 것을 볼 수 있습니다.

여기서 여러 index가 있는데 각 index에 대한 설명은 아래와 같습니다.

* domain (도메인 정보, 서버 환경 정보)
* history (사이트 정보, 소스코드, 언어셋, 제목 등)



## 끝으로

*뭔가를 시작하는 것도 중요하지만, 끝내야 할 때 끝낼 수 있는 것이 제일 좋은 것이다.*

자그만 프로젝트였지만 문제 없이 잘 끝낼 수 있는 것 같아서 좋았습니다.

**Tschüss!**



### Special thanks to..

* 이상섭
* 세종대학교 정보보호동아리 S.S.G
* 이름을 밝힐 수 없는 A모씨
* 그리고 프로젝트를 지켜봐 주신 여러분



### Gurkha Project

새로운 프로젝트의 이름은 세계 최강의 용병의 이름인 **Gurkha** 에서 본따 지어졌습니다.

단순히 데이터를 제공하는 플랫폼이 아닌 데이터를 응용하여 실생활에 도움이 될 수 있는 프로젝트를 지향하고 있습니다.





# Disclosure of darkweb data

## Introduction

As you may already know from the title, we're now releasing data collected from the darkweb.

The reason why I started this project was because of the **curiosity of darkweb** and **desire as a speaker at oversea conferences**, but I learned many things unexpectedly in the process. (but, all my CFPs has been rejected lol)

Originally, I designed various functions as a REST API form and developed some features, but I decided that I couldn't proceed in this project anymore as I started to have a project that I was interested in and wanted to do.

A lot of people have been interested in this project so that it doesn't fit my place for me. Thank you very much for this. 🙇



## Public

I'm going to release the data which crawled on the darkweb to the world.

However, the following conditions are specified when using the data provided:

1. We don't restrict sharing to others, but please use data through the application process.
2. Screenshot data is not available.
3. We don't restrict use for commercial purposes, but if you use the data, please specify **the exact source**.

In addition, if you're not sure how to identify the source, you can identify it as follows.



## How to Apply

Please send the following information to bunseokbot@gmail.com and we'll send back to you with a link.

If you have not received your reply within 48 hours, please do not hesitate to contact me.

- Name (optional)
- E-mail
- Company Name / School Name / Department
- purpose of use



## Getting Started

It's provided in the form of a tar file that backup the image of the Docker. You can run container in the docker environment with the following command.

```
docker load < darklight.tar
```

You can see that elasticsearch container is running when you load and run the container and connect through port 9200.

There are several indexes,

- domain (domain information, server environment information)
- history (site info, html source code, language set, title, etc.)



## Finally

*It's important to start something, but it's best to finish it when you have to finish it.*

It was a little toy project, I'm very thankful that finish it without any problem.

**Tschüss!**



### Special thanks to..

- Sangsup Lee
- Sejong University, Information Security Hacking club named S.S.G
- The man who can't reveal the name
- YOU



### Gurkha Project

The name of the new project is based on **Gurkha**, the name of the world's strongest mercenary.

We are aiming at a project that can be useful for real world life by applying data that is not simply a data providing platform.



