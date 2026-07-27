16단계 — AI Pick & Place
카메라로 물체를 인식하고 로봇팔이 집어서 옮기기

16단계에서는 지금까지 학습한 YOLO 객체 인식, Depth Camera, ROS 2, MoveIt 2, 로봇팔 제어를 하나의 시스템으로 통합합니다.

최종 목표는 다음과 같습니다.

카메라가 컵이나 부품을 발견하면 물체의 3차원 위치를 계산하고, 로봇팔이 해당 위치로 이동해 물체를 집은 뒤 지정된 장소에 내려놓는다.

1. AI Pick & Place란?

Pick & Place는 로봇이 물체를 집어서 다른 위치로 이동시키는 작업입니다.

기본 동작은 다음 순서로 이루어집니다.

물체 탐지
   ↓
물체 중심 좌표 계산
   ↓
카메라 깊이값 측정
   ↓
3차원 좌표 계산
   ↓
로봇 좌표계로 변환
   ↓
로봇팔 이동 경로 계획
   ↓
그리퍼로 물체 잡기
   ↓
목표 위치로 이동
   ↓
그리퍼 열기

단순히 물체를 인식하는 것만으로는 부족합니다.

로봇은 물체가 화면 어디에 있는지뿐 아니라 실제 공간에서 다음 값을 알아야 합니다.

X: 좌우 위치
Y: 앞뒤 위치
Z: 높이
2. 전체 시스템 구성

AI Pick & Place 시스템은 크게 다섯 부분으로 구성됩니다.

2.1 카메라

RGB 영상과 깊이 정보를 얻습니다.

예:

Intel RealSense D435, D435i
Orbbec Depth Camera
ZED Camera
일반 웹캠과 별도 거리 센서

권장 구성은 RGB-D 카메라입니다.

RGB-D 카메라는 한 번에 다음 정보를 제공합니다.

컬러 영상
깊이 영상
카메라 내부 파라미터
2.2 객체 인식 AI

카메라 영상에서 물체를 찾습니다.

예:

YOLOv8
YOLO11
Detectron2
Segment Anything
자체 학습 모델

YOLO가 물체를 발견하면 일반적으로 다음 정보를 출력합니다.

클래스 이름: cup
신뢰도: 0.87
박스 좌표: x1, y1, x2, y2

바운딩 박스 중심 좌표는 다음과 같이 구합니다.

center_x = int((x1 + x2) / 2)
center_y = int((y1 + y2) / 2)
2.3 3차원 좌표 계산

YOLO가 구한 좌표는 카메라 영상의 픽셀 좌표입니다.

예:

u = 320 픽셀
v = 240 픽셀

그러나 로봇팔은 픽셀 좌표로 움직일 수 없습니다.

로봇팔에는 다음과 같은 공간 좌표가 필요합니다.

X = 0.32m
Y = -0.08m
Z = 0.15m

따라서 픽셀 좌표와 깊이값을 이용해 3차원 좌표로 변환해야 합니다.

2.4 MoveIt 2

MoveIt 2는 로봇팔의 이동 경로를 계산합니다.

MoveIt 2가 담당하는 기능은 다음과 같습니다.

목표 자세 설정
충돌 검사
관절 각도 계산
이동 경로 생성
로봇팔 제어 명령 전달

예를 들어 물체가 다음 위치에 있다고 가정합니다.

X = 0.30m
Y = 0.10m
Z = 0.12m

MoveIt 2에 이 좌표를 목표로 전달하면 로봇팔이 접근할 수 있는 관절 각도와 경로를 계산합니다.

2.5 그리퍼

그리퍼는 물체를 실제로 잡고 놓습니다.

기본 명령은 두 가지입니다.

OPEN: 그리퍼 열기
CLOSE: 그리퍼 닫기

실제 시스템에서는 물체 종류에 따라 그리퍼 닫힘 정도를 다르게 설정할 수 있습니다.

컵: 40%
작은 부품: 70%
스펀지: 30%
3. 필요한 하드웨어
기본 구성
Ubuntu 24.04 PC
ROS 2 Jazzy
MoveIt 2
로봇팔
전동 그리퍼
RGB-D 카메라
카메라 고정 브래킷
작업 테이블
테스트용 물체

