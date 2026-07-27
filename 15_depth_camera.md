AI 로봇 제작 마스터 클래스
15단계: Depth Camera — 깊이 카메라와 3차원 좌표 인식

15단계에서는 일반 웹캠의 2차원 영상 좌표를 넘어, 물체까지의 거리와 실제 3차원 위치를 측정하는 방법을 배웁니다.

지금까지 YOLO로 컵을 탐지하면 다음 정보는 얻을 수 있었습니다.

컵 중심 픽셀 좌표: x=320, y=240

하지만 로봇팔이 컵을 잡으려면 화면 좌표만으로는 부족합니다.

컵이 카메라에서 몇 mm 떨어져 있는가?
로봇 기준으로 왼쪽인가, 오른쪽인가?
로봇팔 끝단을 어느 X, Y, Z 위치로 이동해야 하는가?

Depth Camera를 사용하면 다음과 같은 3차원 위치를 얻을 수 있습니다.

X = 0.12 m
Y = -0.08 m
Z = 0.65 m

이 정보가 다음 단계인 16단계 AI Pick & Place의 핵심 입력이 됩니다.

1. Depth Camera란?

Depth Camera는 일반 컬러 영상과 함께 각 픽셀까지의 거리를 측정하는 카메라입니다.

일반 웹캠은 다음 데이터만 제공합니다.

RGB 이미지
너비 × 높이
픽셀 색상

Depth Camera는 다음 데이터를 함께 제공합니다.

컬러 이미지
깊이 이미지
카메라 내부 파라미터
3차원 포인트 클라우드

예를 들어 컬러 영상의 한 픽셀이 다음 위치라고 하겠습니다.

픽셀 위치: u=320, v=240

해당 픽셀의 Depth 값이 다음과 같다면:

Depth = 650 mm

컵이 카메라로부터 약 65cm 떨어져 있다는 뜻입니다.

2. 일반 카메라와 깊이 카메라의 차이
구분	일반 웹캠	Depth Camera
컬러 영상	가능	가능
객체 탐지	가능	가능
물체 거리 측정	직접 불가능	가능
3차원 좌표	직접 불가능	가능
포인트 클라우드	불가능	가능
로봇 Pick & Place	추가 계산 필요	매우 유리
장애물 인식	제한적	유리

일반 웹캠에서도 AI 기반 단안 깊이 추정은 가능하지만, 실제 거리 단위의 정밀한 로봇 제어에는 별도 보정이 필요합니다. 깊이 카메라는 픽셀별 거리 정보를 직접 제공하기 때문에 로봇 조작에 더 적합합니다.

3. 대표적인 Depth Camera

로봇 제작에서는 다음 제품군이 많이 사용됩니다.

Intel RealSense

대표 모델:

D415
D435
D435i
D455

특징:

RGB 영상
Depth 영상
포인트 클라우드
ROS 2 연동
Python 및 C++ SDK
일부 모델 IMU 포함

현재 RealSense ROS 2 Wrapper는 컬러와 깊이 스트림, 동기화, RGB 기준 깊이 정렬, RGB-D 토픽 등을 지원합니다.

로봇팔 실습에서는 보통 다음과 같이 선택할 수 있습니다.

기본 실습: RealSense D435 또는 D435i
거리 안정성과 넓은 작업공간: D455
가까운 거리와 비교적 정밀한 측정: D415 계열
Orbbec

대표 모델:

Astra
Gemini 2
Femto Bolt

ROS 2와 연동할 수 있으며, 모델별 최소 측정 거리와 지원 해상도를 확인해야 합니다.

Luxonis OAK-D

특징:

스테레오 Depth
RGB 카메라
장치 내부 AI 연산
YOLO 모델 실행 가능
DepthAI SDK 사용

AI 추론과 Depth를 카메라 장치 내부에서 처리하려는 경우 유용합니다.

4. Depth Camera의 거리 측정 방식

깊이 카메라는 제품에 따라 서로 다른 원리를 사용합니다.

4.1 스테레오 방식

사람의 두 눈처럼 좌우 카메라 두 개를 사용합니다.

왼쪽 카메라 영상
       +
오른쪽 카메라 영상
       ↓
두 영상의 위치 차이 계산
       ↓
거리 계산

물체가 가까우면 좌우 영상의 차이가 커지고, 멀면 차이가 작아집니다.

기본 관계는 다음과 같습니다.

Z = f × B / d
Z: 물체까지 거리
f: 카메라 초점거리
B: 두 카메라 사이 거리, Baseline
d: 좌우 영상의 시차, Disparity
4.2 구조광 방식

카메라가 특정 적외선 패턴을 물체에 투사하고, 패턴이 변형된 정도를 분석하여 거리를 측정합니다.

4.3 ToF 방식

