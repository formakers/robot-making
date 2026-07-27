AI 로봇 제작 마스터 클래스
14단계: MoveIt 2 Python API로 로봇 제어하기

13단계에서 RViz의 Plan & Execute 버튼으로 로봇팔을 움직였다면, 14단계에서는 같은 작업을 Python 프로그램으로 자동 실행합니다.

MoveIt 2의 공식 Python 인터페이스는 moveit_py입니다. moveit_py를 사용하면 Python에서 현재 관절 상태 읽기, 목표 관절값 설정, 말단장치 목표 좌표 설정, 경로 계획, 충돌 검사, 궤적 실행 등을 수행할 수 있습니다. 공식 문서에서는 Python API를 빠른 프로토타입 제작과 Python 기반 로봇 응용 개발에 적합한 인터페이스로 설명합니다.

1. 14단계의 최종 목표

이번 단계에서는 다음 흐름을 완성합니다.

Python 프로그램
      ↓
목표 관절각 또는 목표 좌표 설정
      ↓
MoveIt 2 경로 계획
      ↓
충돌 여부 검사
      ↓
로봇 궤적 생성
      ↓
ros2_control 컨트롤러
      ↓
Gazebo 또는 실제 OMX-F 로봇 움직임

최종적으로 다음과 같은 프로그램을 만들게 됩니다.

1. 현재 OMX 관절값 읽기
2. joint1의 현재 위치 확인
3. joint1에 +0.05 rad 적용
4. MoveIt으로 이동 경로 계산
5. 안전한 경로이면 실행
6. 실제 OMX-F 로봇 이동
2. MoveIt Python API 핵심 개념
2.1 MoveItPy

MoveItPy는 Python 프로그램과 MoveIt 2를 연결하는 중심 객체입니다.

from moveit.planning import MoveItPy

robot = MoveItPy(node_name="omx_moveit_python")

이 객체는 다음 정보를 사용합니다.

로봇 URDF
SRDF
관절 제한
운동학 설정
OMPL 플래너 설정
ros2_control 컨트롤러 설정
Planning Scene
현재 /joint_states

공식 API에서 MoveItPy는 MoveIt Python API의 주 인터페이스이며 내부적으로 MoveIt C++ 기능을 감싸는 구조입니다.

2.2 Planning Component

Planning Component는 특정 로봇 그룹을 계획하는 객체입니다.

OMX-F의 SRDF에 다음 그룹이 있다고 가정합니다.

<group name="arm">
    ...
</group>

<group name="gripper">
    ...
</group>

그러면 Python에서는 다음과 같이 가져옵니다.

arm = robot.get_planning_component("arm")
gripper = robot.get_planning_component("gripper")

중요한 점은 "arm"이라는 이름을 임의로 정하는 것이 아니라, 반드시 SRDF에 정의된 그룹 이름과 일치해야 한다는 것입니다.

현재 사용 중인 OMX-F 설정에서는 앞서 확인한 것처럼 대체로 다음 그룹 구성을 사용합니다.

arm
 ├── joint1
 ├── joint2
 ├── joint3
 ├── joint4
 ├── joint5
 └── end_effector_joint

gripper
 ├── gripper_joint_1
 └── gripper_joint_2

다음 명령으로 실제 그룹 이름을 확인할 수 있습니다.