현재 사용 중인 OMX-F 로봇팔을 기준으로 구성하면 다음과 같습니다.

OMX-F 로봇팔
OpenRB-150
Dynamixel 모터
USB Depth Camera
Ubuntu PC
ROS 2 Jazzy
MoveIt 2
YOLO
4. 카메라 설치 방식

AI Pick & Place에서는 카메라 설치 방식이 매우 중요합니다.

4.1 Eye-to-Hand 방식

카메라를 작업대 위나 외부 프레임에 고정합니다.

카메라
   ↓
작업대 ─ 물체
   ↑
로봇팔

장점:

카메라 영상이 안정적
물체 전체를 보기 쉬움
초기 구현이 쉬움
케이블 관리가 편함

단점:

카메라 좌표계와 로봇 좌표계 보정 필요
로봇팔이 물체를 가릴 수 있음

초보자에게 가장 권장되는 방식입니다.

4.2 Eye-in-Hand 방식

카메라를 로봇팔 끝부분에 설치합니다.

로봇팔 끝
   ├─ 그리퍼
   └─ 카메라

장점:

물체에 가까이 접근 가능
정밀한 위치 보정 가능
다양한 작업 영역을 관찰 가능

단점:

로봇 이동에 따라 카메라 좌표가 계속 변함
캘리브레이션이 복잡함
케이블 처리와 하중 문제가 생김

처음에는 Eye-to-Hand 방식으로 시작하는 것이 좋습니다.

5. 픽셀 좌표를 3차원 좌표로 변환하기

Depth Camera에서 물체 중심의 픽셀 좌표와 깊이값을 얻었다고 가정합니다.

픽셀 좌표: u, v
깊이값: Z

카메라 내부 파라미터는 다음과 같습니다.

fx: X축 초점거리
fy: Y축 초점거리
cx: 영상 중심 X
cy: 영상 중심 Y

3차원 좌표는 다음 식으로 계산합니다.

X = (u - cx) × Z / fx
Y = (v - cy) × Z / fy
Z = 깊이값

파이썬 함수로 작성하면 다음과 같습니다.

def pixel_to_camera_xyz(u, v, depth, fx, fy, cx, cy):
    """
    영상 픽셀 좌표와 깊이값을
    카메라 기준 3차원 좌표로 변환합니다.
    """

    x = (u - cx) * depth / fx
    y = (v - cy) * depth / fy
    z = depth

    return x, y, z

예를 들어 다음 값이 들어왔다고 가정합니다.

u = 350
v = 220
depth = 0.55m

fx = 615
fy = 615
cx = 320
cy = 240

계산 결과는 대략 다음과 같습니다.

X ≈ 0.0268m
Y ≈ -0.0179m
Z = 0.55m

이 좌표는 아직 카메라 좌표계 기준입니다.

6. 카메라 좌표와 로봇 좌표

가장 중요한 개념 중 하나입니다.

카메라가 측정한 물체 위치는 다음 좌표계를 기준으로 합니다.

camera_color_optical_frame

하지만 로봇팔은 일반적으로 다음 좌표계를 기준으로 움직입니다.

base_link

따라서 좌표 변환이 필요합니다.

카메라 좌표
camera_color_optical_frame
        ↓ TF 변환
로봇 좌표
base_link

ROS 2에서는 TF2가 이 좌표 변환을 담당합니다.

7. TF2를 이용한 좌표 변환

물체 좌표를 PointStamped 메시지로 생성합니다.

from geometry_msgs.msg import PointStamped

point_camera = PointStamped()
point_camera.header.frame_id = "camera_color_optical_frame"
point_camera.header.stamp = node.get_clock().now().to_msg()

point_camera.point.x = object_x
point_camera.point.y = object_y
point_camera.point.z = object_z

그다음 base_link 좌표계로 변환합니다.

point_robot = tf_buffer.transform(
    point_camera,
    "base_link"
)

변환 결과는 다음처럼 사용할 수 있습니다.

robot_x = point_robot.point.x
robot_y = point_robot.point.y
robot_z = point_robot.point.z

