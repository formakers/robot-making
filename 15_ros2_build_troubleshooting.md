ROS2 MoveIt 빌드 오류 해결 과정 요약
1단계. 빌드 실패 확인

빌드 실행

cd ~/omx_moveit_ws

colcon build --symlink-install

오류 발생

Failed <<< open_manipulator_moveit_config
2단계. 상세 오류 확인
colcon build \
  --symlink-install \
  --packages-select open_manipulator_moveit_config \
  --event-handlers console_direct+

또는

cat ~/omx_moveit_ws/log/latest_build/open_manipulator_moveit_config/stderr.log

실제 원인 발견

ModuleNotFoundError: No module named 'catkin_pkg'
3단계. 원인 분석

빌드 로그를 보면

/home/formakers/miniforge3/bin/python3

를 사용하고 있었습니다.

즉

ROS2가 Ubuntu Python이 아니라 Conda Python으로 실행되고 있었던 것입니다.

ROS2는 Ubuntu Python을 기준으로 설치되어 있으므로

catkin_pkg
ament
colcon

등의 패키지를 찾지 못했습니다.

4단계. Conda 종료
conda deactivate

필요하면 한 번 더

conda deactivate
5단계. Python 확인
which python3

잘못된 경우

/home/formakers/miniforge3/bin/python3

정상

/usr/bin/python3
6단계. 필요한 패키지 설치
sudo apt update

sudo apt install -y \
python3-catkin-pkg \
python3-rosdep \
python3-colcon-common-extensions
7단계. ROS 환경 설정
source /opt/ros/jazzy/setup.bash
8단계. 기존 빌드 삭제
rm -rf build/open_manipulator_moveit_config

rm -rf install/open_manipulator_moveit_config
9단계. 패키지만 다시 빌드
colcon build \
--symlink-install \
--packages-select open_manipulator_moveit_config
10단계. 성공
Finished <<< open_manipulator_moveit_config
11단계. 환경 적용
source ~/omx_moveit_ws/install/setup.bash
12단계. Conda 자동 실행 끄기 (권장)

매번 (base)가 실행되지 않도록 설정

conda config --set auto_activate_base false

새 터미널부터는

(base)

가 나타나지 않습니다.

핵심 원인
Conda(base)

↓

python3

↓

/home/formakers/miniforge3/bin/python3

↓

ROS2가 Ubuntu Python 패키지를 찾지 못함

↓

ModuleNotFoundError

↓

빌드 실패
해결 흐름
Conda 종료
      │
      ▼
Ubuntu Python 사용
      │
      ▼
catkin_pkg 정상 인식
      │
      ▼
ROS2 Build 성공
      │
      ▼
MoveIt 정상 빌드
앞으로 ROS2 작업 시 권장 순서

매번 새 터미널에서 아래 순서대로 시작하는 것을 권장합니다.

# 1. 작업 공간으로 이동
cd ~/omx_moveit_ws

# 2. ROS2 환경 설정
source /opt/ros/jazzy/setup.bash

# 3. (빌드 후에만) 워크스페이스 환경 설정
source ~/omx_moveit_ws/install/setup.bash

# 4. 작업 또는 빌드
colcon build --symlink-install

팁: AI/딥러닝 프로젝트는 Conda 환경에서 작업하고, ROS 2/MoveIt 개발은 시스템 Python(/usr/bin/python3) 환경에서 작업하면 Python 충돌을 크게 줄일 수 있습니다.
