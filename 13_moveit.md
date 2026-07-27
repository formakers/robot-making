AI 로봇 제작 마스터 클래스
13단계 — MoveIt 2 로봇팔 모션 플래닝

13단계에서는 12단계에서 만든 URDF·Xacro 로봇 모델을 MoveIt 2에 연결하여, 로봇팔이 목표 위치까지 안전하게 움직이는 경로를 자동으로 계산하도록 만듭니다.

MoveIt 2는 ROS 2용 로봇 조작 프레임워크입니다. 로봇팔의 역기구학, 충돌 검사, 경로 계획, 궤적 생성, 컨트롤러 연결 등을 담당합니다. 현재 사용 중인 Ubuntu 24.04 + ROS 2 Jazzy에서도 공식 바이너리로 설치할 수 있습니다.

1. MoveIt 2란 무엇인가?

로봇팔 끝부분을 특정 위치로 이동시키려면 각 관절을 몇 도씩 움직여야 하는지 계산해야 합니다.

예를 들어 다음과 같은 명령을 내린다고 생각해 보겠습니다.

로봇팔 끝을 앞으로 20cm, 위로 10cm 이동시켜라.

사용자가 직접 joint1, joint2, joint3의 각도를 계산하기는 어렵습니다. MoveIt은 다음 과정을 자동으로 처리합니다.

목표 위치 지정
     ↓
역기구학 계산
     ↓
가능한 관절 각도 검색
     ↓
자기 충돌 및 장애물 검사
     ↓
안전한 이동 경로 생성
     ↓
관절별 시간 궤적 생성
     ↓
ros2_control 컨트롤러로 전달
     ↓
실제 로봇팔 동작

MoveIt을 사용하면 다음 작업을 구현할 수 있습니다.

RViz에서 로봇팔 목표 위치 지정
관절 각도 목표 설정
로봇팔 끝단 위치와 자세 설정
자기 충돌 방지
주변 장애물 회피
직선 경로 생성
로봇팔 궤적 계획
그리퍼 열기와 닫기
Pick & Place
카메라 기반 물체 집기
실제 로봇팔 제어
2. 13단계의 학습 목표

이번 단계의 최종 목표는 다음과 같습니다.

URDF/Xacro 로봇 모델
        ↓
MoveIt 설정 패키지 생성
        ↓
RViz MotionPlanning 실행
        ↓
목표 자세 지정
        ↓
Plan 실행
        ↓
계획된 경로 확인
        ↓
Execute 실행
        ↓
가상 또는 실제 로봇팔 동작

구체적으로 다음 내용을 학습합니다.

MoveIt 2 구조 이해
MoveIt Setup Assistant 사용
SRDF 설정
Planning Group 설정
End Effector 설정
Kinematics 설정
OMPL 경로 계획 설정
RViz에서 Plan과 Execute 실행
충돌 검사
ros2_control 연결
실제 OMX-F 로봇팔 연결 구조 이해
3. MoveIt의 핵심 구성 요소
3.1 URDF와 Xacro

URDF는 로봇의 물리적인 구조를 정의합니다.

링크
관절
크기
모양
질량
관성
충돌 모델

예를 들면 다음과 같습니다.

<joint name="joint1" type="revolute">
    <parent link="base_link"/>
    <child link="link1"/>
    <origin xyz="0 0 0.05" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-3.14"
           upper="3.14"
           effort="10"
           velocity="1.0"/>
</joint>

MoveIt은 URDF에 정의된 다음 정보를 사용합니다.

관절의 회전축
관절 제한값
링크 연결 관계
충돌 형상
로봇의 기준 좌표계
엔드이펙터 위치

URDF가 잘못되어 있으면 MoveIt도 정상적으로 동작하지 않습니다.

3.2 SRDF

SRDF는 Semantic Robot Description Format의 약자입니다.

URDF가 로봇의 물리 구조를 정의한다면 SRDF는 MoveIt이 로봇을 어떻게 사용할지 정의합니다.

SRDF에는 다음 내용이 들어갑니다.

로봇팔 그룹
그리퍼 그룹
엔드이펙터
기본 자세
자기 충돌 제외 목록
가상 관절

예를 들어 OMX-F의 로봇팔 그룹은 다음처럼 구성될 수 있습니다.

<group name="arm">
    <joint name="joint1"/>
    <joint name="joint2"/>
    <joint name="joint3"/>
    <joint name="joint4"/>
    <joint name="joint5"/>
</group>

그리퍼 그룹은 다음과 같이 구성할 수 있습니다.

<group name="gripper">
    <joint name="gripper_joint_1"/>
    <joint name="gripper_joint_2"/>
</group>