이제 MoveIt에 전달할 수 있는 로봇 기준 물체 좌표가 만들어집니다.

8. 카메라와 로봇 사이의 캘리브레이션

카메라 좌표를 로봇 좌표로 정확히 변환하려면 카메라의 위치와 방향을 알아야 합니다.

이를 Hand-Eye Calibration 또는 외부 파라미터 보정이라고 합니다.

간단한 수동 보정

초기 실습에서는 카메라의 위치를 직접 측정할 수 있습니다.

예:

로봇 base_link 기준

카메라 X 위치: 0.35m
카메라 Y 위치: 0.00m
카메라 Z 위치: 0.65m

그리고 카메라가 아래쪽을 바라보도록 회전되어 있다면 정적 TF를 등록합니다.

예:

ros2 run tf2_ros static_transform_publisher \
  --x 0.35 \
  --y 0.0 \
  --z 0.65 \
  --roll 0.0 \
  --pitch 1.5708 \
  --yaw 0.0 \
  --frame-id base_link \
  --child-frame-id camera_link

실제 회전값은 카메라 설치 방향에 맞게 조정해야 합니다.

9. 전체 Pick & Place 동작 순서

안전한 Pick & Place는 물체 위치로 바로 이동하지 않습니다.

일반적으로 다음 위치를 사용합니다.

대기 위치
  ↓
물체 위 접근 위치
  ↓
잡기 위치
  ↓
물체 위 접근 위치
  ↓
목표 위 접근 위치
  ↓
내려놓기 위치
  ↓
목표 위 접근 위치
  ↓
대기 위치
9.1 Home 위치

로봇팔의 기본 대기 자세입니다.

home

카메라 시야를 가리지 않고 다음 동작을 준비할 수 있는 위치로 정합니다.

9.2 Pre-grasp 위치

물체 바로 위쪽의 접근 위치입니다.

물체 위치가 다음과 같다면:

물체 위치
X = 0.30
Y = 0.05
Z = 0.08

접근 위치는 다음처럼 설정할 수 있습니다.

X = 0.30
Y = 0.05
Z = 0.18

즉, 물체보다 10cm 위쪽입니다.

pre_grasp_z = object_z + 0.10
9.3 Grasp 위치

그리퍼가 물체를 잡는 실제 위치입니다.

grasp_z = object_z + grasp_offset

그리퍼의 손가락 길이와 TCP 위치에 따라 오프셋이 달라집니다.

예:

grasp_offset = 0.03
9.4 Lift 위치

물체를 잡은 뒤 바로 옆으로 움직이면 바닥이나 주변 물체에 충돌할 수 있습니다.

먼저 위로 들어 올립니다.

lift_z = object_z + 0.15
9.5 Place 위치

물체를 내려놓을 목표 좌표입니다.

예:

X = 0.20m
Y = -0.20m
Z = 0.10m

목표 위치도 접근 위치와 실제 내려놓기 위치로 나눕니다.

pre-place
place
10. 상태 머신 구조

Pick & Place 프로그램은 한 번에 모든 코드를 실행하기보다 상태 머신으로 구성하는 것이 좋습니다.

IDLE
DETECT
CALCULATE_POSITION
MOVE_PRE_GRASP
MOVE_GRASP
CLOSE_GRIPPER
LIFT
MOVE_PRE_PLACE
MOVE_PLACE
OPEN_GRIPPER
RETURN_HOME

파이썬 구조 예시는 다음과 같습니다.

class PickPlaceState:
    IDLE = 0
    DETECT = 1
    MOVE_PRE_GRASP = 2
    MOVE_GRASP = 3
    CLOSE_GRIPPER = 4
    LIFT = 5
    MOVE_PRE_PLACE = 6
    MOVE_PLACE = 7
    OPEN_GRIPPER = 8
    RETURN_HOME = 9

상태 머신을 사용하면 문제가 발생한 단계를 쉽게 확인할 수 있습니다.

11. ROS 2 노드 구성

초기 시스템은 다음과 같이 구성할 수 있습니다.

