OMX-F 실제 로봇과 MoveIt 연동 전체 과정
1. 목표

이번 과정의 목표는 다음과 같습니다.

ROS 2 Jazzy 환경 불러오기
OMX-F 실제 로봇을 /dev/ttyACM1 포트로 연결
ros2_control을 통해 Dynamixel 모터 제어
MoveIt의 move_group 실행
RViz에서 로봇 모델과 현재 관절 상태 확인
MotionPlanning 패널에서 경로 계획
실제 로봇으로 계획된 동작 실행
이후 Planning Scene과 장애물 추가

전체 연결 구조는 다음과 같습니다.

OMX-F 실제 로봇
    │
    │ USB /dev/ttyACM1
    ▼
OMXFSystem 하드웨어 인터페이스
    │
    ▼
ros2_control
/controller_manager
    │
    ├─ joint_state_broadcaster
    ├─ arm_controller
    └─ gripper_controller
    │
    ▼
/joint_states
    │
    ▼
MoveIt move_group
    │
    ▼
RViz MotionPlanning
    │
    ▼
Plan → Execute
2. 사용 환경

현재 사용한 환경은 다음과 같습니다.

운영체제: Ubuntu 24.04
ROS 2: Jazzy
Workspace: ~/omx_moveit_ws
로봇: ROBOTIS OMX-F
포트: /dev/ttyACM1
MoveIt 패키지: open_manipulator_moveit_config
Bringup 패키지: open_manipulator_bringup
3. OMX-F 포트 확인

먼저 OMX-F가 연결된 USB 포트를 확인합니다.

ls -l /dev/ttyACM*

예:

/dev/ttyACM0
/dev/ttyACM1

현재 OMX-F가 /dev/ttyACM1에 연결되어 있으므로 이 포트를 사용했습니다.

포트 권한을 확인합니다.

ls -l /dev/ttyACM1

권한 문제가 생기지 않도록 임시 권한을 설정합니다.

sudo chmod 666 /dev/ttyACM1

이 설정은 재부팅하거나 USB를 다시 연결하면 초기화될 수 있습니다.

4. 포트 점유 확인

다른 프로그램이 OMX-F 포트를 사용하고 있으면 로봇이 연결되지 않습니다.

다음 명령으로 확인합니다.

sudo fuser -v /dev/ttyACM1

아무것도 출력되지 않으면 다른 프로그램이 포트를 사용하지 않는 상태입니다.

PID가 출력되면 다음 프로그램들을 확인합니다.

Arduino IDE
Arduino Serial Monitor
LeRobot
Python Dynamixel 프로그램
기존 ROS 2 노드

필요한 경우 포트를 점유한 프로세스를 종료합니다.

sudo fuser -k /dev/ttyACM1

다시 권한을 설정합니다.

sudo chmod 666 /dev/ttyACM1
5. ROS 2 환경 불러오기

새 터미널을 열 때마다 ROS 2와 Workspace 환경을 불러와야 합니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

Conda 환경이 활성화되어 있으면 ROS 2 Python 환경과 충돌할 수 있으므로 필요하면 비활성화합니다.

conda deactivate 2>/dev/null || true

권장 시작 명령은 다음과 같습니다.

conda deactivate 2>/dev/null || true

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash
6. MoveIt launch 파일 확인

사용 가능한 MoveIt launch 파일을 확인했습니다.

find ~/omx_moveit_ws/src \
  -type f \
  -name "*moveit.launch.py"

결과:

omy_f3m_moveit.launch.py
omy_3m_moveit.launch.py
open_manipulator_x_moveit.launch.py
omx_f_moveit.launch.py

OMX-F 모델에 맞는 launch 파일은 다음입니다.

omx_f_moveit.launch.py

이전에 다음과 같은 존재하지 않는 파일을 실행하면 오류가 발생했습니다.

moveit.launch.py
moveit_rviz.launch.py

따라서 반드시 실제로 존재하는 파일 이름을 사용해야 합니다.

7. 실제 OMX-F와 MoveIt 실행

OMX-F 실제 로봇과 MoveIt을 함께 실행한 최종 명령입니다.

conda deactivate 2>/dev/null || true

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

