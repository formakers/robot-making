아래 명령어는 ROS 2에서 실제 OMX-F 로봇을 하드웨어와 연결하고, 관절 상태와 컨트롤러를 실행하는 Bringup 명령어입니다.

ros2 launch open_manipulator_bringup \
  omx_f.launch.py \
  port_name:=/dev/ttyACM1
Bringup이란?

Bringup은 로봇을 ROS 2에서 사용할 수 있도록 필요한 구성요소를 한꺼번에 시작하는 과정입니다.

쉽게 말하면 다음 작업을 수행합니다.

OpenRB-150 연결
        ↓
DYNAMIXEL 모터 통신 시작
        ↓
로봇 모델 불러오기
        ↓
관절 상태 읽기
        ↓
ros2_control 실행
        ↓
컨트롤러 시작
        ↓
ROS 2에서 OMX-F 제어 준비 완료

Bringup은 MoveIt처럼 경로를 계획하는 프로그램이 아닙니다.

Bringup의 역할은 먼저 실제 로봇을 ROS 2에 연결해 움직일 준비 상태로 만드는 것입니다.

전체 실행 명령어

새 터미널을 열고 다음 순서로 실행합니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 launch open_manipulator_bringup \
  omx_f.launch.py \
  port_name:=/dev/ttyACM1

각 부분을 하나씩 설명하겠습니다.

1. source /opt/ros/jazzy/setup.bash
source /opt/ros/jazzy/setup.bash

ROS 2 Jazzy 환경을 현재 터미널에 불러오는 명령어입니다.

이 명령을 실행하면 터미널에서 다음 명령들을 사용할 수 있게 됩니다.

ros2 node list
ros2 topic list
ros2 launch
ros2 control list_controllers

이 환경을 불러오지 않으면 다음 오류가 날 수 있습니다.

ros2: command not found

새 터미널을 열 때마다 실행해야 합니다.

2. source ~/omx_moveit_ws/install/setup.bash
source ~/omx_moveit_ws/install/setup.bash

사용자가 직접 빌드한 OMX 관련 ROS 2 패키지를 현재 터미널에 등록하는 명령어입니다.

여기서 워크스페이스 경로는 다음입니다.

/home/formakers/omx_moveit_ws

이 명령을 실행하면 ROS 2가 다음 패키지를 찾을 수 있습니다.

open_manipulator_bringup
open_manipulator_moveit_config
open_manipulator_description
open_manipulator_hardware

이 환경을 불러오지 않으면 다음 오류가 날 수 있습니다.

Package 'open_manipulator_bringup' not found
3. ros2 launch
ros2 launch

ROS 2 패키지 안에 있는 Launch 파일을 실행하겠다는 의미입니다.

Launch 파일은 여러 개의 ROS 노드를 한꺼번에 실행하는 Python 파일입니다.

예를 들어 OMX-F Bringup Launch 파일은 내부적으로 다음과 같은 구성요소를 실행할 수 있습니다.

robot_state_publisher
controller_manager
ros2_control_node
joint_state_broadcaster
arm_controller
gripper_controller
DYNAMIXEL 하드웨어 인터페이스

각 프로그램을 하나씩 실행하는 대신 Launch 파일 하나로 함께 실행합니다.

4. open_manipulator_bringup
open_manipulator_bringup

실행할 ROS 2 패키지 이름입니다.

이 패키지는 OpenMANIPULATOR 로봇의 실제 하드웨어를 시작하기 위한 파일들을 포함합니다.

패키지가 존재하는지 확인하려면 다음 명령어를 실행합니다.

ros2 pkg prefix open_manipulator_bringup

정상이라면 다음과 비슷한 경로가 출력됩니다.

/home/formakers/omx_moveit_ws/install/open_manipulator_bringup

패키지의 Launch 파일 목록을 확인하려면:

ls $(ros2 pkg prefix open_manipulator_bringup)/share/open_manipulator_bringup/launch

여기에서 다음 파일이 보여야 합니다.

omx_f.launch.py
5. omx_f.launch.py
omx_f.launch.py

OMX-F 모델 전용 Launch 파일입니다.

즉, ROS 2에게 다음과 같이 지시하는 것입니다.

OpenMANIPULATOR 계열 중 OMX-F 로봇 설정을 사용해라.

Launch 파일은 보통 다음 설정을 읽습니다.

OMX-F URDF 또는 Xacro
관절 이름
모터 ID
통신 속도
컨트롤러 설정
하드웨어 인터페이스
OpenRB-150 포트

Launch 파일의 실제 위치를 찾으려면:

find ~/omx_moveit_ws/src \
  -type f \
  -name "omx_f.launch.py"

설치된 파일을 찾으려면:

find ~/omx_moveit_ws/install \
  -type f \
  -name "omx_f.launch.py"
6. port_name:=/dev/ttyACM1
port_name:=/dev/ttyACM1

OMX-F에 연결된 OpenRB-150의 USB 시리얼 포트를 지정하는 부분입니다.

구조는 다음과 같습니다.

Launch 인자 이름 := 전달할 값

즉:

port_name := /dev/ttyACM1

의미는 다음과 같습니다.

OMX-F와 통신할 때 /dev/ttyACM1 장치를 사용해라.

Linux에서는 OpenRB-150 같은 USB 시리얼 장치가 일반적으로 다음처럼 나타납니다.

/dev/ttyACM0
/dev/ttyACM1
/dev/ttyACM2

OMX-F가 실제로 어떤 포트인지 확인하려면:

ls -l /dev/ttyACM*

더 정확하게 확인하려면:

ls -l /dev/serial/by-id/

예시:

usb-ROBOTIS_OpenRB-150_ABC-if00 -> ../../ttyACM1

그러면 해당 장치는 /dev/ttyACM1입니다.

역슬래시 \의 의미

다음 명령어는 여러 줄로 작성되어 있습니다.

ros2 launch open_manipulator_bringup \
  omx_f.launch.py \
  port_name:=/dev/ttyACM1

줄 끝의 \는 다음 줄까지 명령어가 이어진다는 의미입니다.

따라서 아래 한 줄 명령어와 완전히 같습니다.

ros2 launch open_manipulator_bringup omx_f.launch.py port_name:=/dev/ttyACM1

여러 줄로 표시하면 각 항목을 구분하기 쉬워집니다.

주의할 점은 역슬래시 뒤에 공백을 넣지 않는 것입니다.

잘못된 형태:

ros2 launch open_manipulator_bringup \ 
  omx_f.launch.py

올바른 형태:

ros2 launch open_manipulator_bringup \
  omx_f.launch.py
Bringup 실행 시 내부 흐름

명령어를 실행하면 대략 다음 과정이 진행됩니다.

1단계: 로봇 모델 불러오기

OMX-F의 로봇 구조를 읽습니다.

joint1
joint2
joint3
joint4
gripper

URDF 또는 Xacro에서 링크와 관절 정보를 불러옵니다.

2단계: OpenRB-150 포트 열기
/dev/ttyACM1

포트를 열어 OpenRB-150과 통신을 시작합니다.

다른 프로그램이 이미 이 포트를 사용 중이면 연결이 실패합니다.

확인 명령어:

sudo lsof /dev/ttyACM1

아무것도 출력되지 않으면 일반적으로 포트를 사용하는 프로그램이 없는 상태입니다.

3단계: DYNAMIXEL 모터 검색

설정된 모터 ID를 찾아 통신 가능한지 확인합니다.

예를 들어:

ID 1
ID 2
ID 3
ID 4
ID 5
ID 6

모터 전원이 꺼져 있거나 통신 케이블이 빠져 있으면 모터를 찾지 못할 수 있습니다.

4단계: ros2_control 시작

ros2_control은 ROS 2 명령과 실제 모터 사이를 연결하는 제어 시스템입니다.

구조는 다음과 같습니다.

ROS 2 명령
   ↓
Controller
   ↓
ros2_control
   ↓
DYNAMIXEL 하드웨어 인터페이스
   ↓
OpenRB-150
   ↓
모터
5단계: 관절 상태 발행

각 모터의 현재 위치, 속도, 힘 정보를 읽어 /joint_states 토픽으로 발행합니다.

확인 명령어:

ros2 topic echo /joint_states

예시:

name:
- joint1
- joint2
- joint3
- joint4

position:
- 0.02
- -0.54
- 0.41
- 0.15

position은 일반적으로 라디안 단위입니다.

0 rad      = 0도
1.57 rad   ≈ 90도
3.14 rad   ≈ 180도
-1.57 rad  ≈ -90도
6단계: 컨트롤러 실행

보통 다음과 같은 컨트롤러가 실행됩니다.

joint_state_broadcaster
arm_controller
gripper_controller

상태를 확인하려면:

ros2 control list_controllers

정상 예시:

joint_state_broadcaster  active
arm_controller           active
gripper_controller       active

중요한 상태는 active입니다.

active       실행 중
inactive     로드됐지만 정지
unconfigured 설정되지 않음
Bringup 명령어 실행 전 확인
1. 포트 확인
ls -l /dev/ttyACM*
2. OpenRB-150 장치 확인
ls -l /dev/serial/by-id/
3. 포트 권한 확인
groups

출력에 다음 항목이 있어야 합니다.

dialout

없다면:

sudo usermod -aG dialout $USER

그다음 재부팅합니다.

sudo reboot

임시 권한 설정은 다음과 같습니다.

sudo chmod 666 /dev/ttyACM1
4. 포트 점유 확인
sudo lsof /dev/ttyACM1

DYNAMIXEL Wizard 2, Arduino IDE, LeRobot 또는 다른 ROS 프로그램이 포트를 사용하고 있으면 종료해야 합니다.

Bringup 실행 후 확인

Bringup 터미널은 종료하지 않고 켜 둡니다.

새 터미널을 열고 환경을 다시 불러옵니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash
ROS 노드 확인
ros2 node list
토픽 확인
ros2 topic list
관절 상태 한 번 확인
ros2 topic echo /joint_states --once
관절 상태 실시간 확인
ros2 topic echo /joint_states
발행 속도 확인
ros2 topic hz /joint_states
컨트롤러 확인
ros2 control list_controllers
액션 서버 확인
ros2 action list

정상이라면 관절 궤적 제어 액션이 표시될 수 있습니다.

/arm_controller/follow_joint_trajectory
Bringup과 MoveIt의 차이
Bringup
ros2 launch open_manipulator_bringup \
  omx_f.launch.py \
  port_name:=/dev/ttyACM1

역할:

실제 로봇 연결
모터 통신
관절 상태 읽기
컨트롤러 실행
명령을 받을 준비
MoveIt

MoveIt의 역할:

RViz에서 목표 자세 설정
경로 계획
충돌 검사
역기구학 계산
계획한 궤적을 컨트롤러에 전달

전체 구조는 다음과 같습니다.

MoveIt
  ↓
arm_controller
  ↓
ros2_control
  ↓
OpenRB-150
  ↓
DYNAMIXEL
  ↓
OMX-F 실제 로봇

Bringup이 먼저 정상적으로 실행되어야 MoveIt이 실제 로봇을 움직일 수 있습니다.

Bringup을 실행하면 바로 움직이는가?

일반적으로 Bringup만 실행하면 로봇이 큰 동작을 자동으로 수행하지는 않습니다.

Bringup은 다음 상태를 만듭니다.

모터 연결
현재 위치 확인
컨트롤러 실행
제어 명령 대기

다만 실행 과정에서 토크가 켜질 수 있습니다. 그러면 로봇팔이 현재 자세를 유지하면서 뻣뻣해질 수 있습니다.

토크가 켜진 상태에서는 관절을 손으로 강하게 움직이면 안 됩니다.

포트가 /dev/ttyACM0인 경우

현재 OMX-F가 /dev/ttyACM0으로 연결되어 있다면 다음처럼 실행합니다.

ros2 launch open_manipulator_bringup \
  omx_f.launch.py \
  port_name:=/dev/ttyACM0

USB를 분리했다가 다시 연결하면 포트 번호가 바뀔 수 있으므로 실행 전에 항상 확인하는 것이 좋습니다.

ls -l /dev/ttyACM*
자주 발생하는 오류
패키지를 찾지 못함
Package 'open_manipulator_bringup' not found

해결:

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash
Launch 파일을 찾지 못함
file 'omx_f.launch.py' was not found

파일 목록 확인:

ls $(ros2 pkg prefix open_manipulator_bringup)/share/open_manipulator_bringup/launch
포트가 없음
No such file or directory: /dev/ttyACM1

확인:

ls -l /dev/ttyACM*
포트 권한 오류
Permission denied

임시 해결:

sudo chmod 666 /dev/ttyACM1

근본 해결:

sudo usermod -aG dialout $USER
sudo reboot
포트 사용 중
Device or resource busy

확인:

sudo lsof /dev/ttyACM1

해당 포트를 사용하는 프로그램을 종료합니다.

모터 검색 실패
Failed to ping DYNAMIXEL

확인할 항목:

모터 전원
OpenRB-150 전원
DYNAMIXEL 케이블
모터 ID
Baud Rate
포트 번호
다른 프로그램의 포트 점유
OpenRB-150 펌웨어
권장 실행 순서
터미널 1: 실제 로봇 Bringup
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ls -l /dev/ttyACM*
sudo lsof /dev/ttyACM1

ros2 launch open_manipulator_bringup \
  omx_f.launch.py \
  port_name:=/dev/ttyACM1
터미널 2: 상태 확인
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 node list
ros2 control list_controllers
ros2 topic echo /joint_states --once

핵심적으로 이 Bringup 명령어는 /dev/ttyACM1에 연결된 OpenRB-150과 OMX-F 모터를 ROS 2에 연결하고, 관절 상태와 컨트롤러를 실행하여 로봇을 제어할 준비를 하는 명령어입니다.