MoveIt Setup Assistant의 주요 역할도 바로 이 SRDF와 관련 설정 파일을 생성하는 것입니다.

3.3 Planning Group

Planning Group은 MoveIt이 함께 움직임을 계산할 관절 묶음입니다.

예를 들어 다음과 같이 나눌 수 있습니다.

arm 그룹
 ├── joint1
 ├── joint2
 ├── joint3
 ├── joint4
 └── joint5

gripper 그룹
 ├── gripper_joint_1
 └── gripper_joint_2

MoveIt에서 arm 그룹에 목표를 주면 로봇팔 관절 전체의 움직임을 계획합니다.

gripper 그룹에 목표를 주면 그리퍼 관절만 움직입니다.

그룹 이름은 나중에 Python 또는 C++ 프로그램에서도 사용합니다.

planning_group = "arm"

따라서 SRDF의 그룹 이름과 프로그램의 그룹 이름이 정확히 같아야 합니다.

3.4 End Effector

End Effector는 로봇팔의 끝부분입니다.

일반적으로 다음 장치가 엔드이펙터가 될 수 있습니다.

2지 그리퍼
흡착 그리퍼
용접 토치
카메라
드릴
전동 드라이버
로봇 손

OMX-F에서는 일반적으로 다음 구조가 됩니다.

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
joint4
   ↓
link4
   ↓
joint5
   ↓
end_effector_link
   ↓
gripper

MoveIt에서 목표 위치를 지정하면 기본적으로 엔드이펙터 링크가 그 위치에 도달하도록 관절 각도를 계산합니다.

3.5 역기구학

역기구학은 영어로 Inverse Kinematics, 줄여서 IK라고 합니다.

엔드이펙터의 위치와 자세가 주어졌을 때 관절 각도를 계산하는 과정입니다.

입력

X = 0.25m
Y = 0.10m
Z = 0.30m

Roll  = 0
Pitch = 1.57
Yaw   = 0

MoveIt은 위 목표를 만족하는 관절 값을 계산합니다.

joint1 = 0.38 rad
joint2 = -0.72 rad
joint3 = 0.91 rad
joint4 = 0.43 rad
joint5 = 0.00 rad

같은 위치에 도달하는 관절 조합이 여러 개일 수도 있습니다. MoveIt은 관절 제한, 충돌 여부와 현재 위치 등을 고려하여 가능한 해를 선택합니다.

3.6 순기구학

순기구학은 Forward Kinematics, 줄여서 FK라고 합니다.

관절 각도를 알고 있을 때 엔드이펙터 위치를 계산합니다.

관절 각도
joint1 = 0.2
joint2 = -0.5
joint3 = 0.8
joint4 = 0.3

        ↓

엔드이펙터 위치
X = 0.24m
Y = 0.05m
Z = 0.31m

정리하면 다음과 같습니다.

순기구학 FK
관절 각도 → 로봇팔 끝 위치

역기구학 IK
로봇팔 끝 위치 → 관절 각도
4. MoveIt의 모션 플래닝

MoveIt의 핵심 기능은 모션 플래닝입니다.

모션 플래닝은 단순히 시작점과 도착점을 연결하는 것이 아닙니다.

다음 조건을 모두 확인해야 합니다.

관절 제한을 초과하지 않는가?
로봇팔이 자기 몸과 충돌하지 않는가?
주변 장애물과 충돌하지 않는가?
목표 위치에 도달할 수 있는가?
컨트롤러가 실행할 수 있는 경로인가?
이동 속도와 가속도가 적절한가?

MoveIt은 로봇 상태와 주변 환경을 Planning Scene에 저장합니다. Planning Scene에는 로봇의 현재 상태와 주변 장애물 정보가 포함되며, /joint_states와 환경 정보 등을 이용해 갱신됩니다.

4.1 OMPL

MoveIt에서 널리 사용하는 경로 계획 라이브러리가 OMPL입니다.

OMPL에는 여러 플래너가 있습니다.

RRTConnect
RRTstar
PRM
PRMstar
EST
KPIECE
BKPIECE

초보 실습에서는 일반적으로 RRTConnect를 많이 사용합니다.

현재 자세
   ↓
랜덤하게 이동 가능 공간 탐색
   ↓
목표 자세와 연결 가능한 경로 발견
   ↓
충돌 없는 궤적 생성

MoveIt이 다음과 같은 메시지를 출력하는 경우가 있습니다.

Planner configuration 'arm' will use planner 'geometric::RRTConnect'

이는 arm Planning Group의 경로를 RRTConnect로 계산한다는 뜻입니다.

5. MoveIt의 전체 구조