/depth_camera_node
        │
        ├─ /camera/color/image_raw
        ├─ /camera/depth/image_raw
        └─ /camera/camera_info

/yolo_detector
        │
        └─ /detected_object

/object_position_node
        │
        └─ /object_point

/pick_place_node
        │
        ├─ MoveIt 2
        └─ Gripper Controller
12. 권장 ROS 2 토픽
/camera/color/image_raw
/camera/depth/image_raw
/camera/color/camera_info
/detections
/target_pixel
/target_point_camera
/target_point_base
/joint_states
/tf
/tf_static

사용자 정의 메시지를 만들지 않고 초기에는 다음 메시지를 사용할 수 있습니다.

geometry_msgs/msg/PointStamped
geometry_msgs/msg/PoseStamped
sensor_msgs/msg/Image
vision_msgs/msg/Detection2DArray
13. 물체 위치 ROS 2 메시지 발행 예제
import rclpy

from rclpy.node import Node
from geometry_msgs.msg import PointStamped


class ObjectPointPublisher(Node):

    def __init__(self):
        super().__init__("object_point_publisher")

        self.publisher = self.create_publisher(
            PointStamped,
            "/target_point_camera",
            10
        )

        self.timer = self.create_timer(
            0.5,
            self.publish_point
        )

    def publish_point(self):
        msg = PointStamped()

        msg.header.frame_id = "camera_color_optical_frame"
        msg.header.stamp = self.get_clock().now().to_msg()

        # 예제 물체 위치
        msg.point.x = 0.02
        msg.point.y = -0.03
        msg.point.z = 0.50

        self.publisher.publish(msg)

        self.get_logger().info(
            f"물체 좌표 발행: "
            f"x={msg.point.x:.3f}, "
            f"y={msg.point.y:.3f}, "
            f"z={msg.point.z:.3f}"
        )


def main(args=None):
    rclpy.init(args=args)

    node = ObjectPointPublisher()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass

    node.destroy_node()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
14. 카메라 중심 깊이값 읽기

깊이 영상은 일반 영상과 달리 각 픽셀에 거리값이 저장되어 있습니다.

단순 예제는 다음과 같습니다.

depth_value = depth_image[center_y, center_x]

하지만 중심 한 점의 깊이값만 읽으면 노이즈가 클 수 있습니다.

따라서 중심 주변 영역의 중앙값을 사용하는 것이 좋습니다.

import numpy as np


def get_stable_depth(depth_image, center_x, center_y, window=5):
    """
    물체 중심 주변의 깊이값 중앙값을 계산합니다.
    """

    half = window // 2

    x1 = max(0, center_x - half)
    x2 = min(depth_image.shape[1], center_x + half + 1)

    y1 = max(0, center_y - half)
    y2 = min(depth_image.shape[0], center_y + half + 1)

    region = depth_image[y1:y2, x1:x2]

    # 깊이값 0은 측정 실패일 가능성이 높으므로 제거합니다.
    valid_values = region[region > 0]

    if len(valid_values) == 0:
        return None

    return float(np.median(valid_values))
15. YOLO와 Depth Camera 결합 구조
# 1. YOLO로 물체 탐지
results = model(color_frame)

# 2. 바운딩 박스 추출
x1, y1, x2, y2 = box

# 3. 중심 좌표 계산
center_x = int((x1 + x2) / 2)
center_y = int((y1 + y2) / 2)

# 4. 깊이값 계산
depth = get_stable_depth(
    depth_image,
    center_x,
    center_y
)

# 5. 3차원 좌표 계산
object_x, object_y, object_z = pixel_to_camera_xyz(
    center_x,
    center_y,
    depth,
    fx,
    fy,
    cx,
    cy
)
16. 여러 물체가 탐지될 때 선택 방법

YOLO가 여러 개의 컵을 발견할 수 있습니다.

어떤 물체를 잡을지 선택하는 규칙이 필요합니다.

가장 신뢰도가 높은 물체
target = max(detections, key=lambda item: item.confidence)
화면 중심에 가장 가까운 물체
distance = (
    (center_x - image_center_x) ** 2
    + (center_y - image_center_y) ** 2
)
로봇팔에서 가장 가까운 물체