grep '<group name=' \
~/omx_moveit_ws/src/*/config/omx_f/omx_f.srdf

파일 위치가 다르면 다음처럼 검색합니다.

find ~/omx_moveit_ws/src \
-name "*.srdf" \
-exec grep -H '<group name=' {} \;
2.3 Robot State

Robot State는 특정 시점의 로봇 관절 상태입니다.

예를 들면 다음과 같습니다.

joint1 = 0.00 rad
joint2 = -1.00 rad
joint3 = 0.70 rad
joint4 = 0.30 rad
joint5 = 0.00 rad

MoveIt은 이 관절값을 사용해 각 링크의 위치와 자세를 계산합니다.

Python에서는 다음과 같은 방식으로 Robot State를 생성합니다.

from moveit.core.robot_state import RobotState

robot_model = robot.get_robot_model()
goal_state = RobotState(robot_model)

현재 상태를 계획 시작점으로 지정할 때는 다음 메서드를 사용합니다.

arm.set_start_state_to_current_state()

공식 예제 역시 현재 상태를 시작 상태로 설정한 뒤 Robot State 또는 Pose를 목표로 지정합니다.

2.4 Planning

Planning은 현재 상태에서 목표 상태까지 이동할 수 있는 경로를 계산하는 과정입니다.

plan_result = arm.plan()

성공 여부를 확인합니다.

if plan_result:
    print("경로 계획 성공")
else:
    print("경로 계획 실패")

계획에 성공하면 궤적이 포함됩니다.

trajectory = plan_result.trajectory
2.5 Execute

계획된 궤적을 컨트롤러로 전달합니다.

robot.execute(trajectory, controllers=[])

공식 예제도 plan()으로 궤적을 만든 뒤 MoveItPy.execute()를 사용해 실행합니다.

controllers=[]는 MoveIt 설정에 등록된 기본 컨트롤러를 사용하라는 의미로 활용됩니다.

OMX-F에서는 일반적으로 다음 컨트롤러가 필요합니다.

joint_state_broadcaster
arm_controller
gripper_controller
3. MoveIt Python 프로그램의 기본 구조

가장 기본적인 구조는 다음과 같습니다.

import rclpy

from moveit.planning import MoveItPy


def main():
    # ROS 2 Python 초기화
    rclpy.init()

    # MoveItPy 객체 생성
    robot = MoveItPy(
        node_name="omx_moveit_python"
    )

    # SRDF의 arm 그룹 가져오기
    arm = robot.get_planning_component("arm")

    # 현재 관절 상태를 시작점으로 설정
    arm.set_start_state_to_current_state()

    # 목표 설정
    # arm.set_goal_state(...)

    # 경로 계획
    plan_result = arm.plan()

    if plan_result:
        print("경로 계획 성공")

        # 계획된 궤적 실행
        robot.execute(
            plan_result.trajectory,
            controllers=[]
        )
    else:
        print("경로 계획 실패")

    rclpy.shutdown()


if __name__ == "__main__":
    main()

단, 이 코드에는 아직 목표값이 없으므로 실제 계획은 실행되지 않습니다.

4. MoveIt Python에서 목표를 설정하는 방법

목표 설정에는 크게 세 가지 방식이 있습니다.

방법 1: SRDF의 저장된 자세 사용

SRDF에 home, ready 같은 자세가 저장되어 있을 때 사용합니다.

arm.set_start_state_to_current_state()
arm.set_goal_state(configuration_name="home")

공식 예제에서는 다음과 같은 구조를 사용합니다.

planning_component.set_start_state(
    configuration_name="ready"
)

planning_component.set_goal_state(
    configuration_name="extended"
)

이 방식은 미리 정의된 안전 자세로 이동할 때 편리합니다.

SRDF의 저장 자세를 확인합니다.

grep -A 10 '<group_state' \
~/omx_moveit_ws/src/*/config/omx_f/omx_f.srdf

예를 들어 SRDF가 다음과 같다면,

<group_state name="home" group="arm">
    <joint name="joint1" value="0.0"/>
    <joint name="joint2" value="-1.0"/>
    <joint name="joint3" value="0.7"/>
    <joint name="joint4" value="0.3"/>
    <joint name="joint5" value="0.0"/>
</group_state>

Python에서 다음처럼 사용할 수 있습니다.

arm.set_goal_state(configuration_name="home")
방법 2: 관절값으로 목표 설정

각 관절을 직접 지정하는 방식입니다.

goal_state.joint_positions = {
    "joint1": 0.2,
    "joint2": -1.0,
    "joint3": 0.7,
    "joint4": 0.3,
    "joint5": 0.0,
}

그다음 제약조건을 생성해 목표로 전달합니다.

from moveit.core.kinematic_constraints import (
    construct_joint_constraint,
)

joint_model_group = (
    robot.get_robot_model()
    .get_joint_model_group("arm")
)

joint_constraint = construct_joint_constraint(
    robot_state=goal_state,
    joint_model_group=joint_model_group,
)

arm.set_goal_state(
    motion_plan_constraints=[joint_constraint]
)

공식 예제에서도 관절 딕셔너리를 Robot State에 적용하고, construct_joint_constraint()로 목표 조건을 만들어 계획합니다.

방법 3: 말단장치 좌표로 목표 설정

로봇 끝부분을 특정 X, Y, Z 위치로 이동시키는 방법입니다.

from geometry_msgs.msg import PoseStamped

target_pose = PoseStamped()

target_pose.header.frame_id = "base_link"

target_pose.pose.position.x = 0.20
target_pose.pose.position.y = 0.00
target_pose.pose.position.z = 0.15

