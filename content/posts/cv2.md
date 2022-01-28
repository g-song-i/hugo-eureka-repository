---
title: Python - cv2 라이브러리를 이용해 동영상 읽기
description: Python - cv2 라이브러리를 이용해 파이썬에서 동영상 읽기
toc: true
authors:
  - songi
tags:
  - python
  - cv2
  - 영상처리
categories:
series: 
date: '2020-11-18'
lastmod: '2020-11-18'
draft: false
---
</br>

## Content

이번 포스트에서는 cv2 라이브러리를 이용해 파이썬에서 동영상과 이미지를 읽어 볼 거예요. 그동안 갑자기 파이썬 및 영상처리를 공부해야 하는 상황이어서 정신이 없다가 드디어 포스트 할 수 있는 짬이 났네요 ㅠ.ㅠ 설치를 위한 명령어는 다음과 같습니다. 아나콘다 프롬프트에서 cv2를 설치해주세요.

    pip install opencv-python

</br>
우선, 이미지 파일을 읽는 법은 다음과 같습니다. 간단하기 때문에 코드로 한 줄만 작성하고 넘어가도록 하겠습니다.
</br>
</br>

```python
image = cv2.imread('image_path')
```

'image_path'에는 이미지의 경로를 넣으면 됩니다. 상대경로도 절대 경로도 상관없고, 두 번째 파라미터로 이미지의 모드를  설정할 수 있습니다.
</br>
</br>
동영상을 읽는 코드는 다음과 같습니다. 

```python
import os
import pandas as pd
import numpy as np
import glob
import cv2
import time

def extract_frames(path):
    
    capture = cv2.VideoCapture(path)
    time.sleep(2.0)

    if not capture.isOpened():
        print("Error opening video")
        exit(0)
        
    while capture.isOpened():
        
        ret, frame = capture.read()
    
        if ret:
            cv2.imshow("show", frame)
        
        if cv2.waitKey(1) & 0xFF == ord('q'):
            exit(0)
        
    cv2.destroyAllWindows()
    capture.release()

video_path = "D:/download/file.mp4"
extract_frames(video_path)
```

우선 cv2 라이브러리의 VideoCapture 함수를 이용해 지정해준 path에 존재하는 동영상을 가져옵니다. 만약 capture가 제대로 이루어지지 않았다면 비디오를 여는 과정에서 에러가 발생했다는 문구를 띄우며 종료합니다. capture가 열려있다면 read() 함수를 이용해 비디오에서 frame을 읽게 됩니다. 만약 return이 있을 경우 imshow() 함수를 통해 가져온 frame을 보여줍니다. waitKey와 ord를 통해 'q'를 누르면 중료 하게 합니다. 마지막으로 꼭 윈도우를 전부 닫고 비디오를 릴리즈 해주세요! imshow를 통해 영상의 이미지를 확인하는 코드를 실행한 결과는 다음과 같습니다.
</br>
</br>
![img1 daumcdn](https://user-images.githubusercontent.com/57793091/151487028-7139763d-e443-4d6f-806e-35c2f79e406a.png)

## Recent Comment

해당 포스트는 제 publication에 있는 딥페이크 영상 탐지 논문을 쓸 때 사용했던 코드의 일부분이에요. 해당 프로젝트는 영상에서 프레임 캡쳐 -> 사람의 얼굴 인식 -> 얼굴 이미지 정렬 -> 특징 추출 (CNN) -> 딥페이크 영상 분류 (LSTM) 이런식으로 진행이 되었는데 ㅠㅡㅠ 코드를 어디 뒀는지 모르겠어서 나머지 부분은 업로드하지 못했네요. 혹시나 찾으면..... 더 업로드 할 수 있도록 해보려고 합니다!