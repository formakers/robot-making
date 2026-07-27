AI 로봇 제작 마스터 클래스
10단계 : 나만의 로봇팔 제작 (Robot Arm Build)

이 단계에서는 지금까지 배운 내용을 모두 활용하여 실제로 동작하는 4~6축 로봇팔을 제작합니다.

이 단계를 끝내면 여러분은

직접 설계한 로봇팔 제작
아두이노 제어
Python 제어
ROS2 연동 준비
MoveIt 연동 준비

까지 가능한 수준이 됩니다.

학습 목표

이번 단계에서는

"내 손으로 움직이는 로봇팔을 만든다."

가 목표입니다.

전체 과정
기획

↓

설계

↓

부품선정

↓

조립

↓

배선

↓

모터제어

↓

Forward Kinematics

↓

Inverse Kinematics

↓

Python 제어

↓

ROS2 준비
1. 로봇팔의 구조 이해

대표적인 로봇팔은 아래와 같습니다.

Base

↓

Shoulder

↓

Elbow

↓

Wrist

↓

Gripper

예를 들어

Joint1
회전

Joint2
어깨

Joint3
팔꿈치

Joint4
손목

Joint5
회전

Gripper
집기

입니다.

자유도(DOF)

DOF(Degree Of Freedom)

자유도란

움직일 수 있는 축의 개수입니다.

예를 들어

2축

좌우

위아래

이면

2DOF

입니다.

4축
Base

Shoulder

Elbow

Wrist
5축
+ Wrist Rotation
6축

산업용 로봇 대부분

6 DOF

입니다.

이번 과정 추천

처음이라면

4축

부터 시작합니다.

이후

5축

6축

으로 업그레이드합니다.

2. 로봇팔 설계

CAD 프로그램

Fusion 360
FreeCAD
SolidWorks

중 하나를 사용합니다.

설계 순서는

Base

↓

Link1

↓

Link2

↓

Link3

↓

Gripper

입니다.

3. 링크(Link)

링크란

모터와 모터 사이의 구조물입니다.

예

Motor

↓

Link

↓

Motor

↓

Link
4. 관절(Joint)

Joint는

회전하는 부분입니다.

예

Servo

↓

Joint

↓

Link
5. 모터 선택

추천

① MG996R

저렴
입문용

② Dynamixel XL430

ROS 지원
산업용 구조

③ Dynamixel XM430

더 강력

④ OpenManipulator-X 모터

ROS2 연동이 매우 쉬움

6. 프레임 제작

방법은

① 3D 프린터

PLA

PETG

ABS

② 알루미늄

2020 프로파일

③ CNC 가공

알루미늄

아크릴

카본

7. 그리퍼 제작

가장 많이 사용하는 형태

Parallel Gripper

양쪽이 동시에 움직입니다.

예

|     |

↓

 \   /

↓

[물체]
8. 제어보드

추천

Arduino UNO
ESP32
OpenRB-150
STM32
9. 전원

절대 USB만 사용하지 않습니다.

예

12V

↓

DC-DC

↓

5V Servo

또는

24V

↓

Dynamixel
10. 배선

구성

Power

Signal

Ground

배선은 가능한 한

짧게
깔끔하게
케이블 체인 사용

을 권장합니다.

11. 조립 순서
Base 제작

↓

모터 장착

↓

링크 장착

↓

배선

↓

테스트

↓

그리퍼

↓

최종 조립
12. Forward Kinematics

순기구학

각도를 알면

끝 위치를 계산합니다.

예

Joint

↓

Joint

↓

Joint

↓

End Effector
13. Inverse Kinematics

역기구학

목표 좌표를 입력하면

(X,Y,Z)

↓

각도 계산

을 수행합니다.

14. Arduino 제어 예제
#include <Servo.h>

Servo base;

void setup() {
  base.attach(9);
}

void loop() {

  base.write(30);
  delay(1000);

  base.write(90);
  delay(1000);

  base.write(150);
  delay(1000);
}
15. Python 제어 예제
import serial
import time

ser = serial.Serial('/dev/ttyACM0',115200)

time.sleep(2)

ser.write(b'90\n')
16. ROS2 준비

다음 단계에서 사용할 구조

URDF

↓

robot_state_publisher

↓

MoveIt

↓

RViz
17. 제작 시 주의사항
중심이 무너지지 않도록 무게 배분
큰 토크가 필요한 축에는 감속기 사용
전원과 신호선 분리
케이블이 관절에 걸리지 않도록 정리
비상 정지(E-Stop) 스위치 준비
18. 실습 프로젝트
실습 1

1축 회전 테스트

Servo

↓

30°

↓

90°

↓

150°
실습 2

2축 로봇팔

컵 잡기

실습 3

3축

좌표 이동

실습 4

4축

픽앤플레이스(Pick & Place)

실습 5

카메라 장착

YOLO 객체 추적

19. 이번 단계 완성 결과

이번 단계를 완료하면 다음과 같은 로봇 시스템을 직접 제작할 수 있습니다.

카메라
   │
   ▼
YOLO 객체인식
   │
   ▼
Python 제어
   │
   ▼
Arduino/OpenRB-150
   │
   ▼
로봇팔(4~6축)
   │
   ▼
그리퍼로 물체 집기
20. 핵심 체크리스트
✅ 로봇팔의 구조와 자유도(DOF)를 이해한다.
✅ CAD를 이용해 링크와 관절을 설계한다.
✅ 서보 또는 다이나믹셀 모터를 선택하고 조립한다.
✅ 전원과 배선을 안전하게 구성한다.
✅ Arduino와 Python으로 기본 동작을 제어한다.
✅ 순기구학(Forward Kinematics)과 역기구학(Inverse Kinematics)의 개념을 이해한다.
✅ ROS2 및 MoveIt 연동을 위한 기본 구조를 준비한다.
다음 단계 (11단계)

ROS2 기초

11단계에서는 이번에 제작한 로봇팔을 ROS2와 연결하여 다음 내용을 학습합니다.

ROS2 설치 및 워크스페이스 구성
Node, Topic, Service 기본 개념
robot_state_publisher와 joint_state_publisher
URDF 로봇 모델 표시
RViz에서 로봇 시각화
실제 로봇팔과 ROS2 통신
이후 MoveIt 기반 경로 계획 및 자동 제어를 위한 기반 구축

이 단계가 끝나면 직접 만든 로봇팔을 ROS2 생태계에서 제어할 준비가 완료됩니다.