각 물체의 3차원 거리를 계산합니다.

distance = (
    x ** 2
    + y ** 2
    + z ** 2
) ** 0.5
특정 클래스만 선택
TARGET_CLASS = "cup"

if detected_class == TARGET_CLASS:
    target = detection

초기에는 컵 하나만 작업대에 놓고 테스트하는 것이 가장 안전합니다.

17. MoveIt 2 목표 Pose 생성

MoveIt은 위치뿐 아니라 그리퍼 방향도 필요합니다.

from geometry_msgs.msg import PoseStamped

target_pose = PoseStamped()

target_pose.header.frame_id = "base_link"

target_pose.pose.position.x = target_x
target_pose.pose.position.y = target_y
target_pose.pose.position.z = target_z

그리퍼가 아래쪽을 향하도록 자세를 설정해야 합니다.

쿼터니언 예시는 다음과 같습니다.

target_pose.pose.orientation.x = 0.0
target_pose.pose.orientation.y = 1.0
target_pose.pose.orientation.z = 0.0
target_pose.pose.orientation.w = 0.0

이 값은 로봇 URDF의 축 방향에 따라 달라질 수 있습니다.

RViz에서 실제 로봇 자세를 확인하며 조정해야 합니다.

18. 그리퍼 기준점 TCP

로봇팔의 마지막 링크 위치와 실제 손가락 끝 위치는 다릅니다.

예:

end_effector_link
        ↓ 8cm
그리퍼 손가락 끝

이 차이를 고려하지 않으면 로봇팔이 물체 위에서 멈추거나 바닥을 누를 수 있습니다.

따라서 TCP, 즉 Tool Center Point를 정확히 설정해야 합니다.

TCP = 실제 물체를 잡는 기준점

URDF나 Xacro에서 고정 조인트로 TCP 링크를 추가할 수 있습니다.

<link name="gripper_tcp"/>

<joint name="gripper_tcp_joint" type="fixed">
  <parent link="end_effector_link"/>
  <child link="gripper_tcp"/>

  <origin
    xyz="0 0 0.08"
    rpy="0 0 0"/>
</joint>

MoveIt의 end effector 기준을 gripper_tcp로 설정하면 좌표 계산이 쉬워집니다.

19. 충돌 방지

Pick & Place에서는 작업대를 충돌 객체로 등록해야 합니다.

그렇지 않으면 MoveIt이 로봇팔이 작업대를 통과하는 경로를 만들 수도 있습니다.

충돌 객체 예:

작업 테이블
물체 보관함
카메라 지지대
로봇 베이스
주변 벽

테이블은 MoveIt Planning Scene에 박스로 등록할 수 있습니다.

개념 예:

table_size_x = 1.0
table_size_y = 0.8
table_size_z = 0.05

테이블 상단 높이가 정확해야 합니다.

20. 기본 Pick & Place 의사 코드
def pick_and_place():
    # 1. 홈 위치 이동
    move_to_named_target("home")

    # 2. 그리퍼 열기
    open_gripper()

    # 3. 물체 탐지
    object_pose = detect_object()

    if object_pose is None:
        print("물체를 찾지 못했습니다.")
        return

    # 4. 접근 위치 생성
    pre_grasp_pose = object_pose.copy()
    pre_grasp_pose.z += 0.10

    # 5. 물체 위로 이동
    move_to_pose(pre_grasp_pose)

    # 6. 잡기 위치로 하강
    move_to_pose(object_pose)

    # 7. 그리퍼 닫기
    close_gripper()

    # 8. 물체 들어 올리기
    move_to_pose(pre_grasp_pose)

    # 9. 목표 위로 이동
    move_to_pose(pre_place_pose)

    # 10. 내려놓기 위치로 이동
    move_to_pose(place_pose)

    # 11. 그리퍼 열기
    open_gripper()

    # 12. 위로 이동
    move_to_pose(pre_place_pose)

    # 13. 홈 위치 복귀
    move_to_named_target("home")
21. 실습 1 — 고정 좌표 Pick & Place

처음부터 YOLO와 카메라를 연결하지 않는 것이 좋습니다.

