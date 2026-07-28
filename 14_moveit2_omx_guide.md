OMX-F + ROS2 Jazzy + MoveIt 실행 전체 과정
1단계. 새로운 터미널 열기

Ubuntu에서 터미널을 엽니다.

단축키

Ctrl + Alt + T
2단계. ROS2 환경 불러오기

ROS2 명령어를 사용할 수 있도록 환경을 설정합니다.

source /opt/ros/jazzy/setup.bash
확인
echo $ROS_DISTRO

결과

jazzy
3단계. Workspace 환경 불러오기

직접 만든 패키지를 사용할 수 있도록 합니다.

source ~/omx_moveit_ws/install/setup.bash
4단계. 연결된 OpenRB 확인
ls /dev/ttyACM*

예)

/dev/ttyACM0
/dev/ttyACM1

또는

ls -l /dev/serial/by-id/

예)

usb-ROBOTIS_OpenRB...

이 명령은

어느 포트가 Leader인지

어느 포트가 Follower인지

확인할 때 사용합니다.

5단계. 실제 OMX 하드웨어 실행

예를 들어 ttyACM1이라면

ros2 launch open_manipulator_bringup \
  omx_f.launch.py \
  port_name:=/dev/ttyACM1

정상이라면

Succeeded to open the port

그리고

Torque ON

이 출력됩니다.

이 터미널은 절대 종료하지 않습니다.

6단계. 새로운 터미널 열기

다시

Ctrl + Alt + T
7단계. 환경 설정
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash
8단계. MoveIt 실행
ros2 launch open_manipulator_moveit_config \
  omx_f_moveit.launch.py

정상이라면

RViz가 열립니다.

9단계. Controller 확인

새 터미널

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

확인

ros2 control list_controllers

정상

joint_state_broadcaster active

arm_controller active

gripper_controller active
10단계. Joint 확인
ros2 topic echo /joint_states --once

정상

joint1

joint2

joint3

joint4

등이 출력됩니다.

11단계. MoveIt에서 테스트

RViz에서

Planning

↓

Plan

↓

Execute

실행하면

실제 OMX가 움직입니다.

12단계. 종료

먼저

MoveIt

Ctrl+C

다음

Bringup

Ctrl+C
문제가 생기면

현재 실행중인 노드 확인

ros2 node list

Controller 확인

ros2 control list_controllers

Robot Description 확인

ros2 topic echo /robot_description --once

Joint 확인

ros2 topic echo /joint_states --once

포트 확인

ls /dev/ttyACM*
만약 launch가 없다고 나오면

실제 launch 확인

find ~/omx_moveit_ws/src \
-type f \
-name "*.launch.py"
패키지만 다시 빌드
cd ~/omx_moveit_ws

source /opt/ros/jazzy/setup.bash

colcon build \
--packages-select open_manipulator_moveit_config \
--symlink-install

환경 다시 적용

source ~/omx_moveit_ws/install/setup.bash
가장 많이 사용하는 명령어만 모아놓은 실행 순서
터미널 1
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 launch open_manipulator_bringup \
omx_f.launch.py \
port_name:=/dev/ttyACM1
터미널 2
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 launch open_manipulator_moveit_config \
omx_f_moveit.launch.py
터미널 3 (확인용)
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 control list_controllers
다음 단계 추천

이제 기본적인 MoveIt 실행이 성공했으므로, 다음 순서로 학습하면 자연스럽게 실력이 쌓입니다.

MoveIt RViz 사용법 (Goal State, Planning, Execute)
Planning Scene과 장애물 추가
MoveIt Python API로 로봇 제어
YOLO + MoveIt 연동하여 물체 위치로 자동 이동
Depth Camera + AI Pick & Place 구현

이 과정을 차근차근 익히면, 실제 OMX-F를 이용한 AI 로봇팔 자동화 프로젝트까지 이어갈 수 있습니다.