빛을 발사한 뒤 반사되어 돌아오는 시간을 측정합니다.

거리 = 빛의 속도 × 왕복 시간 ÷ 2
5. Depth Image 이해하기

Depth Image는 일반 사진처럼 보이지만, 각 픽셀 값이 색상이 아니라 거리입니다.

예:

depth[240, 320] = 650

이 값의 의미가 mm 단위라면:

650 mm = 0.65 m

다만 카메라와 ROS 드라이버에 따라 단위와 인코딩이 달라질 수 있습니다.

ROS 2에서 흔히 볼 수 있는 Depth 인코딩:

16UC1
32FC1
16UC1
16비트 부호 없는 정수
1채널
일반적으로 mm 단위

예:

650 → 650 mm
32FC1
32비트 실수
1채널
일반적으로 m 단위

예:

0.65 → 0.65 m

실제 프로그램에서는 반드시 메시지의 encoding과 카메라 드라이버 설정을 확인해야 합니다.

6. 컬러 영상과 Depth 영상 정렬

Depth Camera의 RGB 센서와 Depth 센서는 물리적으로 다른 위치에 있습니다.

따라서 원본 영상에서는 같은 픽셀 좌표가 서로 다른 공간을 가리킬 수 있습니다.

RGB 이미지의 (320, 240)
Depth 이미지의 (320, 240)

두 좌표가 반드시 같은 물체를 의미하지는 않습니다.

이를 해결하는 작업이:

Depth Alignment
깊이 영상 정렬

입니다.

RealSense ROS 2 Wrapper에서는 다음 옵션을 사용할 수 있습니다.

align_depth.enable:=true

이 기능은 Depth 영상을 컬러 카메라 좌표계에 맞춰 정렬합니다. RealSense Wrapper는 align_depth.enable, 컬러·깊이 동기화 및 RGB-D 스트림 관련 옵션을 제공합니다.

YOLO와 Depth를 함께 사용할 때는 일반적으로 다음 조합을 사용합니다.

YOLO 입력:
컬러 이미지

거리 측정:
컬러 영상에 정렬된 Depth 이미지
7. 픽셀 좌표를 3차원 좌표로 변환하기

YOLO가 컵의 중심점을 찾았다고 가정하겠습니다.

u = 350
v = 260
depth = 0.70 m

이 픽셀을 카메라 기준 3차원 좌표로 변환하려면 카메라 내부 파라미터가 필요합니다.

fx
fy
cx
cy
fx: X 방향 초점거리
fy: Y 방향 초점거리
cx: 영상 중심점 X
cy: 영상 중심점 Y

변환식은 다음과 같습니다.

Z = depth

X = (u - cx) × Z / fx

Y = (v - cy) × Z / fy

예를 들어:

u = 350
v = 260
Z = 0.70

fx = 600
fy = 600
cx = 320
cy = 240

계산하면:

X = (350 - 320) × 0.70 / 600
  = 0.035 m

Y = (260 - 240) × 0.70 / 600
  = 0.0233 m

Z = 0.70 m

결과:

카메라 기준 물체 위치

X = 0.035 m
Y = 0.023 m
Z = 0.700 m

즉 카메라 기준으로:

오른쪽 약 3.5cm
아래쪽 약 2.3cm
앞쪽 약 70cm

에 물체가 있습니다.

8. 카메라 좌표계 이해하기

일반적인 광학 카메라 좌표계는 다음과 같습니다.

          +Z
       카메라 앞쪽

          ↑
          |
          |
카메라 ●────────→ +X
          \
           \
            ↓ +Y

보통 ROS 광학 프레임은:

X: 영상 오른쪽
Y: 영상 아래쪽
Z: 카메라 전방

입니다.

그러나 로봇의 base_link 좌표계는 보통 다음과 다릅니다.

X: 로봇 전방
Y: 로봇 왼쪽
Z: 위쪽

따라서 카메라 좌표를 그대로 MoveIt 목표 위치에 넣으면 로봇이 엉뚱한 방향으로 움직일 수 있습니다.

반드시 TF 변환이 필요합니다.

9. 카메라 좌표와 로봇 좌표 변환

전체 흐름은 다음과 같습니다.

YOLO 객체 탐지
        ↓
픽셀 중심 좌표
        ↓
Depth 값 읽기
        ↓
카메라 3차원 좌표
        ↓
TF2 좌표 변환
        ↓
로봇 base_link 기준 좌표
        ↓
MoveIt 목표 위치 전달

예를 들어 카메라 좌표가 다음과 같다고 하겠습니다.

camera_color_optical_frame

X = 0.03
Y = 0.02
Z = 0.70

TF2를 사용해 다음 좌표계로 변환합니다.

base_link

변환 후:

base_link 기준

X = 0.42
Y = -0.08
Z = 0.15