먼저 좌표를 직접 입력해 로봇팔의 Pick & Place 동작을 확인합니다.

예:

Pick 위치
X = 0.25
Y = 0.10
Z = 0.08

Place 위치
X = 0.25
Y = -0.10
Z = 0.08

실습 순서:

로봇팔을 Home 위치로 이동합니다.
그리퍼를 엽니다.
Pick 위치보다 10cm 위로 이동합니다.
천천히 아래로 이동합니다.
그리퍼를 닫습니다.
10cm 위로 들어 올립니다.
Place 위치 위로 이동합니다.
천천히 내려갑니다.
그리퍼를 엽니다.
Home 위치로 돌아옵니다.

이 단계가 성공해야 카메라를 연결합니다.

22. 실습 2 — 마우스로 선택한 물체 좌표 사용

YOLO 연결 전에 카메라 화면에서 마우스로 물체를 클릭하는 방식도 좋습니다.

카메라 영상 표시
       ↓
사용자가 물체 중심 클릭
       ↓
클릭한 픽셀의 깊이값 계산
       ↓
3차원 좌표 계산
       ↓
로봇팔 이동

이 방법을 사용하면 다음을 먼저 검증할 수 있습니다.

깊이값이 정확한가
카메라 좌표 변환이 정확한가
TF가 올바른가
로봇팔이 목표 위치로 이동하는가
23. 실습 3 — YOLO 자동 Pick & Place

전체 자동화 단계입니다.

YOLO로 cup 탐지
     ↓
cup 중심 픽셀 계산
     ↓
Depth 값 읽기
     ↓
camera 좌표 계산
     ↓
base_link 좌표 변환
     ↓
MoveIt 목표 생성
     ↓
Pick
     ↓
Place

안전상 처음에는 다음 조건으로 시작합니다.

물체 1개
낮은 속도
넓은 작업 공간
장애물 없음
부드러운 물체
비상정지 준비
24. 정확도를 높이는 방법
24.1 중심 한 점 대신 영역 사용

컵 중앙이 비어 있거나 반사되면 깊이 측정이 실패할 수 있습니다.

따라서 바운딩 박스 내부의 여러 깊이값을 사용합니다.

중앙값
평균값
가장 가까운 유효값
분포에서 이상치 제거
24.2 여러 프레임 평균

한 프레임만 사용하지 않고 5~10프레임의 좌표를 평균냅니다.

stable_x = sum(x_history) / len(x_history)
stable_y = sum(y_history) / len(y_history)
stable_z = sum(z_history) / len(z_history)
24.3 좌표 이동 제한

물체 좌표가 갑자기 크게 변하면 잘못된 탐지일 수 있습니다.

MAX_POSITION_CHANGE = 0.03

이전 좌표와 3cm 이상 차이가 나면 잠시 무시하는 방식입니다.

24.4 작업 영역 제한

로봇이 이동할 수 있는 안전 영역을 설정합니다.

X_MIN = 0.15
X_MAX = 0.40

Y_MIN = -0.25
Y_MAX = 0.25

Z_MIN = 0.03
Z_MAX = 0.35

검사 함수:

def is_safe_position(x, y, z):
    return (
        X_MIN <= x <= X_MAX
        and Y_MIN <= y <= Y_MAX
        and Z_MIN <= z <= Z_MAX
    )
25. 실패 처리

실제 Pick & Place에서는 실패를 반드시 고려해야 합니다.

물체가 사라진 경우
탐지 취소
로봇 정지
Home 위치 복귀
깊이값을 읽지 못한 경우
다른 픽셀의 깊이값 사용
여러 프레임 재측정
그래도 실패하면 작업 취소
MoveIt 경로 생성 실패
목표 위치를 조금 위로 변경
그리퍼 방향 변경
다른 경로 계획 시도
그리퍼가 물체를 잡지 못한 경우

가능한 검출 방법:

그리퍼 전류 증가 여부
모터 위치 변화
손가락 사이 거리
카메라로 물체 재확인
무게 센서
힘·토크 센서
26. 안전 규칙

AI Pick & Place는 실제 로봇이 자동으로 움직이므로 안전이 매우 중요합니다.