MoveIt 시스템은 대략 다음과 같이 연결됩니다.

RViz MotionPlanning
        │
        ▼
    move_group
        │
        ├── Robot Model
        ├── Planning Scene
        ├── Kinematics Plugin
        ├── OMPL Planner
        ├── Collision Checker
        └── Trajectory Processing
                │
                ▼
    MoveIt Controller Manager
                │
                ▼
       ros2_control Controller
                │
                ▼
     Gazebo 또는 실제 로봇팔
5.1 move_group 노드

move_group은 MoveIt의 중심 노드입니다.

다음 기능을 담당합니다.

로봇 모델 읽기
현재 관절 상태 확인
목표 상태 수신
역기구학 계산
경로 계획
충돌 검사
궤적 생성
컨트롤러로 실행 명령 전달

실행 중 다음 명령으로 확인할 수 있습니다.

ros2 node list

정상 실행되면 다음과 같은 노드가 나타날 수 있습니다.

/move_group
/robot_state_publisher
/rviz2
/controller_manager
/joint_state_broadcaster
5.2 robot_state_publisher

robot_state_publisher는 URDF와 관절 상태를 이용하여 TF 좌표계를 발행합니다.

/joint_states
      ↓
robot_state_publisher
      ↓
/tf
/tf_static

MoveIt은 이 좌표계를 이용하여 각 링크 위치를 파악합니다.

확인 명령은 다음과 같습니다.

ros2 topic echo /joint_states --once
ros2 topic list | grep tf
ros2 run tf2_tools view_frames
5.3 Joint State Broadcaster

joint_state_broadcaster는 현재 관절 값을 /joint_states 토픽으로 발행합니다.

예시는 다음과 같습니다.

name:
- joint1
- joint2
- joint3
- joint4
- joint5

position:
- 0.0
- -0.5
- 0.7
- 0.3
- 0.0

MoveIt은 이 값을 로봇의 현재 시작 상태로 사용합니다.

따라서 /joint_states가 나오지 않으면 다음 문제가 발생합니다.

Failed to fetch current robot state
Current state is not available
Waiting for initial joint states
5.4 Joint Trajectory Controller

MoveIt은 최종적으로 관절 궤적을 생성합니다.

예를 들면 다음과 같습니다.

시간 0.0초
joint1 = 0.0
joint2 = -0.4

시간 1.0초
joint1 = 0.2
joint2 = -0.6

시간 2.0초
joint1 = 0.5
joint2 = -0.8

이 궤적은 일반적으로 JointTrajectoryController로 전달됩니다.

OMX-F에서는 다음과 같은 컨트롤러 이름을 사용할 수 있습니다.

arm_controller
gripper_controller
joint_state_broadcaster

확인 명령은 다음과 같습니다.

ros2 control list_controllers

정상적인 예:

joint_state_broadcaster  joint_state_broadcaster/JointStateBroadcaster  active
arm_controller           joint_trajectory_controller/JointTrajectoryController active
gripper_controller       position_controllers/GripperActionController active
6. MoveIt 2 설치

현재 환경이 Ubuntu 24.04와 ROS 2 Jazzy라면 다음과 같이 설치할 수 있습니다.

sudo apt update
sudo apt install ros-jazzy-moveit

Setup Assistant도 설치합니다.

sudo apt install ros-jazzy-moveit-setup-assistant

설치 확인:

source /opt/ros/jazzy/setup.bash
ros2 pkg list | grep moveit

다음 패키지들이 검색되면 정상입니다.

moveit_core
moveit_ros_move_group
moveit_ros_planning
moveit_ros_planning_interface
moveit_setup_assistant
moveit_planners_ompl
7. MoveIt 공식 예제 실행

처음에는 자신의 로봇보다 공식 예제로 MoveIt의 작동 원리를 익히는 것이 좋습니다.

MoveIt 공식 튜토리얼에서는 RViz의 MotionPlanning 플러그인으로 시작 상태와 목표 상태를 지정하고 경로를 계획하는 방법을 제공합니다. RViz에서는 Planning Scene 구성, 목표 상태 설정, 플래너 시험과 계획 경로 시각화가 가능합니다.

터미널 1:

source /opt/ros/jazzy/setup.bash

설치된 데모 패키지에 따라 예제 실행 명령은 달라질 수 있습니다. 현재 설치된 MoveIt 튜토리얼 패키지를 먼저 확인합니다.

ros2 pkg list | grep moveit_resources

Kinova 예제가 설치되어 있다면 관련 데모 launch 파일을 확인합니다.

ros2 pkg prefix moveit_resources_panda_moveit_config

Panda 데모 패키지가 설치된 환경에서는 다음 형태로 실행합니다.