sudo chmod 666 /dev/ttyACM1

ros2 launch open_manipulator_moveit_config \
  omx_f_moveit.launch.py \
  use_sim:=false \
  use_mock_hardware:=false \
  port_name:=/dev/ttyACM1

각 옵션의 의미는 다음과 같습니다.

use_sim:=false

Gazebo 같은 시뮬레이션 환경을 사용하지 않는다는 뜻입니다.

false = 실제 로봇 사용
true  = 시뮬레이션 사용
use_mock_hardware:=false

가상 하드웨어가 아니라 실제 OMX-F를 사용한다는 뜻입니다.

false = 실제 모터 사용
true  = 모터 없이 가상 관절 사용
port_name:=/dev/ttyACM1

OMX-F가 연결된 실제 USB 포트를 지정합니다.

8. 정상 실행 로그

실제 하드웨어가 정상적으로 연결되면 터미널에서 다음과 비슷한 로그가 나타납니다.

Dynamixel Hardware Start!
Successful activate of hardware OMXFSystem

모터 ID별로 Torque가 활성화되는 로그도 나타날 수 있습니다.

Torque ON ID: 011
Torque ON ID: 012
Torque ON ID: 013
Torque ON ID: 014
Torque ON ID: 015
Torque ON ID: 016

이 로그는 다음 과정이 성공했다는 뜻입니다.

USB 포트 연결 성공
→ Dynamixel 모터 통신 성공
→ OMXFSystem 활성화 성공
→ ros2_control 하드웨어 활성화 성공
9. MoveIt 화면 확인

MoveIt launch가 정상 실행되면 RViz 화면이 나타납니다.

RViz 화면에는 다음 요소가 보여야 합니다.

로봇 모델
MotionPlanning 패널
Planning Group
Plan 버튼
Execute 버튼

RViz가 실행되었지만 MoveIt 패널이 보이지 않으면 왼쪽 아래에서 다음 순서로 추가합니다.

Add
→ By display type
→ moveit_ros_visualization
→ MotionPlanning
→ OK

오른쪽 MotionPlanning 패널 자체가 보이지 않으면:

Panels
→ Add New Panel
→ MotionPlanning
→ OK
10. /controller_manager 확인

MoveIt 화면이 보이는 것과 실제 로봇이 연결된 것은 서로 다른 문제입니다.

실제 로봇이 연결되려면 반드시 다음 노드가 있어야 합니다.

/controller_manager

새 터미널을 열고 확인합니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 node list | grep controller_manager

처음에는 다음 노드만 보였습니다.

/moveit_simple_controller_manager

하지만 이것은 실제 하드웨어를 관리하는 노드가 아닙니다.

두 노드의 차이
/moveit_simple_controller_manager

MoveIt이 계획된 Trajectory를 어떤 컨트롤러로 보낼지 연결하는 MoveIt 내부 플러그인입니다.

/controller_manager

실제 ros2_control 컨트롤러를 관리하는 핵심 노드입니다.

따라서 실제 로봇 제어를 위해서는 반드시 다음이 있어야 합니다.

/controller_manager

정상 확인 명령:

ros2 node list | grep -E "^/controller_manager$|move_group|rviz"

정상 예:

/controller_manager
/move_group
/rviz2
11. Controller Manager 서비스 확인

Controller Manager가 실행되면 다음 서비스가 생성됩니다.

ros2 service list | grep controller_manager

정상 예:

/controller_manager/list_controllers
/controller_manager/list_hardware_components
/controller_manager/list_hardware_interfaces
/controller_manager/load_controller
/controller_manager/switch_controller

처음에는 다음 서비스만 보였습니다.

/moveit_simple_controller_manager/describe_parameters
/moveit_simple_controller_manager/get_parameters
...

이 상태에서는 실제 Controller Manager가 없는 것이므로 다음 명령이 계속 대기합니다.

ros2 control list_controllers

출력:

waiting for service /controller_manager/list_controllers to become available...
Could not contact service /controller_manager/list_controllers

이는 ros2 control 명령 자체의 문제가 아니라 /controller_manager가 실행되지 않았기 때문입니다.

12. 컨트롤러 상태 확인