이 좌표를 로봇팔의 목표 위치로 사용할 수 있습니다.

10. 카메라 설치 방법

로봇팔에서 Depth Camera를 설치하는 방식은 크게 두 가지입니다.

방법 1: Eye-to-Hand

카메라를 로봇 외부에 고정합니다.

Depth Camera
      ↓
  작업 테이블
      ↑
    로봇팔

장점:

카메라 화면이 안정적
작업영역 전체를 볼 수 있음
배선이 간단함
초기 실습에 적합

단점:

카메라와 로봇 사이의 위치 관계를 정확히 측정해야 함
로봇팔이 물체를 가릴 수 있음

추천:

처음 Depth Camera를 배우는 단계
고정된 테이블에서 Pick & Place
방법 2: Eye-in-Hand

카메라를 로봇팔 끝단에 장착합니다.

로봇팔 끝단
   ├─ 그리퍼
   └─ Depth Camera

장점:

물체를 가까이에서 볼 수 있음
시야를 로봇이 직접 변경할 수 있음
정밀 작업에 유리

단점:

카메라 무게가 로봇 끝단에 추가됨
배선이 복잡함
Hand-Eye Calibration 필요
로봇이 움직일 때마다 카메라 좌표가 바뀜

15단계 초기 실습에서는 Eye-to-Hand 방식을 권장합니다.

11. 권장 실습 구성

현재 사용 중인 환경을 기준으로 구성하면 다음과 같습니다.

Ubuntu 24.04
ROS 2 Jazzy
Python 3
OpenCV
YOLO
MoveIt 2
ROBOTIS OMX-F
Depth Camera

권장 연결 구조:

Depth Camera
   ├─ RGB 영상 ─→ YOLO 객체 탐지
   ├─ Depth 영상 ─→ 거리 측정
   └─ CameraInfo ─→ 3차원 좌표 계산

YOLO + Depth
   ↓
카메라 기준 물체 좌표
   ↓
TF2
   ↓
OMX-F base_link 좌표
   ↓
MoveIt 2
   ↓
로봇팔 이동
12. RealSense 기본 프로그램 설치

아래 예시는 RealSense 계열을 기준으로 합니다.

먼저 ROS 2 환경을 불러옵니다.

source /opt/ros/jazzy/setup.bash

시스템 패키지를 업데이트합니다.

sudo apt update

RealSense 관련 패키지가 Ubuntu 저장소에 제공되는지 확인합니다.

apt search realsense2

ROS 패키지를 검색합니다.

apt search ros-jazzy-realsense

패키지가 제공된다면 설치합니다.

sudo apt install ros-jazzy-realsense2-camera
sudo apt install ros-jazzy-realsense2-description

설치 여부 확인:

ros2 pkg list | grep realsense

예상 출력:

realsense2_camera
realsense2_camera_msgs
realsense2_description

배포판 저장소에 원하는 버전이 없다면 공식 RealSense ROS Wrapper 저장소를 소스 빌드하는 방법을 사용할 수 있습니다. 공식 Wrapper는 ROS 2용 카메라 노드를 제공합니다.

13. 카메라 USB 연결 확인

카메라를 USB 3 포트에 연결합니다.

lsusb

RealSense 카메라라면 제품명이 표시됩니다.

USB 연결 속도 확인:

lsusb -t

다음과 같이 5000M 이상으로 표시되는 것이 좋습니다.

5000M

480M으로 표시되면 USB 2로 연결되었을 가능성이 있습니다.

이 경우:

Depth 해상도 제한
프레임 드롭
영상 끊김
카메라 실행 실패

등이 발생할 수 있습니다.

14. RealSense Viewer 실행

SDK 도구가 설치되어 있다면 다음을 실행합니다.

realsense-viewer

확인할 항목:

Color 영상 출력
Depth 영상 출력
카메라 모델명
USB 연결 방식
Depth Scale
해상도
FPS

처음에는 다음 설정이 무난합니다.

Color: 640 × 480, 30 FPS
Depth: 640 × 480, 30 FPS

PC 부하가 크면:

640 × 480, 15 FPS

또는:

424 × 240, 30 FPS

로 낮춥니다.

15. ROS 2에서 Depth Camera 실행

기본 실행 예:

source /opt/ros/jazzy/setup.bash

ros2 launch realsense2_camera rs_launch.py

컬러와 Depth를 활성화하고 Depth를 컬러 영상에 정렬하려면:

ros2 launch realsense2_camera rs_launch.py \
  enable_color:=true \
  enable_depth:=true \
  align_depth.enable:=true

프레임 동기화까지 활성화하려면:

ros2 launch realsense2_camera rs_launch.py \
  enable_color:=true \
  enable_depth:=true \
  align_depth.enable:=true \
  enable_sync:=true