ros2 launch moveit_resources_panda_moveit_config demo.launch.py

실행 후 RViz에 로봇이 나타납니다.

8. RViz MotionPlanning 화면 이해

RViz에서 중요한 부분은 MotionPlanning 패널입니다.

Planning Group

제어할 관절 그룹을 선택합니다.

Planning Group: arm

또는 예제 로봇에서는 다음과 같이 표시될 수 있습니다.

Planning Group: panda_arm
Start State

로봇이 움직이기 시작할 자세입니다.

일반적으로 다음 값을 선택합니다.

Start State:
Current

즉, 실제 또는 시뮬레이션 로봇의 현재 관절 위치를 출발점으로 사용합니다.

Goal State

로봇이 도착할 목표 자세입니다.

목표 자세는 다음 방법으로 지정할 수 있습니다.

인터랙티브 마커 이동
미리 저장된 자세 선택
관절 값 직접 입력
프로그램에서 Pose 전송

RViz에서 주황색 로봇은 일반적으로 목표 상태를 나타냅니다. 녹색은 시작 상태, 계획 경로는 궤적 형태로 표시될 수 있습니다.

Plan 버튼

Plan 버튼을 누르면 MoveIt이 경로만 계산합니다.

현재 상태
     ↓
목표 상태
     ↓
충돌 검사
     ↓
경로 계산
     ↓
RViz에서 애니메이션 표시

이때 실제 로봇은 움직이지 않습니다.

Execute 버튼

Execute 버튼을 누르면 이미 계산한 경로를 컨트롤러로 전달합니다.

계획된 RobotTrajectory
        ↓
MoveIt Controller Manager
        ↓
FollowJointTrajectory Action
        ↓
arm_controller
        ↓
모터 구동
Plan & Execute 버튼

계획과 실행을 한 번에 수행합니다.

실제 로봇을 처음 테스트할 때는 바로 Plan & Execute를 누르기보다 다음 순서를 권장합니다.

1. 목표 자세 지정
2. Plan
3. 경로 확인
4. 충돌 여부 확인
5. 속도 낮추기
6. Execute
9. RViz 기본 실습
실습 1: 목표 자세 이동
RViz에서 Planning Group을 arm으로 선택합니다.
Interact 버튼을 활성화합니다.
엔드이펙터 주변의 화살표와 원형 마커를 움직입니다.
도달 가능한 가까운 위치로 이동합니다.
Plan을 누릅니다.
계획 경로를 확인합니다.
시뮬레이션이라면 Execute를 누릅니다.

마커의 의미는 다음과 같습니다.

빨간 화살표: X축 이동
초록 화살표: Y축 이동
파란 화살표: Z축 이동

빨간 원: X축 회전
초록 원: Y축 회전
파란 원: Z축 회전
실습 2: 관절 값으로 이동

MotionPlanning 패널의 Joints 항목에서 관절 값을 변경합니다.

예:

joint1 = 0.30
joint2 = -0.50
joint3 = 0.70
joint4 = 0.20
joint5 = 0.00

그 후 다음 순서로 실행합니다.

Plan
   ↓
경로 확인
   ↓
Execute
실습 3: 저장된 자세로 이동

MoveIt Setup Assistant에서 저장한 자세를 Named State라고 합니다.

예:

home
ready
sleep
open
close

SRDF에는 다음처럼 저장됩니다.

<group_state name="home" group="arm">
    <joint name="joint1" value="0"/>
    <joint name="joint2" value="-0.5"/>
    <joint name="joint3" value="0.7"/>
    <joint name="joint4" value="0.3"/>
    <joint name="joint5" value="0"/>
</group_state>

RViz에서 home을 선택하고 계획하면 지정된 관절 자세로 이동합니다.

10. MoveIt Setup Assistant 사용

자신이 만든 로봇 모델을 MoveIt에 연결하려면 MoveIt 설정 패키지를 만들어야 합니다.

실행:

source /opt/ros/jazzy/setup.bash
ros2 launch moveit_setup_assistant setup_assistant.launch.py

GUI가 열리면 다음 과정을 진행합니다.

10.1 Create New MoveIt Configuration Package

새 설정 패키지를 만들 때 선택합니다.

Create New MoveIt Configuration Package
10.2 URDF 또는 Xacro 불러오기

예를 들어 로봇 패키지가 다음 위치에 있다고 가정합니다.

~/robot_ws/src/my_robot_description/urdf/my_robot.urdf.xacro

이 파일을 선택하여 불러옵니다.

불러온 뒤 로봇이 화면에 정상적으로 나타나는지 확인합니다.

확인할 사항:

링크가 분리되어 있지 않은가?
관절 회전축이 올바른가?
링크가 비정상적으로 크지 않은가?
로봇 원점이 올바른가?
그리퍼가 정상 위치에 있는가?
10.3 Self-Collisions 생성

Self-Collisions 메뉴에서 다음 버튼을 누릅니다.

Generate Collision Matrix

MoveIt은 서로 충돌할 필요가 없는 링크 쌍을 자동으로 분석합니다.

예를 들어 항상 붙어 있는 인접 링크는 충돌 검사를 생략할 수 있습니다.

base_link ↔ link1
link1 ↔ link2
link2 ↔ link3

이 설정은 계산량을 줄여 줍니다.

10.4 Virtual Joints

고정형 로봇팔은 일반적으로 다음처럼 설정합니다.

Virtual Joint Name: fixed_base
Child Link: base_link
Parent Frame Name: world
Joint Type: fixed

의미는 다음과 같습니다.

world 좌표계에 base_link가 고정되어 있다.

이동형 로봇에 장착된 로봇팔이라면 planar 또는 floating 구조가 필요할 수 있지만, 테이블 위 고정형 OMX-F는 보통 fixed를 사용합니다.

10.5 Planning Groups

로봇팔 그룹을 생성합니다.

Group Name: arm

Kinematic Solver는 설치된 플러그인에 따라 다음과 같이 선택할 수 있습니다.

kdl_kinematics_plugin/KDLKinematicsPlugin

그다음 관절을 추가합니다.

joint1
joint2
joint3
joint4
joint5

그리퍼 그룹도 생성합니다.

Group Name: gripper
gripper_joint_1
gripper_joint_2

Mimic Joint 구조라면 실제 구동 관절 하나만 그룹에 넣고 나머지는 URDF의 mimic 관계로 움직이도록 구성할 수도 있습니다.

10.6 Robot Poses

자주 사용하는 자세를 등록합니다.

home 자세
joint1 = 0.0
joint2 = -0.5
joint3 = 0.7
joint4 = 0.3
joint5 = 0.0
ready 자세
joint1 = 0.0
joint2 = -0.8
joint3 = 1.0
joint4 = 0.4
joint5 = 0.0

이 값은 실제 로봇의 안전한 관절 범위 안에서 정해야 합니다.

10.7 End Effectors

그리퍼를 엔드이펙터로 등록합니다.

예:

End Effector Name: gripper
End Effector Group: gripper
Parent Link: end_effector_link
Parent Group: arm

로봇 모델에 따라 Parent Link 이름은 다음과 다를 수 있습니다.

tool0
ee_link
end_effector_link
gripper_link
10.8 Passive Joints

모터로 직접 제어하지 않는 관절이 있다면 Passive Joint로 지정합니다.

예:

스프링으로 움직이는 관절
기구적으로 따라 움직이는 관절
직접 구동되지 않는 종속 관절

일반적인 모터 구동 관절은 Passive Joint로 설정하면 안 됩니다.

10.9 ROS 2 Controllers

MoveIt에서 사용할 컨트롤러를 설정합니다.

로봇팔 컨트롤러 예:

Controller Name: arm_controller
Controller Type:
joint_trajectory_controller/JointTrajectoryController

관절:

joint1
joint2
joint3
joint4
joint5

그리퍼 컨트롤러 예:

Controller Name: gripper_controller
Controller Type:
position_controllers/GripperActionController
10.10 설정 패키지 생성

패키지 이름의 예:

my_robot_moveit_config

저장 위치:

~/robot_ws/src/my_robot_moveit_config

Generate Package를 누르면 MoveIt 설정 패키지가 생성됩니다.

11. 생성되는 주요 파일

MoveIt 설정 패키지는 대략 다음 구조를 가집니다.

my_robot_moveit_config/
├── CMakeLists.txt
├── package.xml
├── config/
│   ├── my_robot.srdf
│   ├── kinematics.yaml
│   ├── joint_limits.yaml
│   ├── moveit_controllers.yaml
│   ├── ros2_controllers.yaml
│   ├── ompl_planning.yaml
│   ├── initial_positions.yaml
│   └── pilz_cartesian_limits.yaml
├── launch/
│   ├── demo.launch.py
│   ├── move_group.launch.py
│   ├── moveit_rviz.launch.py
│   ├── rsp.launch.py
│   └── setup_assistant.launch.py
└── .setup_assistant
11.1 kinematics.yaml

역기구학 플러그인을 설정합니다.

arm:
  kinematics_solver: kdl_kinematics_plugin/KDLKinematicsPlugin
  kinematics_solver_search_resolution: 0.005
  kinematics_solver_timeout: 0.05