MoveIt과 실제 OMX-F가 정상 실행된 후 새 터미널에서 다음 명령을 실행합니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 control list_controllers

정상적으로 다음 컨트롤러들이 보여야 합니다.

joint_state_broadcaster
arm_controller
gripper_controller

정상 예:

joint_state_broadcaster    active
arm_controller             active
gripper_controller         active

각 컨트롤러의 역할은 다음과 같습니다.

joint_state_broadcaster

실제 모터의 현재 관절 위치를 읽어서 /joint_states 토픽으로 발행합니다.

arm_controller

로봇팔의 관절 Trajectory를 실행합니다.

gripper_controller

그리퍼의 열기와 닫기 동작을 제어합니다.

13. 관절 상태 확인

실제 로봇의 관절값이 ROS 2로 들어오는지 확인합니다.

ros2 topic echo /joint_states --once

정상 예:

name:
- joint1
- joint2
- joint3
- joint4
- joint5
- gripper_joint

position:
- 0.0
- -0.8
- 0.7
- 0.2
- 0.0
- 0.01

중요한 점은 RViz 화면의 로봇 자세와 실제 OMX-F의 자세가 같아야 한다는 것입니다.

실제 로봇을 손으로 움직였을 때 RViz 모델도 같이 움직이면 /joint_states가 정상적으로 연결된 것입니다.

14. MoveIt에서 Planning Group 선택

RViz의 MotionPlanning 패널에서 다음과 같이 설정합니다.

Planning Group: arm

OMX-F SRDF에는 일반적으로 다음 그룹이 있습니다.

arm
gripper

arm 그룹은 로봇팔 관절을 계획합니다.

gripper 그룹은 그리퍼를 제어합니다.

15. MoveIt에서 Plan 실행

처음에는 실제 로봇을 바로 움직이지 말고 Plan만 실행합니다.

순서:

1. Planning Group을 arm으로 선택
2. Interactive Marker를 움직여 목표 자세 지정
3. Plan 클릭
4. RViz에서 예상 경로 확인

정상적으로 계획되면 RViz에서 로봇이 움직이는 미리보기 애니메이션이 나타납니다.

터미널에는 다음과 비슷한 로그가 나올 수 있습니다.

Planning request received
Goal reached
Planning succeeded
16. 실제 로봇 Execute

계획 경로가 안전한 것을 확인한 후에만 Execute를 누릅니다.

Plan
→ 경로 확인
→ Execute

Execute를 누르면 MoveIt은 계획된 Trajectory를 다음 경로로 전달합니다.

MoveIt
→ moveit_simple_controller_manager
→ arm_controller
→ ros2_control
→ OMXFSystem
→ Dynamixel 모터
17. 실제 로봇 실행 전 안전 확인

Execute 전에 반드시 확인해야 합니다.

로봇 주변에 사람이나 물체가 없는가
USB 케이블이 로봇 관절에 걸리지 않는가
비상 정지가 가능한가
RViz 모델과 실제 로봇 자세가 같은가
목표 자세가 로봇 가동 범위 안에 있는가
관절이 급격하게 꺾이지 않는가

처음 실행할 때는 로봇 가까이에 손을 두지 않는 것이 좋습니다.

18. MoveIt이 실행되지 않을 때 확인 순서
1단계: 포트 확인
ls -l /dev/ttyACM*
2단계: 권한 설정
sudo chmod 666 /dev/ttyACM1
3단계: 포트 점유 확인
sudo fuser -v /dev/ttyACM1
4단계: MoveIt launch 확인
find ~/omx_moveit_ws/src \
  -type f \
  -name "*moveit.launch.py"
5단계: MoveIt 실행
ros2 launch open_manipulator_moveit_config \
  omx_f_moveit.launch.py \
  use_sim:=false \
  use_mock_hardware:=false \
  port_name:=/dev/ttyACM1
6단계: 노드 확인
ros2 node list | grep -E "controller_manager|move_group|rviz"
7단계: 서비스 확인
ros2 service list | grep controller_manager
8단계: 컨트롤러 확인
ros2 control list_controllers
9단계: 관절값 확인
ros2 topic echo /joint_states --once
19. 자주 발생한 문제
문제 1: 존재하지 않는 launch 파일

