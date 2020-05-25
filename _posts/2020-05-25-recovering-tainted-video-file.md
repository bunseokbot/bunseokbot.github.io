---
title: 손상된 영상 파일 복구하기
description: 손상된 MPEG-4 Part 14 영상 파일 포멧 복구하기
categories:
 - forensic
tags: forensic
---

# 손상된 영상 파일 복구하기

## 계기

어쩌다 보니 손상된 MPEG-4 Part 14 파일 포멧의 영상을 복구 해야 할 일이 있었다. 

무슨 일이 있었는 지는 알 수 없었지만 영상이 엄청나게 깨져 있었다. (아마 패킷 유실 때문에 그런 것 같다.)

근데 이걸 빠르게 복구 해야 했었기 때문에 (30분 정도) 그 시간 동안의 스토리를 순서대로 작성 해볼려고 한다.

최대한 많은 사람들이 읽을 수 있게 쉽게 쓸려고 노력까진 해봤는데 어려웠다.



## Introduce MPEG-4 Part 14

우선 영상 파일을 복구하기 전에 처음 접하는 파일 포멧이므로 우선 포멧에 대해서 알아 보도록 하자.

이 글은 해당 [위키피디아]( https://en.wikipedia.org/wiki/MPEG-4_Part_14 ) 에서도 나오는 내용을 기반으로 이해 했다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/Relations_between_ISO_Base_Media_File_Format_and_MP4_File_Format.svg/1920px-Relations_between_ISO_Base_Media_File_Format_and_MP4_File_Format.svg.png)

MP4를 구성하는 주요 데이터는 아래와 같다.

* 영상
  * MPEG-H Part 2, MPEG-4 Part 10, MPEG-4 Part 2 등등
* 음성
  * MPEG-4 Part 3, ALS, SLS 등등
* 자막
  * MPEG-4 Timed Text



그러니깐 그냥 간단하게 말하면 MP4는 위의 데이터가 저장되어 있는 무수한(?) 박스들의 집합이다.

실제로 010 에디터를 통해서 보면 ftyp, moov, free, mdat과 같은 박스들이 보인다.



## 삽질기

여튼 소개는 간단하게 하고 (궁금하면 찾아보자) 이걸 복구하게 된 방법에 대한 삽질기를 써 볼려고 한다.

원래 고전적으로 쓰던 mp4box, recoveryvideo와 같은 프로그램을 쓸 수 있는 상황이 아니였다.

내가 맘대로 프로그램을 깔 수 있는 상황이 아니였고, 그렇다고 외부 서버에 붙어서 작업하기엔..

그리고 이게 어떤 프로그램으로 촬영 된 영상인지 확인할 수 있는 방법이 없었다.

(대부분의 복구 프로그램은 이를 기반으로 작업한다.)



그래서 그냥 추정해서 만들었다.

공개된 프로그램을 찾느니 그냥 나만의 방식으로 만드는게 빠르지 않을까 싶었다. (시간도 없었고)



우선, 이미 가지고 있는 정상적인 MP4 파일을 최대한 모았다.

[샘플 파일]( https://file-examples.com/index.php/sample-video-files/sample-mp4-files/ )도 좋고, 개인적으로 가지고 있는 파일들에 대해서도 최대한 수집했다.



그리고 나서 영상 복구 방법에 대한 논문, 자료들을 찾아봤다.

아래 링크들은 찾아 보면서 도움이 되었던 링크들이였다.

*  http://dfrws.org/sites/default/files/session-files/paper-forensic_analysis_of_video_file_formats.pdf 
*  [https://wikileaks.org/sony/docs/05/docs/DECE/TWG/TechnicalWorkGroup_1.0.3%20Specs/TechnicalWorkGroup_Download/1.0.3_459/CFFMediaFormat-1.0.3.pdf](https://wikileaks.org/sony/docs/05/docs/DECE/TWG/TechnicalWorkGroup_1.0.3 Specs/TechnicalWorkGroup_Download/1.0.3_459/CFFMediaFormat-1.0.3.pdf) 
*  https://ieeexplore.ieee.org/document/8419785 



저 문서들과 다른 읽었던 글 들을 기반으로 이상한 점을 찾기 시작했다. (with 010 Editor)

일단 mdat 박스의 boxheader.size의 값이 0인 부분이 제일 이상했다. 이렇게 되면 사이즈가 0이여서 뒷 값을 읽지 못하기 때문에 의미가 없어진다.

이 부분을 그냥 아무 쓰레기 값이나 채우고 Windows Media Player로 재생 해 보았다.

결과는.. 그냥 뭐 크게 다르지 않았다. ffmpeg도 똑같은 오류를 내뿜는다.

여기서 생각한 결론은 이게 워낙 깨지는 부분도 많고



두번째로 시도했던 방법은 MP4 파일 파서를 이용해서 기존 양식에 저기서 최대한 정상적인 박스들을 가져다 인용하는 방식으로 복구를 시도 해 보았다.

아까 저 샘플 파일이랑 가지고 있는 모든 mp4 파일을 끌어다가 파싱하고, 파싱한 골격에다가 손상된 영상에서 살아 있는 box 내에 있는 데이터를 가져다가 붙여 넣는 방식을 이용했다.

(*나중에 알았는데 이게 untrunc에서 쓰는 방식이라고 한다.. 근데 untrunc 써 보긴 했는데 잘 안 되었다.)



근데 ffmpeg으로 영상을 읽다가 이상한 점을 발견했다..

```
[mov,mp4,m4a,3gp,3g2,mj2 @ 000002732f8ab440] moov atom not found
original.mp4: Invalid data found when processing input
```

moov atom을 찾을 수 없다고..?

![](/assets/images/video-recovery/hex.PNG)

진짜 없네..?



여기서 세울 수 있는 가설들

* 초반에 네트워크 문제던 업로드 문제던 기타 문제던 앞 부분이 일부 손상 되었다.
* 이 과정에서 moov를 날려 먹었다.
* moov만 잘 살려 주면 꽤 살릴 수 있을 것 같았다.



moov란 파일 시작에 꼭 있어야 하는 것으로 영상의 모든 메타데이터를 담는 박스를 말한다.

잘 모르겠다면 [상세 설명]( https://www.adobe.com/devnet/video/articles/mp4_movie_atom.html ) 을 참고 하도록 하자.



이게 없으니깐 어떤 파일이고, 어떻게 읽어야 하는지에 대하서 알 수가 없었던 것 이였다.

그래서 일단 moov를 구성하고 있는 mvhd, iods, trak를 hex로 만들어 주었다.

* mvhd - movie header
* iods - object descriptors
* trak - track

만들 때 [참고했던 블로그]( [https://videocube.tistory.com/entry/MP4-%EB%B6%84%EC%84%9D-%ED%95%98%EA%B8%B0-MPEG4-%ED%8C%8C%ED%8A%B8-14](https://videocube.tistory.com/entry/MP4-분석-하기-MPEG4-파트-14) )를 보고 moov를 재건했다. 여기서 timescale은 계속 조정 하면서 데이터를 조정했다.

ffmpeg으로 유효성 검사를 하고, 계속 moov에 가변적인 데이터를 수정 해 가면서 조건이 맞는 경우를 찾았다.

* ffmpeg으로 읽었을 때 문제가 없었을 것
* 복구된 영상은 20분 이상일 것 (이건 도박 이였는데 다행히 잘 되었다)



## 결과

전체 영상이 얼만진 알 수 없었지만, 20분 넘는 영상이 정상적으로 복구 되었다. (오예!)

좀 더 좋은 방법이 있지 않을까 생각 해봤지만 내 머리로는 일단 이게 한계였다.



## 회고

급하게 하게 되기도 했고 MPEG도 처음이였지만 나름 재밌었다.

더 자세하게 쓰고 싶었지만, 내일 할 일이 많기 때문에..

나중에 시간되면 더 써볼 수 있으면 좋겠다.. 끝