항목 의미:

kinematics_solver
역기구학 계산 플러그인

kinematics_solver_search_resolution
해를 탐색하는 간격

kinematics_solver_timeout
역기구학 계산 제한 시간
11.2 joint_limits.yaml

관절별 속도와 가속도 제한을 설정합니다.

joint_limits:
  joint1:
    has_velocity_limits: true
    max_velocity: 0.5
    has_acceleration_limits: true
    max_acceleration: 0.5

실제 로봇을 처음 움직일 때는 속도를 낮게 설정하는 것이 좋습니다.

예:

max_velocity: 0.2
max_acceleration: 0.2

URDF의 관절 제한과 YAML 설정이 서로 맞아야 합니다.

11.3 ompl_planning.yaml

OMPL 플래너를 설정합니다.

예:

planner_configs:
  RRTConnectkConfigDefault:
    type: geometric::RRTConnect
    range: 0.0

Planning Group에 플래너를 연결합니다.

arm:
  planner_configs:
    - RRTConnectkConfigDefault
11.4 moveit_controllers.yaml

MoveIt이 궤적을 어느 컨트롤러로 보낼지 정의합니다.

moveit_controller_manager:
  moveit_simple_controller_manager:
    controller_names:
      - arm_controller
      - gripper_controller

    arm_controller:
      type: FollowJointTrajectory
      action_ns: follow_joint_trajectory
      default: true
      joints:
        - joint1
        - joint2
        - joint3
        - joint4
        - joint5

    gripper_controller:
      type: GripperCommand
      action_ns: gripper_cmd
      default: true
      joints:
        - gripper_joint_1

여기 있는 컨트롤러 이름과 ros2_control의 컨트롤러 이름이 반드시 같아야 합니다.

12. 패키지 빌드

워크스페이스로 이동합니다.

cd ~/robot_ws

의존성을 설치합니다.

rosdep install --from-paths src --ignore-src -r -y

빌드합니다.

colcon build --symlink-install

환경을 적용합니다.

source /opt/ros/jazzy/setup.bash
source ~/robot_ws/install/setup.bash

패키지 확인:

ros2 pkg list | grep my_robot_moveit_config
13. Demo 실행

생성된 MoveIt 설정 패키지를 실행합니다.

source /opt/ros/jazzy/setup.bash
source ~/robot_ws/install/setup.bash
ros2 launch my_robot_moveit_config demo.launch.py

정상 실행되면 다음 구성 요소가 실행됩니다.

robot_state_publisher
move_group
RViz
joint_state_broadcaster 또는 mock 상태
MoveIt Controller Manager

RViz에서 다음 순서로 테스트합니다.

Planning Group 선택
→ 목표 마커 이동
→ Plan
→ 궤적 확인
→ Execute
14. Plan과 Execute의 차이
Plan
경로를 계산만 한다.
실제 모터는 움직이지 않는다.
Execute
이미 계산된 경로를 컨트롤러로 전달한다.
가상 로봇 또는 실제 로봇이 움직인다.
Plan & Execute
경로 계산 후 즉시 실행한다.

실제 하드웨어에서는 반드시 처음에 Plan만 눌러 경로를 먼저 확인해야 합니다.

15. 충돌 검사 실습

MoveIt은 Planning Scene에 장애물을 추가하고, 해당 장애물과 충돌하지 않는 경로를 계산할 수 있습니다.

예를 들어 로봇 앞에 상자를 추가합니다.

상자 위치
X = 0.25m
Y = 0.0m
Z = 0.15m

상자 크기
가로 = 0.15m
세로 = 0.30m
높이 = 0.30m

MoveIt은 다음 두 종류의 충돌을 검사합니다.

자기 충돌
로봇팔 링크 ↔ 로봇팔 링크

예:

link2가 base_link와 충돌
gripper가 link1과 충돌
환경 충돌
로봇팔 ↔ 테이블
로봇팔 ↔ 벽
로봇팔 ↔ 상자

Pick & Place를 구현할 때 테이블과 물체를 Planning Scene에 등록하는 것이 매우 중요합니다.

16. 실제 OMX-F와 연결되는 과정

사용자의 OMX-F 환경에서는 MoveIt과 실제 하드웨어가 대략 다음과 같이 연결됩니다.

RViz
  ↓
move_group
  ↓
arm_controller/follow_joint_trajectory
  ↓
ros2_control controller_manager
  ↓
OMXFSystem 하드웨어 인터페이스
  ↓
OpenRB-150
  ↓
Dynamixel ID 11~16
  ↓
실제 OMX-F 로봇팔

실제 하드웨어를 실행할 때는 포트를 확인합니다.