오류:

file 'moveit.launch.py' was not found

해결:

find ~/omx_moveit_ws/src \
  -type f \
  -name "*moveit.launch.py"

실제 존재하는 다음 파일을 사용합니다.

omx_f_moveit.launch.py
문제 2: /controller_manager가 없음

증상:

ros2 control list_controllers

실행 시 계속 대기합니다.

Could not contact service /controller_manager/list_controllers

원인:

ros2_control_node가 실행되지 않음
하드웨어 초기화 실패
포트 오류
포트 점유
launch가 실제 하드웨어 모드가 아님

해결:

use_sim:=false
use_mock_hardware:=false
port_name:=/dev/ttyACM1

로 다시 실행합니다.

문제 3: /moveit_simple_controller_manager만 보임

이 노드는 실제 하드웨어 Controller Manager가 아닙니다.

/moveit_simple_controller_manager

실제 로봇 제어에는 다음 노드가 별도로 필요합니다.

/controller_manager
문제 4: 포트 사용 중

확인:

sudo fuser -v /dev/ttyACM1

해결:

sudo fuser -k /dev/ttyACM1
sudo chmod 666 /dev/ttyACM1
문제 5: RViz에 MotionPlanning이 없음

RViz에서:

Add
→ moveit_ros_visualization
→ MotionPlanning

또는:

Panels
→ Add New Panel
→ MotionPlanning
20. 실행 명령 최종 정리
터미널 1: MoveIt과 OMX-F 실행
conda deactivate 2>/dev/null || true

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

sudo chmod 666 /dev/ttyACM1

ros2 launch open_manipulator_moveit_config \
  omx_f_moveit.launch.py \
  use_sim:=false \
  use_mock_hardware:=false \
  port_name:=/dev/ttyACM1

이 터미널은 종료하지 않습니다.

터미널 2: 상태 확인
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 node list | grep -E "controller_manager|move_group|rviz"
ros2 control list_controllers
ros2 topic echo /joint_states --once
21. 정상 상태 기준

다음 조건을 모두 만족하면 실제 OMX-F와 MoveIt 연동이 완료된 것입니다.

RViz 화면이 나타남
MoveIt MotionPlanning 패널이 보임
/controller_manager 노드가 있음
/move_group 노드가 있음
/rviz2 노드가 있음
joint_state_broadcaster가 active
arm_controller가 active
gripper_controller가 active
/joint_states 값이 출력됨
RViz 로봇 자세와 실제 로봇 자세가 일치함
Plan이 성공함
Execute 시 실제 로봇이 움직임
22. 다음 단계: Planning Scene과 장애물

실제 로봇과 MoveIt 연결이 완료되었으므로 다음 단계는 Planning Scene에 장애물을 추가하는 것입니다.

장애물 예:

작업 테이블
로봇 오른쪽 벽
로봇 앞쪽 박스
카메라 지지대
작업 대상 물체

MoveIt은 등록된 장애물과 충돌하지 않는 경로를 계산합니다.

전체 흐름:

실제 OMX-F 연결
→ MoveIt 실행
→ 관절 상태 확인
→ Planning Scene 장애물 추가
→ 충돌 회피 Plan
→ 안전 확인
→ Execute
23. 핵심 요약

실제 OMX-F와 MoveIt을 연동하는 가장 중요한 명령은 다음입니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

sudo chmod 666 /dev/ttyACM1

ros2 launch open_manipulator_moveit_config \
  omx_f_moveit.launch.py \
  use_sim:=false \
  use_mock_hardware:=false \
  port_name:=/dev/ttyACM1

실행 후 반드시 확인합니다.

ros2 control list_controllers

그리고:

ros2 topic echo /joint_states --once

MoveIt 화면만 뜨는 것으로 실제 로봇 연동이 완료된 것은 아닙니다.

반드시 다음 세 가지가 확인되어야 합니다.

/controller_manager 존재
컨트롤러 active
/joint_states 수신

이 세 가지가 정상이고 RViz에서 Plan과 Execute가 동작하면 OMX-F 실제 로봇과 MoveIt 연동이 완료된 것입니다.
