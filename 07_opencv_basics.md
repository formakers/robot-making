AI 로봇 제작 마스터 클래스
7단계 : OpenCV 영상처리 기초

지금까지의 학습

✅ 1단계 : 로봇의 기초 이해
✅ 2단계 : 전기와 전자 기초
✅ 3단계 : 아두이노 기초
✅ 4단계 : 모터의 종류와 제어
✅ 5단계 : 센서와 입력장치
✅ 6단계 : Python 프로그래밍
▶ 7단계 : OpenCV 영상처리 (이번 단계)
OpenCV란?

OpenCV(Open Source Computer Vision Library)는

카메라 영상을 분석하는 가장 유명한 오픈소스 라이브러리입니다.

쉽게 말하면

"컴퓨터에게 눈을 만들어주는 프로그램"

이라고 생각하면 됩니다.

왜 OpenCV를 배워야 하는가?

AI 로봇은

사람처럼

본다.
인식한다.
판단한다.
움직인다.

이 과정 중

"본다"를 담당하는 것이 OpenCV입니다.

예를 들어

카메라 화면에서

컵 찾기
사람 찾기
얼굴 찾기
손 찾기
QR코드 읽기

모두 OpenCV가 담당합니다.

OpenCV가 하는 일

OpenCV는

① 카메라 연결

USB 카메라

웹캠

산업용 카메라

IP카메라

모두 연결 가능합니다.

예)

import cv2

cap = cv2.VideoCapture(0)
② 화면 출력
while True:
    ret, frame = cap.read()

    cv2.imshow("Camera", frame)

    if cv2.waitKey(1)==27:
        break

ESC를 누르면 종료됩니다.

③ 이미지 저장
cv2.imwrite("photo.jpg", frame)
④ 동영상 저장
fourcc = cv2.VideoWriter_fourcc(*'XVID')
⑤ 이미지 크기 변경
small = cv2.resize(frame,(640,480))
⑥ 회전
rotate = cv2.rotate(frame,cv2.ROTATE_90_CLOCKWISE)
⑦ 좌우반전
flip = cv2.flip(frame,1)
⑧ 흑백 영상
gray = cv2.cvtColor(frame,cv2.COLOR_BGR2GRAY)
⑨ Blur

노이즈 제거

blur = cv2.GaussianBlur(gray,(5,5),0)
⑩ Edge Detection

윤곽선 검출

edge = cv2.Canny(blur,100,200)
OpenCV 영상처리 흐름
카메라

↓

영상 입력

↓

영상 전처리

↓

색상 변환

↓

노이즈 제거

↓

윤곽선 검출

↓

객체 검출

↓

AI 인식(YOLO)

↓

로봇 제어
OpenCV의 핵심 함수
함수	설명
VideoCapture()	카메라 연결
imshow()	영상 출력
waitKey()	키 입력
resize()	크기 변경
rotate()	회전
flip()	좌우반전
cvtColor()	색상변환
GaussianBlur()	블러
Canny()	윤곽선
findContours()	윤곽선 찾기
rectangle()	사각형
circle()	원
line()	선
putText()	텍스트
영상처리 순서
원본

↓

Gray

↓

Blur

↓

Threshold

↓

Contour

↓

객체 인식
Threshold

흑백으로 변환합니다.

ret, thresh = cv2.threshold(gray,127,255,cv2.THRESH_BINARY)
Contour

윤곽선을 찾습니다.

contours, _ = cv2.findContours(
    thresh,
    cv2.RETR_EXTERNAL,
    cv2.CHAIN_APPROX_SIMPLE
)
사각형 그리기
cv2.rectangle(
    frame,
    (100,100),
    (300,300),
    (0,255,0),
    2
)
원 그리기
cv2.circle(
    frame,
    (320,240),
    20,
    (0,0,255),
    -1
)
글자 출력
cv2.putText(
    frame,
    "Robot",
    (50,50),
    cv2.FONT_HERSHEY_SIMPLEX,
    1,
    (255,0,0),
    2
)
OpenCV와 AI의 관계

OpenCV만으로도

색상 검출
원 검출
선 검출
얼굴 검출

은 가능합니다.

하지만

YOLO는

컵
사람
자동차
강아지
병

처럼 학습된 객체를 인식합니다.

즉

OpenCV
↓

영상 전처리

↓

YOLO

↓

객체 인식

↓

로봇팔 제어
앞으로 배우게 될 내용

다음 단계에서는 OpenCV를 기반으로 AI 기능을 점점 확장합니다.

8단계 : YOLO 객체인식
9단계 : 기구설계(CAD, 3D 프린터)
10단계 : 나만의 로봇팔 제작
실습 과제
실습 1 : 웹캠 화면 출력
USB 카메라 연결
실시간 영상 보기
ESC 키로 종료
실습 2 : 흑백 영상 만들기
컬러 영상을 Gray 영상으로 변환
원본과 비교하기
실습 3 : 블러와 윤곽선 검출
GaussianBlur 적용
Canny Edge로 윤곽선 확인
실습 4 : 도형과 텍스트 그리기
사각형, 원, 선 그리기
화면에 텍스트 출력하기
실습 5 : 간단한 색상 추적
특정 색상(예: 빨간색 또는 파란색)만 검출하여 위치를 표시하기
GitHub 파일명

이번 단계는 아래 파일명으로 저장하면 전체 시리즈와 일관성이 유지됩니다.

07_opencv_basics.md

다음에는 **GitHub에 바로 올릴 수 있는 07_opencv_basics.md 전체 문서(이론 + 실습 + 예제 코드 + 학습 정리)**를 초보자도 따라 할 수 있도록 완성된 형태로 만들어 드리겠습니다.