ls -l /dev/ttyACM*

안정적으로 구분하려면 다음도 확인합니다.

ls -l /dev/serial/by-id/

사용자가 확인한 OMX-F 포트가 /dev/ttyACM0이라면 다음 형태로 실행할 수 있습니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash
ros2 launch open_manipulator_bringup omx_f.launch.py \
  use_sim:=false \
  use_mock_hardware:=false \
  port_name:=/dev/ttyACM0

그다음 별도 터미널에서 MoveIt을 실행합니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash
ros2 launch open_manipulator_moveit_config move_group.launch.py \
  ros2_control_type:=omx_f

실제 launch 인자는 설치된 ROBOTIS 패키지 버전에 따라 다를 수 있으므로 다음 명령으로 확인합니다.

ros2 launch open_manipulator_bringup omx_f.launch.py --show-args
17. 실제 OMX-F 실행 전 점검
17.1 관절 상태 확인
ros2 topic echo /joint_states --once

다음과 같은 관절 이름이 나와야 합니다.

joint1
joint2
joint3
joint4
joint5
end_effector_joint

설정에 따라 실제 관절 이름은 다를 수 있습니다.

17.2 컨트롤러 확인
ros2 control list_controllers

다음 컨트롤러가 active 상태여야 합니다.

joint_state_broadcaster
arm_controller
gripper_controller
17.3 액션 확인

MoveIt은 일반적으로 FollowJointTrajectory 액션으로 명령을 보냅니다.

ros2 action list

예:

/arm_controller/follow_joint_trajectory
/gripper_controller/gripper_cmd

액션 정보 확인:

ros2 action info /arm_controller/follow_joint_trajectory
17.4 move_group 확인
ros2 node list | grep move_group

정상:

/move_group
17.5 robot_description 확인
ros2 topic info /robot_description -v

또는 파라미터 확인:

ros2 param get /robot_state_publisher robot_description

MoveIt과 controller_manager가 동일한 로봇 모델을 사용해야 합니다.

18. 실제 로봇에서 안전하게 테스트하는 방법

실제 OMX-F에서 처음 실행할 때는 다음 원칙을 지키는 것이 중요합니다.

1단계: 로봇 주변 정리
로봇팔 주변의 공구 제거
케이블 걸림 확인
비상 정지 준비
사람의 손을 로봇 가까이에 두지 않기
2단계: 속도 낮추기

RViz에서 다음 값을 낮게 설정합니다.

Velocity Scaling: 0.05
Acceleration Scaling: 0.05

즉 최대 속도의 약 5% 수준부터 시작합니다.

3단계: 작은 이동만 테스트

처음에는 큰 각도로 움직이지 않습니다.

joint1 현재 위치 + 0.05rad

또는 엔드이펙터를 몇 cm 정도만 이동합니다.

4단계: Plan 먼저 실행
Plan
→ 궤적 확인
→ Execute
5단계: 이상하면 즉시 정지

다른 터미널에서 컨트롤러를 중지할 수 있습니다.

ros2 control set_controller_state arm_controller inactive

하지만 실제 긴급 상황에서는 소프트웨어 명령보다 전원 차단이나 물리적 비상 정지가 더 확실합니다.

19. 자주 발생하는 오류
오류 1: Robot model not loaded
Robot model not loaded

원인:

robot_description 없음
URDF/Xacro 로딩 실패
robot_state_publisher 미실행
launch 파일에서 파라미터 전달 실패

확인:

ros2 node list
ros2 topic info /robot_description -v
ros2 param list /robot_state_publisher
오류 2: No planning library loaded
No planning library loaded

원인:

OMPL 설정 누락
ompl_planning.yaml 로딩 실패
MoveIt OMPL 패키지 미설치

설치:

sudo apt install ros-jazzy-moveit-planners-ompl
오류 3: Unable to sample any valid states
Unable to sample any valid states for goal tree

원인:

목표가 로봇 작업 범위 밖
목표 자세가 충돌 상태
역기구학 해 없음
엔드이펙터 설정 오류
관절 제한값 오류

해결:

목표를 현재 위치 가까이 옮기기
목표 방향을 조금 변경하기
URDF 관절 제한 확인
SRDF 그룹 확인
kinematics.yaml 확인
오류 4: Failed to fetch current robot state
Failed to fetch current robot state

원인:

/joint_states 미발행
관절 이름 불일치
타임스탬프 문제
하드웨어 노드 미실행

확인:

ros2 topic hz /joint_states
ros2 topic echo /joint_states --once

MoveIt의 관절 이름과 /joint_states의 관절 이름이 정확히 같아야 합니다.

오류 5: Controller failed during execution
Controller failed during execution