RealSense ROS Wrapper는 컬러와 깊이 활성화, 동기화 및 정렬 기능을 지원합니다.

16. ROS 2 토픽 확인

새 터미널을 열고:

source /opt/ros/jazzy/setup.bash

토픽 목록을 확인합니다.

ros2 topic list

대표적인 토픽은 다음과 같습니다.

/camera/camera/color/image_raw
/camera/camera/color/camera_info
/camera/camera/depth/image_rect_raw
/camera/camera/depth/camera_info
/camera/camera/aligned_depth_to_color/image_raw
/camera/camera/depth/color/points

설치 버전과 네임스페이스 설정에 따라 토픽 이름은 달라질 수 있습니다.

따라서 항상 실제 목록을 확인합니다.

ros2 topic list | grep camera
17. Depth 메시지 정보 확인
ros2 topic info \
  /camera/camera/aligned_depth_to_color/image_raw

메시지 타입 확인:

sensor_msgs/msg/Image

한 번만 출력:

ros2 topic echo \
  /camera/camera/aligned_depth_to_color/image_raw \
  --once

전체 픽셀 데이터가 매우 길게 출력되므로 주로 다음 항목만 확인합니다.

header
height
width
encoding
step

예상 예:

height: 480
width: 640
encoding: 16UC1
18. CameraInfo 확인

카메라 내부 파라미터는 CameraInfo 토픽에서 확인합니다.

ros2 topic echo \
  /camera/camera/color/camera_info \
  --once

중요한 값:

k:
- fx
- 0
- cx
- 0
- fy
- cy
- 0
- 0
- 1

ROS에서는 행렬 K가 다음 형식입니다.

K = [fx, 0,  cx,
     0,  fy, cy,
     0,  0,  1]

Python에서는:

fx = msg.k[0]
fy = msg.k[4]
cx = msg.k[2]
cy = msg.k[5]

로 읽습니다.

19. RViz2에서 Depth 영상 보기

RViz2를 실행합니다.

rviz2

왼쪽 아래에서:

Add
→ By topic
→ Image

컬러 영상:

/camera/camera/color/image_raw

정렬된 Depth 영상:

/camera/camera/aligned_depth_to_color/image_raw

포인트 클라우드:

/camera/camera/depth/color/points

PointCloud2를 표시할 때 Fixed Frame을 카메라 프레임으로 설정합니다.

예:

camera_link

또는:

camera_depth_optical_frame

프레임 이름 확인:

ros2 topic echo /tf_static --once

또는:

ros2 run tf2_tools view_frames
20. Point Cloud란?

Point Cloud는 3차원 공간의 점 집합입니다.

각 점은 다음 좌표를 갖습니다.

X, Y, Z

컬러 정보가 포함되면:

X, Y, Z, R, G, B

형태가 됩니다.

예:

Point 1: X=0.10, Y=0.02, Z=0.65
Point 2: X=0.11, Y=0.02, Z=0.65
Point 3: X=0.12, Y=0.03, Z=0.66

많은 점이 모이면 테이블, 컵, 로봇팔 등의 3차원 형태가 만들어집니다.

ROS 2의 depth_image_proc는 Depth 영상 처리, 포인트 클라우드 생성 및 다른 카메라 프레임으로의 깊이 등록 기능을 제공합니다.

21. Python으로 중앙 픽셀 거리 읽기

ROS 2 Python 패키지를 만들어 보겠습니다.

작업공간 이동
cd ~/omx_moveit_ws/src

패키지 생성:

ros2 pkg create depth_camera_tutorial \
  --build-type ament_python \
  --dependencies \
  rclpy \
  sensor_msgs \
  cv_bridge

Python 파일 생성:

cd ~/omx_moveit_ws/src/depth_camera_tutorial
mkdir -p depth_camera_tutorial
nano depth_camera_tutorial/depth_center_reader.py

다음 코드를 입력합니다.

#!/usr/bin/env python3

import cv2
import numpy as np
import rclpy

from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge


class DepthCenterReader(Node):
    """Depth 영상의 중앙 픽셀 거리를 읽는 ROS 2 노드."""

    def __init__(self):
        super().__init__("depth_center_reader")

        self.bridge = CvBridge()

        self.depth_sub = self.create_subscription(
            Image,
            "/camera/camera/aligned_depth_to_color/image_raw",
            self.depth_callback,
            10,
        )

        self.get_logger().info("Depth 중앙거리 측정 노드 시작")

    def depth_callback(self, msg: Image):
        try:
            depth_image = self.bridge.imgmsg_to_cv2(
                msg,
                desired_encoding="passthrough",
            )
        except Exception as error:
            self.get_logger().error(f"Depth 변환 실패: {error}")
            return

        height, width = depth_image.shape[:2]

        center_x = width // 2
        center_y = height // 2

        raw_depth = depth_image[center_y, center_x]

        if msg.encoding == "16UC1":
            depth_m = float(raw_depth) / 1000.0
        elif msg.encoding == "32FC1":
            depth_m = float(raw_depth)
        else:
            self.get_logger().warning(
                f"확인되지 않은 Depth 인코딩: {msg.encoding}"
            )
            return

        if not np.isfinite(depth_m) or depth_m <= 0.0:
            self.get_logger().warning("중앙 픽셀의 Depth 값이 유효하지 않습니다.")
            return

        self.get_logger().info(
            f"중앙 픽셀 ({center_x}, {center_y}) "
            f"거리: {depth_m:.3f} m"
        )