반드시 다음 사항을 지켜야 합니다.

최초 테스트는 낮은 속도로 진행합니다.
로봇 근처에 손을 넣지 않습니다.
비상정지 방법을 미리 준비합니다.
최대 관절 속도와 가속도를 낮춥니다.
작업 영역 제한을 설정합니다.
카메라 인식 결과만 믿고 즉시 이동하지 않습니다.
여러 프레임에서 안정된 좌표인지 확인합니다.
바닥과 작업대를 충돌 객체로 등록합니다.
처음에는 가벼운 스펀지나 빈 종이컵을 사용합니다.
전원 차단 스위치를 가까운 곳에 둡니다.
27. 권장 개발 순서

전체 시스템을 한 번에 만들지 말고 다음 순서로 진행합니다.

1단계: 로봇팔만 제어
Home 이동
지정 좌표 이동
그리퍼 열기와 닫기
2단계: 고정 좌표 Pick & Place
미리 입력한 좌표에서 물체 이동
3단계: Depth Camera 좌표 확인
마우스로 픽셀 선택
깊이값 표시
3차원 좌표 출력
4단계: TF 좌표 변환
카메라 좌표
→ base_link 좌표
5단계: 카메라 좌표로 로봇 이동

아직 물체를 잡지 않고 물체 위로만 이동합니다.

6단계: YOLO 연결
YOLO 탐지
→ 물체 좌표
→ 로봇 접근
7단계: 자동 Pick

물체를 집고 위로 들어 올리는 것까지 구현합니다.

8단계: 자동 Place

지정된 장소에 물체를 내려놓습니다.

28. 추천 프로젝트 폴더 구조
ai_pick_place_ws/
└── src/
    └── ai_pick_place/
        ├── ai_pick_place/
        │   ├── __init__.py
        │   ├── yolo_detector_node.py
        │   ├── depth_position_node.py
        │   ├── tf_transform_node.py
        │   ├── pick_place_node.py
        │   └── gripper_controller.py
        │
        ├── launch/
        │   └── ai_pick_place.launch.py
        │
        ├── config/
        │   ├── camera.yaml
        │   ├── pick_place.yaml
        │   └── safety_limits.yaml
        │
        ├── models/
        │   └── best.pt
        │
        ├── package.xml
        ├── setup.py
        └── README.md
29. 설정 파일 예제

config/pick_place.yaml

pick_place_node:
  ros__parameters:

    target_class: "cup"

    confidence_threshold: 0.60

    approach_height: 0.10
    lift_height: 0.15

    grasp_offset_z: 0.03

    place_x: 0.25
    place_y: -0.15
    place_z: 0.08

    max_velocity_scale: 0.10
    max_acceleration_scale: 0.10

    workspace_x_min: 0.15
    workspace_x_max: 0.40

    workspace_y_min: -0.25
    workspace_y_max: 0.25

    workspace_z_min: 0.03
    workspace_z_max: 0.35
30. 실행 전 확인 명령

ROS 환경을 적용합니다.

source /opt/ros/jazzy/setup.bash
source ~/omx_moveit_ws/install/setup.bash

노드를 확인합니다.

ros2 node list

관절 상태를 확인합니다.

ros2 topic echo /joint_states --once

TF 구조를 확인합니다.

ros2 run tf2_tools view_frames

카메라 토픽을 확인합니다.

ros2 topic list | grep camera

MoveIt 관련 노드를 확인합니다.

ros2 node list | grep move

컨트롤러를 확인합니다.

ros2 control list_controllers

최소한 다음 컨트롤러가 활성화되어야 합니다.

joint_state_broadcaster
arm_controller
gripper_controller
31. 자주 발생하는 문제
로봇이 물체 반대 방향으로 이동함

원인:

카메라 축 방향 오류
TF 회전값 오류
X, Y 축 부호 오류

해결:

ros2 run tf2_ros tf2_echo base_link camera_color_optical_frame

좌표축을 RViz에서 확인합니다.

물체보다 위나 아래에서 멈춤

원인:

깊이 단위 오류
TCP 오프셋 오류
작업대 높이 오류
카메라 캘리브레이션 오류