원인:

컨트롤러가 inactive
컨트롤러 이름 불일치
관절 순서 불일치
하드웨어 연결 실패
모터 Torque 비활성
시리얼 포트 오류

확인:

ros2 control list_controllers
ros2 action list
ros2 topic echo /joint_states
오류 6: Waiting for service controller_manager
waiting for service /controller_manager/list_controllers

원인:

ros2_control_node 미실행
하드웨어 플러그인 초기화 실패
robot_description 미전달
시리얼 포트 사용 중
컨트롤러 매니저 종료

확인:

ros2 node list | grep controller
ros2 service list | grep controller_manager
fuser -v /dev/ttyACM0
20. OMX-F에서 특히 중요한 설정 일치

MoveIt과 실제 하드웨어를 연결할 때 다음 네 가지가 모두 일치해야 합니다.

관절 이름
URDF
SRDF
ros2_controllers.yaml
moveit_controllers.yaml
/joint_states

예를 들어 한 파일에서 joint1, 다른 파일에서 joint_1로 되어 있으면 실행되지 않습니다.

컨트롤러 이름
ros2_controllers.yaml:
arm_controller

moveit_controllers.yaml:
arm_controller

두 이름이 같아야 합니다.

액션 이름

일반적인 로봇팔 컨트롤러 액션:

/arm_controller/follow_joint_trajectory

MoveIt 설정:

action_ns: follow_joint_trajectory
로봇 모델

다음 노드가 같은 URDF를 사용해야 합니다.

robot_state_publisher
controller_manager
move_group
RViz

서로 다른 로봇 모델을 사용하면 관절 수, 관절 이름 또는 링크 구조가 달라져 오류가 발생합니다.

21. 13단계 권장 실습 순서
실습 1

공식 MoveIt 데모를 실행하고 RViz에서 Plan을 수행합니다.

실습 2

인터랙티브 마커를 움직여 엔드이펙터 목표를 지정합니다.

실습 3

관절 값으로 목표 자세를 지정합니다.

실습 4

home, ready와 같은 Named Pose를 생성합니다.

실습 5

상자를 Planning Scene에 추가하고 장애물 회피를 확인합니다.

실습 6

자신의 URDF/Xacro를 Setup Assistant에 불러옵니다.

실습 7

arm과 gripper Planning Group을 생성합니다.

실습 8

생성한 moveit_config 패키지를 빌드합니다.

실습 9

Mock Hardware에서 Plan과 Execute를 시험합니다.

실습 10

Gazebo 또는 시뮬레이션 컨트롤러와 연결합니다.

실습 11

실제 OMX-F의 /joint_states를 MoveIt에 연결합니다.

실습 12

낮은 속도로 실제 로봇의 작은 움직임을 실행합니다.

22. 13단계 완료 기준

다음 항목을 수행할 수 있으면 13단계를 완료한 것입니다.

□ MoveIt의 역할을 설명할 수 있다.
□ URDF와 SRDF의 차이를 설명할 수 있다.
□ Planning Group을 만들 수 있다.
□ End Effector를 설정할 수 있다.
□ MoveIt Setup Assistant를 사용할 수 있다.
□ MoveIt Config 패키지를 생성할 수 있다.
□ RViz에서 목표 자세를 지정할 수 있다.
□ Plan과 Execute의 차이를 이해한다.
□ 충돌 없는 경로를 계획할 수 있다.
□ /joint_states를 확인할 수 있다.
□ ros2_control 컨트롤러 상태를 확인할 수 있다.
□ MoveIt과 실제 로봇의 연결 구조를 이해한다.
23. 핵심 요약
URDF/Xacro
로봇의 물리적인 구조

SRDF
MoveIt에서 사용하는 그룹과 의미 설정

Planning Group
함께 계획할 관절 묶음

End Effector
로봇팔 끝에서 작업하는 부분

Inverse Kinematics
목표 위치에서 관절 각도 계산

Planning Scene
로봇 상태와 주변 환경 저장

OMPL
충돌 없는 이동 경로 탐색

move_group
MoveIt의 중심 노드

ros2_control
계획된 관절 궤적을 실제 모터에 전달

13단계의 핵심 흐름은 다음 한 줄로 정리할 수 있습니다.

로봇 모델 작성 → MoveIt 설정 → 목표 지정 → 충돌 검사 → 경로 계획 → 컨트롤러 전달 → 로봇팔 동작

다음 14단계 MoveIt Python API에서는 RViz 버튼 대신 Python 프로그램으로 관절 목표, 위치 목표, 직선 경로, 장애물 추가와 실제 OMX-F 동작을 제어하게 됩니다.