target_pose.pose.orientation.x = 0.0
target_pose.pose.orientation.y = 0.0
target_pose.pose.orientation.z = 0.0
target_pose.pose.orientation.w = 1.0

목표 자세를 설정합니다.

arm.set_goal_state(
    pose_stamped_msg=target_pose,
    pose_link="end_effector_link"
)

공식 MoveIt Python 예제도 PoseStamped를 만들고 기준 프레임, 위치, 방향, 말단 링크를 지정합니다.

여기서 다음 두 이름은 실제 OMX URDF에 맞춰야 합니다.

base_link
end_effector_link

확인 명령:

grep '<link name=' \
~/omx_moveit_ws/src/*/urdf/*.urdf*

또는 실행 중 다음 명령으로 TF 프레임을 확인합니다.

ros2 run tf2_tools view_frames
5. 실습 1: OMX 현재 관절 상태 확인

먼저 MoveIt Python을 사용하기 전에 /joint_states가 정상인지 확인합니다.

터미널 1
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 topic echo /joint_states --once

정상 예:

name:
- joint1
- joint2
- joint3
- joint4
- joint5

position:
- 0.001
- -1.000
- 0.700
- 0.300
- 0.000

다음과 같은 메시지가 나오면 MoveIt Python을 실행하면 안 됩니다.

topic [/joint_states] does not appear to be published

이 경우 먼저 다음을 점검합니다.

ros2 node list
ros2 control list_controllers
ros2 topic list | grep joint

정상적인 컨트롤러 상태:

joint_state_broadcaster active
arm_controller active
gripper_controller active
6. 실습 2: Python 패키지 만들기

현재 워크스페이스를 사용합니다.

cd ~/omx_moveit_ws/src

Python 패키지를 생성합니다.

ros2 pkg create omx_moveit_python \
  --build-type ament_python \
  --dependencies \
  rclpy \
  geometry_msgs \
  sensor_msgs \
  moveit_py \
  moveit_msgs

생성 구조:

omx_moveit_python/
├── package.xml
├── setup.py
├── setup.cfg
├── resource/
│   └── omx_moveit_python
└── omx_moveit_python/
    ├── __init__.py
    └── ...
7. 실습 3: SRDF에 저장된 자세로 이동

파일을 만듭니다.

nano ~/omx_moveit_ws/src/omx_moveit_python/omx_moveit_python/move_home.py

아래 코드를 입력합니다.

#!/usr/bin/env python3

"""
OMX-F 로봇팔을 SRDF에 정의된 자세로 이동시키는 예제
"""

import rclpy

from moveit.planning import MoveItPy


def main():
    # ROS 2 Python 시스템 초기화
    rclpy.init()

    logger = rclpy.logging.get_logger(
        "omx_move_home"
    )

    logger.info("MoveItPy 초기화 시작")

    # MoveIt 2 Python 인터페이스 생성
    robot = MoveItPy(
        node_name="omx_move_home"
    )

    # SRDF에 정의된 arm 그룹 가져오기
    arm = robot.get_planning_component("arm")

    logger.info("현재 상태를 시작 상태로 설정")

    # 현재 실제 관절 상태를 출발점으로 사용
    arm.set_start_state_to_current_state()

    logger.info("목표 자세 설정")

    # SRDF에 home이라는 group_state가 있어야 함
    arm.set_goal_state(
        configuration_name="home"
    )

    logger.info("경로 계획 시작")

    plan_result = arm.plan()

    if plan_result:
        logger.info("경로 계획 성공")

        robot.execute(
            plan_result.trajectory,
            controllers=[]
        )

        logger.info("궤적 실행 완료")

    else:
        logger.error("경로 계획 실패")

    rclpy.shutdown()


if __name__ == "__main__":
    main()

주의:

configuration_name="home"

home이 SRDF에 없다면 오류가 발생합니다.

저장 자세 이름을 먼저 확인해야 합니다.

grep 'group_state name=' \
~/omx_moveit_ws/src/*/config/omx_f/omx_f.srdf
8. setup.py에 실행 명령 등록

파일을 엽니다.

nano ~/omx_moveit_ws/src/omx_moveit_python/setup.py

entry_points를 다음처럼 수정합니다.

entry_points={
    "console_scripts": [
        "move_home = omx_moveit_python.move_home:main",
    ],
},

전체 핵심 구조:

from setuptools import find_packages, setup

package_name = "omx_moveit_python"