특히 깊이값이 밀리미터인지 미터인지 확인해야 합니다.

depth_m = depth_mm / 1000.0
경로 계획이 실패함

원인:

목표 위치가 로봇 작업 반경 밖
그리퍼 방향이 불가능함
충돌 객체와 겹침
관절 제한 초과

해결:

목표 Z를 높입니다.
손목 방향을 변경합니다.
접근 위치를 먼저 사용합니다.
작업 영역을 축소합니다.
로봇이 물체 위치를 정확히 못 맞춤

원인:

카메라와 로봇 사이 TF 오차
카메라 왜곡
깊이값 노이즈
로봇 기구 오차
TCP 설정 오류

처음에는 좌표별 오차를 측정하여 보정값을 추가할 수 있습니다.

robot_x = measured_x + offset_x
robot_y = measured_y + offset_y
robot_z = measured_z + offset_z

예:

offset_x = 0.012
offset_y = -0.008
offset_z = 0.015
32. 단계별 성공 기준
초급 성공 기준
고정된 좌표에서 컵을 집는다.
지정된 좌표에 컵을 내려놓는다.
중급 성공 기준
카메라에서 컵을 탐지한다.
컵의 3차원 위치를 계산한다.
컵 위 접근 위치로 이동한다.
고급 성공 기준
여러 물체 중 특정 물체를 선택한다.
자동으로 집어서 분류 장소에 놓는다.
실패 시 재시도한다.
충돌을 회피한다.
33. 응용 프로젝트

16단계를 완성하면 다음 프로젝트로 확장할 수 있습니다.

색상별 부품 분류
빨간 부품 → 1번 상자
파란 부품 → 2번 상자
노란 부품 → 3번 상자
불량품 검사와 분류
정상 제품 → 정상 라인
불량 제품 → 불량 상자
컨베이어 벨트 Pick & Place

움직이는 물체의 위치와 속도를 추정해 집습니다.

공구 정리 로봇

드라이버, 렌치, 펜치 등을 인식해서 지정된 자리에 놓습니다.

재활용품 분류
캔
페트병
종이
플라스틱
AI 조립 로봇

볼트, 너트, 부품을 인식해 조립 작업을 수행합니다.

34. 16단계 최종 실습 목표

이번 단계의 최종 실습은 다음과 같습니다.

Depth Camera로 작업대 위의 컵을 탐지하고, YOLO로 컵 중심을 찾은 뒤, 깊이값으로 3차원 좌표를 계산합니다. 이 좌표를 base_link 기준으로 변환하고 MoveIt 2로 OMX-F 로봇팔을 이동시켜 컵을 잡아 지정된 위치에 내려놓습니다.

최종 실행 흐름:

ROS 2 실행
   ↓
OMX-F 하드웨어 연결
   ↓
MoveIt 2 실행
   ↓
Depth Camera 실행
   ↓
YOLO 실행
   ↓
컵 탐지
   ↓
3차원 좌표 계산
   ↓
TF 변환
   ↓
안전 영역 검사
   ↓
Pre-grasp 이동
   ↓
Grasp 이동
   ↓
그리퍼 닫기
   ↓
Lift
   ↓
Place 위치 이동
   ↓
그리퍼 열기
   ↓
Home 복귀
35. 16단계 핵심 정리

16단계에서 반드시 이해해야 할 핵심은 다음과 같습니다.

YOLO는 물체의 2차원 위치를 찾는다.

Depth Camera는 물체까지의 거리를 측정한다.

카메라 내부 파라미터를 이용해
픽셀 좌표를 3차원 좌표로 변환한다.

TF2를 이용해 카메라 좌표를
로봇의 base_link 좌표로 변환한다.

MoveIt 2는 목표 위치까지의
관절 각도와 안전한 경로를 계산한다.

그리퍼는 물체를 잡고 놓는다.

Pick & Place는 접근, 잡기, 들어 올리기,
이동, 내려놓기의 순서로 실행한다.

16단계는 지금까지 따로 배웠던 기술들이 실제 로봇 작업 하나로 연결되는 핵심 통합 단계입니다.