def main(args=None):
    rclpy.init(args=args)

    node = DepthCenterReader()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == "__main__":
    main()
22. setup.py 등록
nano ~/omx_moveit_ws/src/depth_camera_tutorial/setup.py

entry_points 부분에 다음을 등록합니다.

entry_points={
    "console_scripts": [
        "depth_center_reader = "
        "depth_camera_tutorial.depth_center_reader:main",
    ],
},

실행 권한 추가:

chmod +x \
  ~/omx_moveit_ws/src/depth_camera_tutorial/depth_camera_tutorial/depth_center_reader.py
23. 패키지 빌드
cd ~/omx_moveit_ws

ROS 환경 불러오기:

source /opt/ros/jazzy/setup.bash

빌드:

colcon build \
  --packages-select depth_camera_tutorial \
  --symlink-install

환경 적용:

source ~/omx_moveit_ws/install/setup.bash
24. 중앙거리 측정 실행

터미널 1:

source /opt/ros/jazzy/setup.bash

ros2 launch realsense2_camera rs_launch.py \
  enable_color:=true \
  enable_depth:=true \
  align_depth.enable:=true

터미널 2:

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

ros2 run depth_camera_tutorial depth_center_reader

예상 출력:

[INFO] 중앙 픽셀 (320, 240) 거리: 0.654 m
[INFO] 중앙 픽셀 (320, 240) 거리: 0.653 m
[INFO] 중앙 픽셀 (320, 240) 거리: 0.655 m
25. Depth 값을 컬러 영상으로 표시하기

Depth 영상의 실제 값은 사람이 보기 어렵습니다.

이를 컬러맵으로 변환할 수 있습니다.

depth_display = cv2.convertScaleAbs(
    depth_image,
    alpha=0.03,
)

depth_colormap = cv2.applyColorMap(
    depth_display,
    cv2.COLORMAP_JET,
)

cv2.imshow("Depth", depth_colormap)
cv2.waitKey(1)

주의:

컬러맵은 시각화를 위한 색상입니다.
컬러맵 픽셀값을 실제 거리로 사용하면 안 됩니다.

실제 거리 계산은 원본 Depth 배열을 사용해야 합니다.

26. YOLO 탐지 중심점의 Depth 읽기

YOLO가 컵을 탐지했다고 가정하겠습니다.

Bounding Box:

x1 = 250
y1 = 180
x2 = 390
y2 = 340

중심점:

center_x = int((x1 + x2) / 2)
center_y = int((y1 + y2) / 2)

Depth 읽기:

raw_depth = depth_image[center_y, center_x]
depth_m = float(raw_depth) / 1000.0

전체 흐름:

for box in result.boxes:
    x1, y1, x2, y2 = map(
        int,
        box.xyxy[0].tolist(),
    )

    center_x = (x1 + x2) // 2
    center_y = (y1 + y2) // 2

    raw_depth = depth_image[center_y, center_x]

    if raw_depth > 0:
        depth_m = float(raw_depth) / 1000.0

        print(
            f"컵 중심=({center_x}, {center_y}), "
            f"거리={depth_m:.3f}m"
        )
27. 중심 픽셀 하나만 읽으면 생기는 문제

물체 중심점의 Depth 값이 항상 정확하지는 않습니다.

예를 들어:

컵이 투명함
컵 가운데가 비어 있음
반사가 강함
중심점에 Depth 구멍이 있음
Bounding Box 중심이 배경에 위치함

이 경우 Depth 값이 다음처럼 나올 수 있습니다.

0
NaN
비정상적으로 먼 거리

그래서 중심 주변 영역의 중앙값을 사용하는 것이 좋습니다.

half_size = 5

x_start = max(center_x - half_size, 0)
x_end = min(center_x + half_size + 1, depth_image.shape[1])

y_start = max(center_y - half_size, 0)
y_end = min(center_y + half_size + 1, depth_image.shape[0])

depth_region = depth_image[
    y_start:y_end,
    x_start:x_end,
]

valid_values = depth_region[
    (depth_region > 0)
]