setup(
    name=package_name,
    version="0.0.0",
    packages=find_packages(
        exclude=["test"]
    ),
    data_files=[
        (
            "share/ament_index/resource_index/packages",
            ["resource/" + package_name],
        ),
        (
            "share/" + package_name,
            ["package.xml"],
        ),
    ],
    install_requires=["setuptools"],
    zip_safe=True,
    maintainer="formakers",
    maintainer_email="formakers@example.com",
    description="OMX MoveIt Python examples",
    license="Apache-2.0",
    entry_points={
        "console_scripts": [
            "move_home = "
            "omx_moveit_python.move_home:main",
        ],
    },
)
9. 빌드하기
cd ~/omx_moveit_ws

source /opt/ros/jazzy/setup.bash

colcon build \
  --symlink-install \
  --packages-select omx_moveit_python

빌드 후 환경을 다시 적용합니다.

source ~/omx_moveit_ws/install/setup.bash

실행 파일이 등록되었는지 확인합니다.

ros2 pkg executables omx_moveit_python

정상 출력:

omx_moveit_python move_home
10. 중요한 문제: MoveIt 파라미터 전달

MoveItPy()는 다음 설정이 필요합니다.

robot_description
robot_description_semantic
robot_description_kinematics
planning_pipelines
trajectory_execution
planning_scene_monitor
controllers

따라서 단순히 다음 명령만 실행하면,

ros2 run omx_moveit_python move_home

설정에 따라 다음 오류가 생길 수 있습니다.

Robot model not loaded
Could not find parameter robot_description
Planning pipeline not configured
No planning plugin loaded

MoveIt 공식 예제 역시 Python 노드에 Planning Scene Monitor와 planning pipeline 등의 파라미터를 제공해야 한다고 설명합니다.

따라서 Launch 파일로 Python 노드에 MoveIt 설정을 전달하는 것이 안전합니다.

11. 실습 4: Python 노드용 Launch 파일 만들기

폴더를 만듭니다.

mkdir -p \
~/omx_moveit_ws/src/omx_moveit_python/launch

파일 생성:

nano \
~/omx_moveit_ws/src/omx_moveit_python/launch/move_home.launch.py

기본 구조:

from launch import LaunchDescription

from launch_ros.actions import Node

from moveit_configs_utils import (
    MoveItConfigsBuilder,
)


def generate_launch_description():

    # OMX-F MoveIt 설정 패키지에서
    # URDF, SRDF, 운동학, 플래너,
    # 컨트롤러 설정을 불러옵니다.
    moveit_config = (
        MoveItConfigsBuilder(
            robot_name="omx_f",
            package_name=(
                "open_manipulator_moveit_config"
            ),
        )
        .to_moveit_configs()
    )

    move_home_node = Node(
        package="omx_moveit_python",
        executable="move_home",
        output="screen",
        parameters=[
            moveit_config.to_dict(),
        ],
    )

    return LaunchDescription([
        move_home_node,
    ])

다만 open_manipulator_moveit_config의 폴더 구조가 일반적인 MoveIt Config 패키지와 조금 다르면 robot_name="omx_f" 자동 탐색이 실패할 수 있습니다.

현재 사용 중인 패키지는 다음 구조를 가진 것으로 확인되었습니다.

open_manipulator_moveit_config/
└── config/
    └── omx_f/
        ├── omx_f.srdf
        ├── kinematics.yaml
        ├── joint_limits.yaml
        ├── ompl_planning.yaml
        └── ...

따라서 실제 패키지에서 제공하는 기존 Launch 파일의 설정 방식을 재사용하는 것이 가장 안전합니다.

확인:

find \
~/omx_moveit_ws/src \
-path "*open_manipulator_moveit_config*" \
-name "*.launch.py"

그리고 다음 파일을 확인합니다.

