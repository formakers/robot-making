AI 로봇 제작 마스터 클래스
8단계 : YOLO 객체인식 (Object Detection)

7단계에서 OpenCV로 영상을 처리했다면, 이제는 AI가 영상 속 물체가 무엇인지 스스로 인식하는 단계입니다.

이번 단계에서는 YOLO(You Only Look Once)를 이용하여 사람, 컵, 병, 자동차 등 다양한 물체를 실시간으로 인식하고, 이후 로봇팔이나 이동로봇과 연동하는 기초를 배웁니다.

이번 단계의 목표

학습이 끝나면 다음과 같은 프로그램을 만들 수 있습니다.

웹캠에서 사람 찾기
컵 찾기
병 찾기
휴대폰 찾기
여러 개의 물체 동시에 찾기
중심 좌표 계산
거리 계산 준비
OpenCV와 연동
Arduino 연동
ROS2 연동 준비
무엇을 배우는가?
1. AI 객체인식이란?

기존 OpenCV

→ 색상

→ 윤곽선

→ 모양

으로 찾았습니다.

예)

파란 공 찾기

빨간색 찾기

원 찾기

하지만

YOLO는

사진만 보고

"이건 컵"

"이건 사람"

"이건 강아지"

를 스스로 구분합니다.

예시

웹캠

↓

YOLO

↓

Person
Cup
Bottle
Mouse
Laptop
Keyboard

이처럼 이름까지 알려줍니다.

2. YOLO란?

YOLO

=

You Only Look Once

한 번만 영상을 보고

모든 객체를 찾는 AI입니다.

장점

매우 빠름
정확도 높음
실시간 가능
GPU 사용 가능
CPU도 가능

그래서

드론

로봇

자율주행

공장

모두 사용합니다.

3. YOLO 버전

현재 많이 사용하는 버전

YOLOv8

YOLOv9

YOLOv10

YOLO11

처음 배우는 사람은

YOLO11 또는 YOLOv8부터 시작하면 충분합니다.

4. 설치

Python 설치되어 있다면

pip install ultralytics

설치 확인

python
from ultralytics import YOLO

오류가 없으면 성공입니다.

5. 첫 번째 프로그램
from ultralytics import YOLO

model = YOLO("yolo11n.pt")

처음 실행하면

자동으로 모델을 다운로드합니다.

6. 이미지 인식
from ultralytics import YOLO

model = YOLO("yolo11n.pt")

results = model("test.jpg")

results[0].show()

실행 결과

사진 위에

person
cup
dog

가 표시됩니다.

7. 웹캠 실시간 객체인식
from ultralytics import YOLO
import cv2

model = YOLO("yolo11n.pt")

cap = cv2.VideoCapture(0)

while True:

    ret, frame = cap.read()

    results = model(frame)

    annotated = results[0].plot()

    cv2.imshow("YOLO", annotated)

    if cv2.waitKey(1) == 27:
        break

cap.release()
cv2.destroyAllWindows()

ESC를 누르면 종료됩니다.

8. 객체 이름 출력
for r in results:

    for box in r.boxes:

        cls = int(box.cls)

        print(model.names[cls])

예)

cup

person

mouse
9. 신뢰도 출력
confidence = float(box.conf)

print(confidence)

예)

0.95

95%

정확도로 판단했다는 뜻입니다.

10. 좌표 얻기
x1,y1,x2,y2 = box.xyxy[0]

예)

120
80
300
400
11. 중심 좌표 계산
cx = (x1+x2)/2

cy = (y1+y2)/2

이 좌표를 이용하여

로봇이 움직입니다.

12. 컵만 찾기
if model.names[cls]=="cup":

    print("컵 발견")
13. 사람만 찾기
if model.names[cls]=="person":

    print("사람")
14. 여러 객체 찾기

동시에

person

cup

bottle

chair

laptop

모두 찾을 수 있습니다.

15. OpenCV와 연동

OpenCV

↓

영상 입력

↓

YOLO

↓

객체 찾기

↓

OpenCV

↓

화면 출력

16. Arduino 연동

YOLO

↓

컵 발견

↓

Serial 통신

↓

Arduino

↓

모터 회전

예)

ser.write(b"LEFT\n")

Arduino

if(cmd=="LEFT")
{
   //모터 회전
}
17. ROS2 연동

YOLO

↓

객체 중심 계산

↓

ROS Topic Publish

↓

MoveIt

↓

로봇팔 이동

18. 실제 응용
자동문
자동 분류기
불량 검사
사람 추적
얼굴 추적
로봇팔 Pick & Place
AGV
AMR
휴머노이드
19. 이번 단계 실습
실습 1

YOLO 설치

실습 2

사진 인식

실습 3

웹캠 인식

실습 4

컵만 찾기

실습 5

컵 중심 출력

예)

Cup

X=320

Y=210
실습 6

컵 중심에 원 그리기

cv2.circle(frame,(cx,cy),5,(0,0,255),-1)
실습 7

컵이 왼쪽이면

LEFT

오른쪽이면

RIGHT

출력하기

실습 8

Arduino에

LEFT

RIGHT

STOP

전송하기

실습 9

스텝모터 회전시키기

실습 10

컵 자동추적 완성

최종 프로젝트

이번 단계의 목표는 다음과 같은 시스템을 직접 구현하는 것입니다.

웹캠
   │
   ▼
OpenCV 영상 입력
   │
   ▼
YOLO 객체인식
   │
   ▼
컵 중심 좌표 계산
   │
   ▼
Arduino 시리얼 통신
   │
   ▼
스텝모터 제어
   │
   ▼
컵 자동 추적

이 프로젝트를 완성하면 이후 Depth Camera, MoveIt, AI Pick & Place, LeRobot 단계의 핵심 기반 기술을 갖추게 됩니다.

GitHub 파일 구성

robot-making 저장소에 다음 파일을 추가하는 것을 권장합니다.

robot-making/
├── README.md
├── 01_robot_basics.md
├── 02_electronics_basics.md
├── 03_arduino_basics.md
├── 04_motor_basics.md
├── 05_sensor_basics.md
├── 06_python_basics.md
├── 07_opencv_basics.md
└── 08_yolo_object_detection.md

파일 생성 후 GitHub에 올리는 명령은 다음과 같습니다.

cd ~/robot-making

nano 08_yolo_object_detection.md

내용을 붙여넣고 저장한 뒤:

git add 08_yolo_object_detection.md
git commit -m "8단계 YOLO 객체인식 추가"
git push origin main
다음 단계 (9단계)

기구설계(CAD, 3D 프린터)

Fusion 360 기초
Onshape 기초
FreeCAD 기초
3D 프린터 출력 과정
로봇 부품 설계
브래킷 및 링크 제작
실제 조립을 위한 설계 노하우

9단계에서는 AI와 전자제어를 실제 하드웨어와 연결하기 위한 기계 설계와 제작 기술을 배우게 됩니다.