if valid_values.size > 0:
    depth_mm = float(np.median(valid_values))
    depth_m = depth_mm / 1000.0

중앙값은 단일 픽셀보다 노이즈와 이상치에 강합니다.

28. 픽셀을 3차원 좌표로 변환하는 ROS 2 노드

필요한 입력:

Depth Image
CameraInfo
검출 중심점 u, v

핵심 함수는 다음과 같습니다.

def pixel_to_camera_point(
    u: int,
    v: int,
    depth_m: float,
    fx: float,
    fy: float,
    cx: float,
    cy: float,
):
    x = (u - cx) * depth_m / fx
    y = (v - cy) * depth_m / fy
    z = depth_m

    return x, y, z

사용 예:

camera_x, camera_y, camera_z = pixel_to_camera_point(
    center_x,
    center_y,
    depth_m,
    self.fx,
    self.fy,
    self.cx,
    self.cy,
)

self.get_logger().info(
    f"카메라 좌표: "
    f"X={camera_x:.3f}, "
    f"Y={camera_y:.3f}, "
    f"Z={camera_z:.3f}"
)
29. CameraInfo 저장 코드
from sensor_msgs.msg import CameraInfo

구독자:

self.camera_info_sub = self.create_subscription(
    CameraInfo,
    "/camera/camera/color/camera_info",
    self.camera_info_callback,
    10,
)

콜백:

def camera_info_callback(self, msg: CameraInfo):
    self.fx = msg.k[0]
    self.fy = msg.k[4]
    self.cx = msg.k[2]
    self.cy = msg.k[5]

    self.camera_frame = msg.header.frame_id

초깃값:

self.fx = None
self.fy = None
self.cx = None
self.cy = None
self.camera_frame = None

계산 전 확인:

if self.fx is None:
    self.get_logger().warning(
        "CameraInfo를 아직 받지 못했습니다."
    )
    return
30. 3차원 좌표를 PointStamped로 발행하기

로봇과 연동하려면 단순한 숫자보다 ROS 메시지로 발행하는 것이 좋습니다.

from geometry_msgs.msg import PointStamped

Publisher 생성:

self.point_pub = self.create_publisher(
    PointStamped,
    "/detected_object/camera_point",
    10,
)

메시지 생성:

point_msg = PointStamped()

point_msg.header.stamp = self.get_clock().now().to_msg()
point_msg.header.frame_id = self.camera_frame

point_msg.point.x = camera_x
point_msg.point.y = camera_y
point_msg.point.z = camera_z

self.point_pub.publish(point_msg)

확인:

ros2 topic echo /detected_object/camera_point

예상 결과:

header:
  frame_id: camera_color_optical_frame
point:
  x: 0.035
  y: 0.023
  z: 0.700
31. TF2로 base_link 좌표 변환

카메라 기준 좌표를 base_link 기준으로 변환합니다.

필요한 패키지:

tf2_ros
tf2_geometry_msgs
geometry_msgs

패키지 의존성 추가:

cd ~/omx_moveit_ws/src/depth_camera_tutorial

rosdep install \
  --from-paths . \
  --ignore-src \
  -r \
  -y

Python 주요 코드:

from tf2_ros import Buffer
from tf2_ros import TransformListener
from tf2_ros import TransformException

초기화:

self.tf_buffer = Buffer()
self.tf_listener = TransformListener(
    self.tf_buffer,
    self,
)

변환:

try:
    transformed_point = self.tf_buffer.transform(
        point_msg,
        "base_link",
        timeout=rclpy.duration.Duration(seconds=0.5),
    )

    self.get_logger().info(
        f"base_link 좌표: "
        f"X={transformed_point.point.x:.3f}, "
        f"Y={transformed_point.point.y:.3f}, "
        f"Z={transformed_point.point.z:.3f}"
    )

except TransformException as error:
    self.get_logger().warning(
        f"TF 변환 실패: {error}"
    )
32. 카메라 고정 TF 등록

Eye-to-Hand 방식이라면 카메라와 로봇 베이스 사이의 위치가 고정되어 있습니다.

예를 들어 카메라가 로봇 기준:

X = 0.20m
Y = 0.00m
Z = 0.65m

위치에 있다고 가정합니다.

테스트용 Static Transform:

ros2 run tf2_ros static_transform_publisher \
  --x 0.20 \
  --y 0.00 \
  --z 0.65 \
  --roll 0.0 \
  --pitch 0.0 \
  --yaw 0.0 \
  --frame-id base_link \
  --child-frame-id camera_link

하지만 이것은 단순 예시입니다.

실제로는 카메라의 위치뿐 아니라 회전도 정확히 측정해야 합니다.

Translation:
X, Y, Z

Rotation:
Roll, Pitch, Yaw