sed -n '1,240p' \
~/omx_moveit_ws/src/*/launch/*.launch.py
12. Launch 파일 설치 설정

setup.py에 Launch 파일 설치 항목을 추가합니다.

import os

from glob import glob
from setuptools import find_packages, setup

data_files에 다음을 추가합니다.

(
    os.path.join(
        "share",
        package_name,
        "launch"
    ),
    glob("launch/*.launch.py"),
),

예:

data_files=[
    (
        "share/ament_index/resource_index/packages",
        ["resource/" + package_name],
    ),
    (
        "share/" + package_name,
        ["package.xml"],
    ),
    (
        os.path.join(
            "share",
            package_name,
            "launch"
        ),
        glob("launch/*.launch.py"),
    ),
],

다시 빌드합니다.

cd ~/omx_moveit_ws

colcon build \
  --symlink-install \
  --packages-select omx_moveit_python

source install/setup.bash

실행:

ros2 launch \
  omx_moveit_python \
  move_home.launch.py
13. 실습 5: 현재 joint1에서 0.05rad 이동

이번 단계에서 가장 중요한 실습입니다.

목표:

현재 joint1 = 현재값
목표 joint1 = 현재값 + 0.05 rad
나머지 관절 = 현재값 유지

0.05rad는 약 2.86°입니다.

다음 파일을 만듭니다.

nano \
~/omx_moveit_ws/src/omx_moveit_python/omx_moveit_python/move_joint1_relative.py

코드:

#!/usr/bin/env python3

"""
현재 OMX-F 관절값을 읽고
joint1만 현재 위치에서 +0.05rad 이동하는 예제
"""

import time

import rclpy

from moveit.core.kinematic_constraints import (
    construct_joint_constraint,
)
from moveit.core.robot_state import RobotState
from moveit.planning import MoveItPy


ARM_GROUP = "arm"
TARGET_JOINT = "joint1"
DELTA_RAD = 0.05


def main():
    rclpy.init()

    logger = rclpy.logging.get_logger(
        "move_joint1_relative"
    )

    logger.info("MoveItPy 초기화")

    robot = MoveItPy(
        node_name="move_joint1_relative"
    )

    arm = robot.get_planning_component(
        ARM_GROUP
    )

    # /joint_states가 MoveIt에 들어올 시간을
    # 잠시 확보합니다.
    time.sleep(2.0)

    # 현재 상태를 계획 시작점으로 지정
    arm.set_start_state_to_current_state()

    # MoveIt 로봇 모델 가져오기
    robot_model = robot.get_robot_model()

    # 목표 상태용 RobotState 생성
    goal_state = RobotState(robot_model)

    # Planning Scene에서 현재 상태 읽기
    planning_scene_monitor = (
        robot.get_planning_scene_monitor()
    )

    with planning_scene_monitor.read_only() as scene:
        current_state = scene.current_state

        # arm 그룹의 현재 관절값 읽기
        current_positions = (
            current_state
            .get_joint_group_positions(
                ARM_GROUP
            )
        )

    logger.info(
        f"현재 arm 관절값: "
        f"{current_positions}"
    )

    # 현재 위치를 목표 상태에 복사
    goal_state.set_joint_group_positions(
        ARM_GROUP,
        current_positions,
    )

    # joint1이 arm 그룹의 첫 번째 관절이라는
    # 전제에서 첫 번째 값을 변경합니다.
    target_positions = list(current_positions)

    target_positions[0] += DELTA_RAD

    logger.info(
        f"joint1 목표 이동량: "
        f"{DELTA_RAD:.3f} rad"
    )

    logger.info(
        f"목표 arm 관절값: "
        f"{target_positions}"
    )

    # 수정된 관절값을 목표 RobotState에 반영
    goal_state.set_joint_group_positions(
        ARM_GROUP,
        target_positions,
    )

    # 링크 변환값 업데이트
    goal_state.update()

    # 목표 관절 상태를 MoveIt 제약조건으로 변환
    joint_model_group = (
        robot_model.get_joint_model_group(
            ARM_GROUP
        )
    )

    joint_constraint = (
        construct_joint_constraint(
            robot_state=goal_state,
            joint_model_group=(
                joint_model_group
            ),
        )
    )

    # 목표 상태 지정
    arm.set_goal_state(
        motion_plan_constraints=[
            joint_constraint
        ]
    )

    logger.info("경로 계획 시작")

    plan_result = arm.plan()

    if not plan_result:
        logger.error("경로 계획 실패")
        rclpy.shutdown()
        return

    logger.info("경로 계획 성공")
    logger.info("궤적 실행 시작")

    robot.execute(
        plan_result.trajectory,
        controllers=[],
    )

    logger.info("joint1 이동 명령 완료")

    rclpy.shutdown()


if __name__ == "__main__":
    main()
setup.py에 등록
entry_points={
    "console_scripts": [
        "move_home = "
        "omx_moveit_python.move_home:main",

        "move_joint1_relative = "
        "omx_moveit_python."
        "move_joint1_relative:main",
    ],
},

빌드:

cd ~/omx_moveit_ws

colcon build \
  --symlink-install \
  --packages-select omx_moveit_python

source install/setup.bash
14. 실제 OMX-F에서 실행하기 전 점검

실제 하드웨어를 연결합니다.

터미널 1: 포트 확인
ls -l /dev/ttyACM*

안정적으로 장치를 찾으려면:

ls -l /dev/serial/by-id/

권한 확인:

groups

출력에 dialout이 있어야 합니다.

formakers adm cdrom sudo dip plugdev lpadmin dialout

없다면:

sudo usermod -aG dialout $USER

이후 로그아웃과 로그인이 필요합니다.

터미널 2: OMX-F 하드웨어 실행
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

현재 사용 중인 OMX-F Launch 예:

ros2 launch \
  open_manipulator_bringup \
  omx_f.launch.py \
  use_sim:=false \
  use_mock_hardware:=false \
  port_name:=/dev/ttyACM0

실행 로그에서 다음 내용이 나와야 합니다.

Dynamixel Hardware Start!
Successful activate of hardware OMXFSystem
Torque ON ID: 011
Torque ON ID: 012
Torque ON ID: 013
Torque ON ID: 014
Torque ON ID: 015
Torque ON ID: 016
터미널 3: 컨트롤러 확인
source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 control list_controllers

정상 예:

joint_state_broadcaster
  joint_state_broadcaster/JointStateBroadcaster
  active

arm_controller
  joint_trajectory_controller/JointTrajectoryController
  active

gripper_controller
  position_controllers/GripperActionController
  active
터미널 4: 현재 관절값 확인
ros2 topic echo /joint_states --once

관절 이름을 반드시 확인합니다.

name:
- joint1
- joint2
- joint3
- joint4

코드가 사용하는 관절 이름과 일치해야 합니다.

15. 처음에는 Gazebo 또는 Mock Hardware에서 실행

실제 로봇을 바로 움직이지 말고 먼저 다음 순서로 테스트하는 것이 안전합니다.

1. RViz 또는 Gazebo 실행
2. /joint_states 확인
3. 컨트롤러 active 확인
4. Python 계획만 실행
5. RViz 궤적 확인
6. 실행 테스트
7. 실제 OMX-F 연결
8. 0.02~0.05rad의 작은 이동부터 테스트

실제 로봇에서는 다음 조건을 적용하세요.

DELTA_RAD = 0.02

처음에는 0.05rad보다 더 작은 값으로 시험할 수 있습니다.

또한 로봇 주변에서 손을 떼고 비상 정지 방법을 확보해야 합니다.

16. 실습 6: XYZ 좌표로 OMX 말단장치 이동

파일:

nano \
~/omx_moveit_ws/src/omx_moveit_python/omx_moveit_python/move_pose.py

코드:

#!/usr/bin/env python3

"""
OMX-F 말단장치를 지정된 XYZ 좌표로 이동
"""

import rclpy

from geometry_msgs.msg import PoseStamped
from moveit.planning import MoveItPy


def main():
    rclpy.init()

    logger = rclpy.logging.get_logger(
        "omx_move_pose"
    )

    robot = MoveItPy(
        node_name="omx_move_pose"
    )

    arm = robot.get_planning_component(
        "arm"
    )

    # 현재 관절 상태에서 계획 시작
    arm.set_start_state_to_current_state()

    target_pose = PoseStamped()

    # OMX의 기준 프레임 이름으로 변경
    target_pose.header.frame_id = "base_link"

    # 목표 위치 단위는 미터
    target_pose.pose.position.x = 0.20
    target_pose.pose.position.y = 0.00
    target_pose.pose.position.z = 0.15

    # Quaternion 방향
    target_pose.pose.orientation.x = 0.0
    target_pose.pose.orientation.y = 0.0
    target_pose.pose.orientation.z = 0.0
    target_pose.pose.orientation.w = 1.0

    # 실제 OMX의 말단 링크 이름으로 변경
    arm.set_goal_state(
        pose_stamped_msg=target_pose,
        pose_link="end_effector_link",
    )

    logger.info(
        "말단장치 목표 위치: "
        "x=0.20, y=0.00, z=0.15"
    )

    plan_result = arm.plan()

    if plan_result:
        logger.info("경로 계획 성공")

        robot.execute(
            plan_result.trajectory,
            controllers=[],
        )
    else:
        logger.error(
            "경로 계획 실패: "
            "좌표, IK, 관절 제한을 확인하세요."
        )

    rclpy.shutdown()


if __name__ == "__main__":
    main()

말단 목표 Pose 계획은 MoveIt이 내부적으로 역기구학을 이용해 해당 Pose를 만족하는 관절값을 찾고, 충돌 없는 궤적을 계산하는 방식입니다. 공식 예제도 PoseStamped와 pose_link를 사용해 목표를 지정합니다.

17. 좌표 목표에서 자주 발생하는 실패 원인
17.1 목표 위치가 로봇 작업 범위 밖
x = 1.0m
y = 1.0m
z = 1.0m

작은 OMX 로봇팔이 도달할 수 없는 좌표라면 IK가 실패합니다.

처음에는 현재 위치와 가까운 좌표를 사용해야 합니다.

17.2 말단 링크 이름 오류
pose_link="end_effector_link"

실제 URDF에 이 이름이 없으면 실패합니다.

확인:

grep '<link name=' \
~/omx_moveit_ws/src/*/urdf/*.xacro
17.3 기준 프레임 이름 오류
target_pose.header.frame_id = "base_link"

실제 기준 프레임이 다음 중 하나일 수 있습니다.

world
base_link
link1
base_footprint

확인:

ros2 run tf2_ros tf2_echo \
  base_link \
  end_effector_link
17.4 Quaternion 오류

다음처럼 모든 값이 0이면 잘못된 Quaternion입니다.

x = 0.0
y = 0.0
z = 0.0
w = 0.0

최소한 기본 방향은 다음처럼 설정합니다.

x = 0.0
y = 0.0
z = 0.0
w = 1.0
18. Planning Scene과 충돌물체

MoveIt의 중요한 기능은 장애물을 피해 움직이는 것입니다.

예를 들어 작업대 위에 상자를 추가할 수 있습니다.

from geometry_msgs.msg import Pose
from moveit_msgs.msg import CollisionObject
from shape_msgs.msg import SolidPrimitive

Planning Scene Monitor를 가져옵니다.

planning_scene_monitor = (
    robot.get_planning_scene_monitor()
)

상자를 추가합니다.

with planning_scene_monitor.read_write() as scene:

    collision_object = CollisionObject()

    collision_object.header.frame_id = (
        "base_link"
    )

    collision_object.id = "work_table"

    box = SolidPrimitive()
    box.type = SolidPrimitive.BOX

    # 가로, 세로, 높이
    box.dimensions = [
        0.50,
        0.50,
        0.05,
    ]

    box_pose = Pose()
    box_pose.position.x = 0.20
    box_pose.position.y = 0.00
    box_pose.position.z = -0.025
    box_pose.orientation.w = 1.0

    collision_object.primitives.append(box)
    collision_object.primitive_poses.append(
        box_pose
    )

    collision_object.operation = (
        CollisionObject.ADD
    )

    scene.apply_collision_object(
        collision_object
    )

    scene.current_state.update()

공식 Python API 예제에서도 Planning Scene의 쓰기 컨텍스트 안에서 CollisionObject를 추가하고, 상태를 업데이트합니다.

19. 계획과 실행을 분리해야 하는 이유

다음처럼 계획 성공 직후 바로 실행할 수 있습니다.

plan_result = arm.plan()

if plan_result:
    robot.execute(
        plan_result.trajectory,
        controllers=[]
    )

하지만 실제 로봇에서는 다음 구조가 더 안전합니다.

1. 목표 입력
2. 경로 계획
3. 계획 결과 확인
4. 사용자 승인
5. 실행

예제:

plan_result = arm.plan()

if not plan_result:
    logger.error("경로 계획 실패")
    return

answer = input(
    "계획이 성공했습니다. "
    "실행하려면 y 입력: "
)

if answer.lower() != "y":
    logger.info("실행을 취소했습니다.")
    return

robot.execute(
    plan_result.trajectory,
    controllers=[]
)
20. MoveIt Python과 MoveIt Servo 차이
MoveItPy 경로 계획
목표를 먼저 정함
→ 전체 경로 계산
→ 궤적 실행

적합한 작업:

Home 위치 이동
물체 집기 전 접근
정해진 관절 자세 이동
Pick & Place
장애물 회피 이동
MoveIt Servo
속도 또는 작은 위치 명령을 계속 전달
→ 실시간에 가깝게 움직임

적합한 작업:

조이스틱 제어
카메라 기반 추적
YOLO 물체 중심 추적
손 추적
실시간 미세 보정

MoveIt Servo는 관절 속도, 말단장치 속도, 목표 Pose 명령을 받아 실시간 제어에 사용할 수 있습니다.

사용자님의 향후 YOLO 응용에서는 다음과 같이 구분하는 것이 좋습니다.

컵 위치까지 큰 이동
    → MoveItPy

카메라 중심을 맞추는 작은 연속 이동
    → MoveIt Servo
21. 자주 발생하는 오류와 해결
오류 1
ModuleNotFoundError:
No module named 'moveit'

확인:

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

python3 -c \
"from moveit.planning import MoveItPy; print('OK')"

패키지 확인:

apt list --installed 2>/dev/null \
| grep moveit

설치가 필요하다면:

sudo apt update
sudo apt install ros-jazzy-moveit
오류 2
Could not find parameter robot_description

원인:

Python 노드에 URDF 파라미터가 전달되지 않음

해결:

ros2 run 대신 MoveIt 설정을 포함한
ros2 launch로 실행
오류 3
Group 'arm' was not found

원인:

SRDF 그룹 이름 불일치

확인:

grep '<group name=' \
~/omx_moveit_ws/src/*/config/omx_f/*.srdf
오류 4
Planning failed

가능한 원인:

목표가 작업 범위 밖
시작 상태가 충돌 중
목표 상태가 충돌 중
관절 제한 초과
IK 해 없음
잘못된 말단 링크
/joint_states 미수신
OMPL 설정 미전달
오류 5
Execution failed

가능한 원인:

arm_controller 비활성
FollowJointTrajectory Action 없음
컨트롤러 관절 이름 불일치
실제 모터 Torque OFF
USB 포트 오류
Dynamixel 통신 실패

확인:

ros2 control list_controllers
ros2 action list \
| grep follow_joint_trajectory

예상되는 Action:

/arm_controller/follow_joint_trajectory
오류 6
Could not contact service
/controller_manager/list_controllers

원인:

controller_manager가 실행되지 않았거나
ros2_control 하드웨어 초기화 실패

확인:

ros2 node list \
| grep controller_manager
ros2 service list \
| grep controller_manager
22. 14단계 권장 실습 순서
14-1. 환경 확인
python3 -c \
"from moveit.planning import MoveItPy; print('MoveItPy OK')"
14-2. /joint_states 확인
ros2 topic echo /joint_states --once
14-3. 컨트롤러 확인
ros2 control list_controllers
14-4. 저장 자세 이동
현재 상태 → home
14-5. 단일 관절 이동
joint1 현재값 + 0.02rad
14-6. 여러 관절 이동
joint1~joint5 목표값 설정
14-7. XYZ 위치 이동
말단장치 X, Y, Z 목표 설정
14-8. 장애물 추가
Planning Scene에 작업대 추가
14-9. 실제 OMX-F 이동
작은 이동량부터 테스트
14-10. 다음 단계 준비
Depth Camera 좌표
→ 로봇 기준 좌표 변환
→ MoveIt 목표 좌표
23. 14단계에서 반드시 이해해야 할 핵심
MoveItPy
= Python과 MoveIt을 연결하는 주 객체
PlanningComponent
= arm 또는 gripper 그룹의 계획 담당
RobotState
= 특정 순간의 전체 관절 상태
set_start_state_to_current_state()
= 현재 로봇 상태에서 계획 시작
set_goal_state()
= 목표 관절 또는 목표 Pose 지정
plan()
= 충돌 없는 경로 계산
execute()
= 계산한 궤적을 컨트롤러로 전송
Planning Scene
= 로봇과 장애물의 충돌 환경
24. 14단계 완성 기준

다음 항목이 모두 되면 14단계를 완료한 것입니다.

□ Python에서 MoveItPy 객체 생성

□ SRDF의 arm 그룹 불러오기

□ 현재 /joint_states 읽기

□ 현재 상태를 계획 시작점으로 설정

□ 저장 자세로 이동 계획

□ joint1을 상대적으로 0.02~0.05rad 이동

□ XYZ 목표 Pose 계획

□ 계획 성공 여부 처리

□ arm_controller로 궤적 실행

□ 실제 OMX-F에서 작은 이동 테스트

□ Planning Scene에 장애물 추가
다음 단계와 연결

14단계에서 Python으로 로봇팔을 움직일 수 있게 되면, 15단계의 Depth Camera에서 얻은 3차원 좌표를 목표로 사용할 수 있습니다.

15단계 Depth Camera
        ↓
카메라에서 물체의 X, Y, Z 측정
        ↓
카메라 좌표를 로봇 좌표로 변환
        ↓
MoveIt Python 목표 Pose 생성
        ↓
로봇팔 이동
        ↓
16단계 AI Pick & Place

즉, 14단계는 단순한 Python 연습이 아니라 AI가 인식한 물체를 로봇팔이 자동으로 집도록 만드는 핵심 연결 단계입니다.
