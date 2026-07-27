11단계 : ROS 2 기초 (Robot Operating System 2)
학습 목표

이번 단계에서는

ROS2가 무엇인지 이해하기
ROS2 설치 구조 이해
Node
Topic
Service
Action
Parameter
Launch
Package
Workspace
실제 로봇 연결

까지 배우게 됩니다.

왜 ROS2를 배우는가?

아두이노는

센서를 읽고
모터를 움직이는 프로그램

입니다.

하지만

AI 로봇은

카메라
라이다
Depth Camera
AI
모터
GUI
네트워크

등 수십 개 프로그램이 동시에 실행됩니다.

이것들을 하나로 연결하는 운영체제가

바로

ROS2

입니다.

ROS2의 역할

예를 들어

AI 로봇팔이라면

카메라
   │
   ▼
YOLO
   │
   ▼
물체 좌표
   │
   ▼
MoveIt
   │
   ▼
경로 계산
   │
   ▼
모터 제어

모든 프로그램이

ROS2를 통해 데이터를 주고받습니다.

ROS2 구조
        ROS2

   Node
   Node
   Node
   Node

      │

   Topic

      │

 Service

      │

 Action
가장 중요한 개념

ROS2는

프로그램 하나를

Node

라고 부릅니다.

예)

카메라 Node

YOLO Node

MoveIt Node

Motor Node

Depth Camera Node

Navigation Node

전부 Node입니다.

Node란?

하나의 기능을 수행하는 프로그램입니다.

예)

Camera Node

↓

이미지 발행

↓

YOLO Node

↓

컵 좌표 발행

↓

MoveIt Node

↓

경로 생성

↓

Robot Driver

↓

모터 회전
Topic이란?

Node끼리 데이터를 보내는 통신입니다.

예)

카메라

↓

/camera/image

↓

YOLO

또는

YOLO

↓

/cup_position

↓

MoveIt
Topic 예시
/joint_states

/camera/image

/tf

/tf_static

/scan

/cmd_vel
Publisher

데이터를 보내는 Node

Camera

↓

Image Publish
Subscriber

데이터를 받는 Node

YOLO

↓

Image Subscribe
예시
Publisher

↓

Hello

↓

Topic

↓

Subscriber

Publisher가

Hello

를 보내면

Subscriber가

Hello

를 받습니다.

Service

요청하면

한 번만 응답합니다.

예)

LED 켜기

↓

Service

↓

성공
Action

시간이 오래 걸리는 작업

예)

Move Robot

↓

10초 이동

↓

현재 진행률 표시

↓

완료

MoveIt이 Action을 많이 사용합니다.

Parameter

Node 설정값입니다.

예)

Camera Resolution

640x480

1280x720

실행 중에도 변경 가능합니다.

Launch

Node를 여러 개 동시에 실행합니다.

예)

Camera

YOLO

MoveIt

RViz

Robot Driver

한 번에 실행됩니다.

Workspace

ROS2 프로젝트 폴더입니다.

예)

~/robot_ws

안에는

src

build

install

log

가 있습니다.

Package

ROS2 프로그램 하나입니다.

예)

camera_pkg

moveit_pkg

robot_driver

navigation_pkg
실제 AI 로봇 구조
USB Camera

↓

Camera Node

↓

YOLO Node

↓

Object Position

↓

MoveIt

↓

Trajectory

↓

Controller

↓

Robot Driver

↓

OMX Robot

여러분이 앞으로 만들 로봇도

거의 동일한 구조입니다.

지금까지 배운 OMX도 ROS2 구조입니다
OpenRB

↓

ros2_control

↓

Controller Manager

↓

MoveIt

↓

RViz

↓

Python API

이미 지금까지 학습한 내용이 ROS2 위에서 동작하고 있습니다.

자주 사용하는 ROS2 명령어
# 현재 실행 중인 노드 확인
ros2 node list

# 토픽 확인
ros2 topic list

# 토픽 내용 확인
ros2 topic echo /joint_states

# 서비스 확인
ros2 service list

# 액션 확인
ros2 action list

# 파라미터 확인
ros2 param list

# 패키지 실행
ros2 run 패키지명 실행파일

# 런치파일 실행
ros2 launch 패키지명 launch.py
실습 1 : 현재 노드 확인

터미널에서

source /opt/ros/jazzy/setup.bash

ros2 node list
실습 2 : 토픽 확인
ros2 topic list
실습 3 : joint_states 보기
ros2 topic echo /joint_states
실습 4 : 현재 Parameter 보기
ros2 param list
실습 5 : Service 보기
ros2 service list
실습 6 : Action 보기
ros2 action list
이번 단계 핵심 요약
Node : 하나의 기능을 수행하는 프로그램
Topic : 노드 간 데이터 통신
Publisher / Subscriber : 데이터 송신 / 수신
Service : 요청 후 한 번 응답
Action : 오래 걸리는 작업과 진행 상태 관리
Parameter : 실행 중 변경 가능한 설정값
Launch : 여러 노드를 동시에 실행
Workspace : ROS2 프로젝트 작업 공간
Package : ROS2 프로그램 단위
다음 단계 (12단계)

다음 단계에서는 URDF와 Xacro를 배우며,

로봇 모델 작성
링크(Link)와 조인트(Joint)
Xacro를 이용한 재사용 가능한 URDF 작성
RViz에서 로봇 모델 시각화
MoveIt과 연동 가능한 로봇 모델 제작

까지 단계별로 실습해 보겠습니다.