카메라가 아래쪽을 바라보고 있다면 회전값이 반드시 들어가야 합니다.

33. TF 연결 확인
ros2 run tf2_ros tf2_echo \
  base_link \
  camera_color_optical_frame

정상이라면 Translation과 Rotation이 계속 출력됩니다.

Translation:
x: ...
y: ...
z: ...

Rotation:
x: ...
y: ...
z: ...
w: ...

TF 트리 생성:

ros2 run tf2_tools view_frames

생성된 파일:

frames.pdf

TF 경로는 다음처럼 연결되어야 합니다.

base_link
   ↓
camera_link
   ↓
camera_color_frame
   ↓
camera_color_optical_frame
34. MoveIt과 연결되는 최종 좌표

TF 변환 후 얻은 좌표가 다음과 같다고 하겠습니다.

base_link 기준

X = 0.30 m
Y = -0.08 m
Z = 0.12 m

로봇팔이 바로 물체 중심으로 이동하면 충돌할 수 있으므로 접근 위치를 별도로 만듭니다.

object_x = 0.30
object_y = -0.08
object_z = 0.12

approach_x = object_x
approach_y = object_y
approach_z = object_z + 0.10

즉:

물체 위 10cm 위치로 이동
       ↓
수직으로 내려감
       ↓
그리퍼 닫기
       ↓
다시 위로 이동

이 흐름이 Pick 동작입니다.

35. Depth Camera 기반 Pick & Place 전체 과정
1. 카메라 실행
2. RGB 영상 수신
3. YOLO로 컵 탐지
4. Bounding Box 중심 계산
5. 정렬된 Depth 영상에서 거리 읽기
6. CameraInfo로 3차원 좌표 계산
7. TF2로 base_link 좌표 변환
8. 작업영역 안인지 확인
9. 접근 위치 생성
10. MoveIt으로 경로 계획
11. 접근 위치로 이동
12. 컵 위치로 하강
13. 그리퍼 닫기
14. 물체 들어 올리기
15. 지정 위치로 이동
16. 그리퍼 열기
36. 반드시 추가해야 하는 안전 조건

Depth 좌표를 로봇에 바로 전달하면 위험합니다.

다음 제한을 반드시 적용해야 합니다.

MIN_X = 0.10
MAX_X = 0.45

MIN_Y = -0.30
MAX_Y = 0.30

MIN_Z = 0.03
MAX_Z = 0.40

검사:

def is_inside_workspace(x, y, z):
    return (
        MIN_X <= x <= MAX_X
        and MIN_Y <= y <= MAX_Y
        and MIN_Z <= z <= MAX_Z
    )

사용:

if not is_inside_workspace(
    target_x,
    target_y,
    target_z,
):
    self.get_logger().warning(
        "탐지 좌표가 로봇 작업영역 밖입니다."
    )
    return

추가 안전 조건:

Depth 값이 0이면 이동 금지
Depth가 너무 가까우면 이동 금지
Depth가 너무 멀면 이동 금지
TF 변환 실패 시 이동 금지
객체가 일정 시간 안정적으로 탐지된 후 이동
한 번의 탐지 결과만으로 움직이지 않기
비상정지 버튼 준비
저속으로 시험하기
37. Depth 값 안정화 방법

Depth 값은 프레임마다 흔들릴 수 있습니다.

예:

0.651
0.647
0.655
0.649
0.652

이동평균을 사용할 수 있습니다.

from collections import deque
import numpy as np

self.depth_history = deque(maxlen=10)

새 값 저장:

self.depth_history.append(depth_m)

중앙값 계산:

filtered_depth = float(
    np.median(self.depth_history)
)

물체가 안정적으로 정지했는지 확인:

depth_std = float(
    np.std(self.depth_history)
)

if depth_std < 0.005:
    print("Depth 값이 안정적입니다.")
38. 투명하고 반사되는 물체 문제

Depth Camera는 다음 물체에서 측정 오류가 발생하기 쉽습니다.

투명한 컵
유리
거울
광택 금속
검은색 흡광 소재
햇빛이 강한 환경

흔한 결과:

Depth = 0
Depth에 구멍 발생
배경 거리로 측정
값이 크게 흔들림

해결 방법:

불투명한 물체로 먼저 실습
무광 컵 사용
조명 방향 조정
카메라 각도 변경
주변 Depth 중앙값 사용
여러 프레임 중앙값 사용
포인트 클라우드 기반 평면 제거
컬러 탐지 결과와 Depth 마스크 결합

초기 실습 물체로는 다음이 좋습니다.

무광 플라스틱 컵
종이 박스
색깔 블록
작은 상자
39. 테이블 평면과 물체 구분

Depth 영상에는 컵뿐 아니라 테이블도 포함됩니다.

예:

컵 상단 거리: 0.62m
테이블 거리: 0.70m

