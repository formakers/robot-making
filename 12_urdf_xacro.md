제12단계. URDF와 Xacro (로봇 모델 만들기)

이 단계는 ROS 2와 MoveIt을 사용하는 모든 로봇 개발의 핵심 단계입니다.

앞에서 배운 11단계 ROS 2 기초에서는 노드(Node), 토픽(Topic), 서비스(Service), Launch 등을 배웠습니다.

이제는 ROS가 "이 로봇이 어떻게 생겼는지"를 이해하도록 만드는 단계입니다.

즉,

실제 로봇을 컴퓨터 안에 그대로 만들어 주는 과정

입니다.

이번 단계의 목표

이번 단계를 끝내면 여러분은

✅ 자신의 로봇 모델 제작

✅ RViz에서 3D 로봇 확인

✅ Joint 움직이기

✅ Robot State Publisher 이해

✅ TF 이해

✅ MoveIt에서 사용하는 robot_description 이해

까지 배우게 됩니다.

전체 흐름
실제 로봇

↓

치수 측정

↓

URDF 작성

↓

RViz 표시

↓

TF 생성

↓

MoveIt 연결

↓

Gazebo 연결

↓

실제 로봇 연결

즉

URDF는

ROS의 가장 중요한 시작점입니다.

URDF란?

URDF

Unified Robot Description Format

입니다.

쉽게 말하면

"로봇 설계도"

입니다.

예를 들어

로봇팔

Base

↓

Joint1

↓

Link1

↓

Joint2

↓

Link2

↓

Joint3

↓

Link3

이 관계를 XML로 작성합니다.

예)

<link name="base_link"/>

<link name="link1"/>

<joint name="joint1">

</joint>

이렇게 하나씩 연결합니다.

URDF가 하는 일

URDF는

1. 로봇 모양
길이

폭

높이

2. 무게
질량

관성

3. 관절
회전

슬라이드

고정

4. 좌표계
XYZ

Roll

Pitch

Yaw
5. 충돌 모델
Collision
6. 화면 표시
Visual

이 모든 정보를 저장합니다.

URDF 구조

기본 구조는

<robot>

    <link/>

    <joint/>

    <link/>

    <joint/>

</robot>

입니다.

Link란?

Link는

"하나의 물체"

입니다.

예를 들면

베이스

팔

손목

그리퍼

카메라


모두 Link입니다.

예)

<link name="base_link"/>
Joint란?

Joint는

Link와 Link를 연결합니다.

Base

↓

Joint

↓

Arm

예)

<joint

name="joint1"

type="revolute">

</joint>
Joint 종류
1. Fixed

고정

움직이지 않음
2. Revolute

회전

Servo Motor

가장 많이 사용합니다.

3. Continuous

무한 회전

바퀴
4. Prismatic

직선 운동

실린더

리니어 모터
5. Floating

6축 자유도

6. Planar

평면 이동

Visual

RViz에 보이는 모델입니다.

예)

<visual>

<geometry>

<box size="0.1 0.1 0.1"/>

</geometry>

</visual>
Collision

충돌 계산용입니다.

MoveIt에서 매우 중요합니다.

<collision>

<geometry>

<cylinder/>

</geometry>

</collision>
Inertial

질량입니다.

<inertial>

<mass value="1.0"/>

</inertial>
Origin

위치입니다.

XYZ

RPY

예)

<origin

xyz="0 0 0.1"

rpy="0 0 0"/>
XYZ
X 앞뒤

Y 좌우

Z 위아래
RPY
Roll

Pitch

Yaw

입니다.

Link 연결 구조
base_link

↓

joint1

↓

link1

↓

joint2

↓

link2

↓

joint3

↓

link3

↓

gripper

이 구조를

Tree라고 합니다.

Robot State Publisher

URDF를 읽어서

TF를 생성합니다.

URDF

↓

Robot State Publisher

↓

TF

↓

RViz
TF란?

TF는

좌표계입니다.

예)

Base

↓

Shoulder

↓

Elbow

↓

Wrist

↓

Camera

각 좌표가 계속 계산됩니다.

MoveIt도 TF를 사용합니다.

RViz에서 보이는 과정
URDF

↓

robot_state_publisher

↓

joint_state_publisher

↓

RViz

Joint를 움직이면

로봇이 움직입니다.

Xacro란?

URDF는 XML이라 반복이 많습니다.

예)

<link>

...

</link>

<link>

...

</link>

<link>

...

</link>

수백 줄이 반복됩니다.

그래서 나온 것이

Xacro

입니다.

Xacro 장점
변수 사용
<xacro:property

name="length"

value="0.3"/>
계산 가능
${length*2}
함수(Macro)
<xacro:macro>

</xacro:macro>
반복 제거

같은 Link를 여러 번 생성할 수 있습니다.

Macro 예제
<xacro:macro name="Link" params="name">

<link name="${name}">

...

</link>

</xacro:macro>

사용

<xacro:Link name="link1"/>

<xacro:Link name="link2"/>
실제 OpenManipulator-X(OMX) 구조

현재 여러분이 사용 중인 OMX도 내부적으로 Xacro와 URDF를 사용합니다.

base_link
   │
 joint1
   │
 link1
   │
 joint2
   │
 link2
   │
 joint3
   │
 link3
   │
 joint4
   │
 link4
   │
 gripper

이 모델을 기반으로 robot_state_publisher가 /robot_description을 발행하고, RViz와 MoveIt이 이를 읽어 로봇을 표시하고 경로를 계획합니다.

URDF → MoveIt 연결
Xacro

↓

URDF

↓

robot_description

↓

SRDF

↓

MoveIt

↓

Planning

↓

Trajectory

↓

실제 로봇
실습 프로젝트

이번 단계에서는 다음과 같은 실습을 진행하는 것을 권장합니다.

간단한 1축 로봇팔 URDF 만들기
2축 로봇팔로 확장하기
RViz에서 Joint 움직여 보기
Robot State Publisher와 Joint State Publisher 실행하기
Xacro로 코드 리팩터링하기
OpenManipulator-X의 URDF/Xacro 파일 구조 분석하기
MoveIt Setup Assistant로 SRDF 생성하기
Gazebo(또는 시뮬레이터)에서 동작 확인하기
실제 OMX 하드웨어와 동일한 모델 연결하기
이번 단계를 마치면 가능한 것

이 단계를 완전히 이해하면 다음과 같은 작업을 직접 할 수 있습니다.

자신의 로봇을 URDF/Xacro로 모델링하기
RViz에서 3D 로봇 시각화하기
TF 좌표계를 이해하고 디버깅하기
MoveIt과 연동하여 경로 계획하기
Gazebo 시뮬레이션과 실제 하드웨어를 동일한 모델로 운용하기
이후 13단계(MoveIt), 14단계(MoveIt Python API), 15단계(Depth Camera), 16단계(AI Pick & Place) 의 기반을 완성하기
다음 단계 (13단계)

다음 단계에서는 MoveIt 기초를 배우며 다음 내용을 다룹니다.

MoveIt의 전체 구조
Motion Planning(경로 계획)
IK(역기구학)와 FK(순기구학)
Planning Scene과 Collision Object
OMPL Planner
실제 OpenManipulator-X와 MoveIt 연동
RViz에서 목표 자세 지정 및 경로 실행

12단계를 충분히 이해하면 이후의 AI 로봇 제어와 자율 작업 구현이 훨씬 수월해집니다.