Depth 차이:

0.70 - 0.62 = 0.08m

즉 컵 높이가 약 8cm라는 것을 추정할 수 있습니다.

더 전문적인 방식은 포인트 클라우드에서 테이블 평면을 제거하는 것입니다.

포인트 클라우드 입력
       ↓
RANSAC 평면 검출
       ↓
테이블 점 제거
       ↓
남은 점들을 군집화
       ↓
물체 위치 계산

향후 PCL 또는 Open3D를 이용해 구현할 수 있습니다.

40. 15단계 권장 실습 순서
실습 1: 카메라 연결

목표:

USB 장치 확인
RealSense Viewer 실행
RGB와 Depth 영상 확인
실습 2: ROS 2 토픽 확인

목표:

컬러 토픽 확인
Depth 토픽 확인
CameraInfo 확인
PointCloud2 확인
실습 3: RViz2 시각화

목표:

컬러 이미지 표시
Depth 이미지 표시
포인트 클라우드 표시
TF 프레임 확인
실습 4: 중앙 거리 측정

목표:

화면 중앙 픽셀의 거리를 m 단위로 출력
실습 5: 마우스로 선택한 위치 거리 측정

목표:

사용자가 클릭한 픽셀의 Depth 출력
실습 6: YOLO와 결합

목표:

컵 중심 좌표 검출
컵까지 거리 출력
실습 7: 3차원 카메라 좌표 계산

목표:

픽셀 좌표와 Depth를 X, Y, Z로 변환
실습 8: PointStamped 발행

목표:

/detected_object/camera_point

토픽으로 좌표 발행

실습 9: TF2 좌표 변환

목표:

camera frame
→ base_link

변환

실습 10: MoveIt 목표 위치 표시

목표:

로봇은 움직이지 않고
RViz Marker로 목표점만 표시
실습 11: OMX-F 저속 접근

목표:

물체 위 10cm 접근 위치로만 이동
실습 12: Pick & Place 준비

목표:

접근
하강
그리퍼 닫기
상승

동작 구성

41. 15단계에서 만들어야 할 파일 구성

GitHub 프로젝트에서는 다음과 같이 정리할 수 있습니다.

robot-making/
├── README.md
├── 01_robot_basics.md
├── 02_electronics_basics.md
├── 03_arduino_basics.md
├── 04_motor_basics.md
├── 05_sensor_input.md
├── 06_python_basics.md
├── 07_opencv_basics.md
├── 08_yolo_object_detection.md
├── 09_mechanical_design.md
├── 10_robot_arm_build.md
├── 11_ros2_basics.md
├── 12_urdf_xacro.md
├── 13_moveit_basics.md
├── 14_moveit_python_api.md
└── 15_depth_camera.md

ROS 2 실습 패키지는:

omx_moveit_ws/
└── src/
    └── depth_camera_tutorial/
        ├── package.xml
        ├── setup.py
        ├── resource/
        └── depth_camera_tutorial/
            ├── __init__.py
            ├── depth_center_reader.py
            ├── pixel_to_3d.py
            ├── yolo_depth_detector.py
            └── camera_to_robot_tf.py

형태로 구성하면 좋습니다.

42. 15단계 완료 기준

다음 항목을 모두 수행하면 15단계를 완료한 것입니다.

[ ] Depth Camera 연결 확인
[ ] RGB 영상 출력
[ ] Depth 영상 출력
[ ] ROS 2 카메라 토픽 확인
[ ] CameraInfo 확인
[ ] RViz2에서 포인트 클라우드 출력
[ ] 중앙 픽셀 거리 측정
[ ] YOLO 탐지 중심의 Depth 측정
[ ] 픽셀 좌표를 카메라 X, Y, Z로 변환
[ ] PointStamped 메시지 발행
[ ] TF2로 base_link 좌표 변환
[ ] 로봇 작업영역 검사
[ ] MoveIt 목표 좌표로 사용할 준비 완료
43. 핵심 정리

15단계의 핵심은 다음 한 줄입니다.

2차원 객체 탐지 결과를 로봇이 사용할 수 있는 3차원 좌표로 바꾸는 단계

데이터 흐름은 다음과 같습니다.

RGB 영상
  ↓
YOLO 탐지
  ↓
픽셀 좌표 u, v
  ↓
Depth 값 Z
  ↓
카메라 내부 파라미터
  ↓
카메라 좌표 X, Y, Z
  ↓
TF2 좌표 변환
  ↓
로봇 base_link 좌표
  ↓
MoveIt 목표 위치

15단계를 완료하면 다음 단계인 16단계 AI Pick & Place에서 로봇팔이 카메라로 컵을 찾고, 실제 위치를 계산해 자동으로 집는 시스템을 만들 수 있습니다.
