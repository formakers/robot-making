17단계: LeRobot으로 로봇 학습시키기

LeRobot은 Hugging Face에서 개발한 실제 로봇 학습용 오픈소스 프레임워크입니다. 로봇 조작, 카메라 영상과 관절 데이터 수집, 데이터셋 관리, AI 정책 학습, 실제 로봇 자율 실행까지 하나의 흐름으로 구성합니다. 특히 사람이 로봇을 조작한 동작을 AI가 따라 배우는 모방학습(기계가 사람의 시범 동작을 학습하는 방식)에 적합합니다.

현재 포메이커스님의 구성에서는 다음 장비를 활용할 수 있습니다.

리더 로봇팔: OpenRB-150, 모터 ID 11~16
팔로워 로봇팔: OpenRB-150, 모터 ID 1~6
USB 카메라: Logitech C920 등
운영체제: Ubuntu 24.04
Python 환경: lerobot
LeRobot 버전: 현재 설치 환경 기준 0.6.x 계열
목표 작업: 컵 집기, 컵 이동, 지정 위치에 놓기
1. LeRobot의 핵심 개념

기존 로봇 제어에서는 사람이 좌표, 속도, 경로를 직접 프로그래밍합니다.

목표 좌표 설정
→ 역기구학 계산
→ 관절 각도 계산
→ 모터 이동

LeRobot에서는 사람이 여러 번 시범을 보여주고, AI가 시범 데이터에서 동작 규칙을 학습합니다.

사람이 로봇 조작
→ 카메라 영상과 관절값 기록
→ 데이터셋 생성
→ AI 모델 학습
→ 카메라 영상을 보고 로봇이 스스로 움직임

공식 LeRobot 실물 로봇 학습 과정도 크게 다음 세 단계로 설명합니다.

로봇을 원격 조작하며 데이터를 기록한다.
기록된 데이터로 정책 모델을 학습한다.
학습된 정책을 실제 로봇에서 실행하고 평가한다.

여기서 정책(Policy)은 현재 카메라 영상과 관절 상태를 입력받아, 로봇이 다음에 어떤 동작을 해야 하는지 출력하는 AI 모델입니다.

2. LeRobot 전체 구조
입력 데이터

LeRobot이 받는 대표적인 입력은 다음과 같습니다.

카메라 영상
현재 관절 위치
그리퍼 상태
로봇의 현재 센서값
작업 설명

예를 들면 다음과 같습니다.

observation = {
    "observation.images.camera": camera_image,
    "observation.state": [
        joint1,
        joint2,
        joint3,
        joint4,
        joint5,
        gripper
    ]
}
출력 데이터

정책 모델은 다음에 실행할 관절 목표값을 출력합니다.

action = [
    target_joint1,
    target_joint2,
    target_joint3,
    target_joint4,
    target_joint5,
    target_gripper
]
전체 동작
카메라 촬영
    ↓
현재 관절값 읽기
    ↓
AI 정책 모델에 입력
    ↓
다음 관절 목표값 출력
    ↓
팔로워 로봇에 명령
    ↓
새로운 카메라 영상 입력
    ↓
반복
3. LeRobot에서 사용하는 주요 용어
Robot

실제로 움직이는 로봇입니다.

포메이커스님의 경우:

OMX Follower
/dev/ttyACM1
모터 ID 1~6

일반적인 역할은 다음과 같습니다.

robot.connect()
robot.get_observation()
robot.send_action(action)
robot.disconnect()
Teleoperator

사람이 움직이는 조작 장치입니다.

포메이커스님의 경우:

OMX Leader
/dev/ttyACM0
모터 ID 11~16

리더 로봇팔을 사람이 움직이면 팔로워 로봇팔이 같은 동작을 따라갑니다.

Observation

로봇이 현재 보고 있는 정보입니다.

카메라 영상
현재 관절 각도
그리퍼 상태
센서 정보
Action

로봇이 실행해야 할 명령입니다.

각 관절의 목표 위치
그리퍼 열기 또는 닫기
Episode

하나의 작업을 시작부터 끝까지 수행한 기록입니다.

예:

에피소드 시작
→ 컵으로 접근
→ 그리퍼 열기
→ 컵 잡기
→ 컵 들어 올리기
→ 목표 위치로 이동
→ 컵 놓기
→ 에피소드 종료

컵 집기 작업을 50번 기록한다면 데이터셋에는 약 50개의 에피소드가 만들어집니다.

Dataset

여러 에피소드를 모아 놓은 학습 데이터입니다.

LeRobotDataset 3.0은 에피소드, 프레임, 관절 상태, 행동값, 영상 등을 구조적으로 저장하며 Hugging Face Hub에 업로드하거나 학습에 불러올 수 있도록 설계되어 있습니다.

Policy

데이터를 학습해 로봇의 행동을 결정하는 신경망입니다.

현재 영상 + 현재 관절값
→ 정책 모델
→ 다음 관절 명령
4. 17단계 학습 목표

17단계를 마치면 다음 작업을 할 수 있어야 합니다.

1. LeRobot 환경을 실행한다.
2. 리더와 팔로워 로봇을 연결한다.
3. 리더로 팔로워를 원격 조작한다.
4. 카메라 영상을 확인한다.
5. 작업 데이터를 에피소드 단위로 기록한다.
6. 데이터를 Hugging Face Hub에 업로드한다.
7. ACT 등의 정책 모델을 학습한다.
8. 학습된 모델을 실제 로봇에서 실행한다.
9. 실패 데이터를 분석하고 추가 학습한다.
5. 실습 준비
터미널 열기
conda activate lerobot

설치 확인:

python --version
pip show lerobot

명령어 확인:

lerobot-teleoperate --help
lerobot-record --help
lerobot-train --help
lerobot-rollout --help

최근 LeRobot에서는 실제 로봇 실행에 lerobot-rollout 명령을 사용하도록 공식 문서에서 안내합니다. 버전에 따라 예전의 lerobot-eval 또는 Python 모듈 실행 방식과 옵션 이름이 달라질 수 있으므로, 설치된 버전의 --help 출력을 우선해야 합니다.

6. USB 포트 확인
ls -l /dev/ttyACM*

예상 결과:

/dev/ttyACM0
/dev/ttyACM1

안정적으로 구분하려면 다음 명령을 사용합니다.

ls -l /dev/serial/by-id/

예:

usb-ROBOTIS_OpenRB-150_리더 → ../../ttyACM0
usb-ROBOTIS_OpenRB-150_팔로워 → ../../ttyACM1

USB 번호는 컴퓨터를 다시 시작하거나 연결 순서를 바꾸면 변경될 수 있습니다. 가능하면 /dev/serial/by-id/ 경로를 사용하는 것이 안전합니다.

권한 확인:

groups

출력에 dialout이 없으면:

sudo usermod -aG dialout $USER

그다음 로그아웃하고 다시 로그인합니다.

7. 포트를 점유한 프로그램 확인

다음 오류가 자주 발생할 수 있습니다.

Device or resource busy
could not open port /dev/ttyACM0

점유 프로세스 확인:

sudo lsof /dev/ttyACM0
sudo lsof /dev/ttyACM1

또는:

fuser /dev/ttyACM0
fuser /dev/ttyACM1

프로세스 종료:

sudo fuser -k /dev/ttyACM0
sudo fuser -k /dev/ttyACM1

Arduino IDE의 시리얼 모니터, 다른 LeRobot 프로그램, ROS 2 하드웨어 노드가 같은 포트를 사용하고 있으면 먼저 종료해야 합니다.

8. 카메라 확인

카메라 장치 목록:

v4l2-ctl --list-devices

장치 파일 확인:

ls -l /dev/video*

카메라 테스트:

ffplay /dev/video0

또는:

guvcview

Python으로 확인:

python - <<'PY'
import cv2

cap = cv2.VideoCapture("/dev/video0")

if not cap.isOpened():
    raise RuntimeError("카메라를 열 수 없습니다.")

ret, frame = cap.read()
print("프레임 읽기:", ret)

if ret:
    print("영상 크기:", frame.shape)

cap.release()
PY
9. 1차 실습: 리더-팔로워 원격 조작

원격 조작은 데이터 수집 전에 반드시 충분히 테스트해야 합니다.

개념적인 실행 구조는 다음과 같습니다.

lerobot-teleoperate \
  --robot.type=omx_follower \
  --robot.port=/dev/ttyACM1 \
  --robot.id=my_omx_follower \
  --teleop.type=omx_leader \
  --teleop.port=/dev/ttyACM0 \
  --teleop.id=my_omx_leader

단, OMX 지원 클래스의 정확한 이름과 옵션은 설치한 LeRobot 버전 또는 사용 중인 커스텀 OMX 통합 코드에 따라 달라질 수 있습니다.

현재 설치된 옵션은 반드시 다음 명령으로 확인합니다.

lerobot-teleoperate --help

정상 동작 흐름:

리더 연결
→ 팔로워 연결
→ 모터 캘리브레이션 불러오기
→ 리더 관절값 읽기
→ 팔로워에 목표 관절값 전송
→ 반복
테스트 순서

처음부터 빠르게 움직이지 마십시오.

1. 리더의 joint1만 조금 움직인다.
2. 팔로워 joint1이 같은 방향으로 움직이는지 확인한다.
3. joint2를 천천히 움직인다.
4. joint3, joint4, joint5를 차례대로 확인한다.
5. 마지막에 그리퍼를 확인한다.
즉시 중단해야 하는 현상
모터가 반대 방향으로 움직임
관절이 끝까지 빠르게 회전함
팔이 책상이나 카메라에 충돌함
그리퍼가 계속 닫히며 진동함
모터에서 과도한 소음이 발생함
통신 오류가 반복됨

이 경우 프로그램을 종료하고 토크를 해제한 뒤 캘리브레이션, 모터 ID, 관절 방향, 제한값을 확인해야 합니다.

10. 캘리브레이션

리더와 팔로워가 같은 자세를 재현하려면 각각의 모터 범위를 맞춰야 합니다.

캘리브레이션에는 보통 다음 정보가 저장됩니다.

모터 최소 위치
모터 최대 위치
중앙 위치
관절 방향
모터 ID
정규화 범위

예를 들어 리더의 joint1이 오른쪽으로 움직였는데 팔로워는 왼쪽으로 움직인다면 다음 중 하나가 잘못됐을 가능성이 있습니다.

관절 방향 부호
모터 장착 방향
캘리브레이션 최소·최대값
리더와 팔로워의 모터 순서

캘리브레이션 파일은 삭제하거나 무작정 복사하기보다 먼저 백업합니다.

cp calibration.json calibration_backup.json
11. 2차 실습: 학습 작업 선정

처음에는 단순하고 성공 여부가 분명한 작업을 선택해야 합니다.

추천 작업
빨간 컵을 오른쪽으로 옮기기
블록을 작은 상자에 넣기
스펀지를 집어서 표시된 원에 놓기
그리퍼로 물체를 밀어 목표선까지 이동하기
처음부터 피해야 할 작업
여러 종류 물체를 분류하기
컵에 물 따르기
뚜껑 열기
복잡한 조립
투명 물체 집기
반사광이 심한 금속 물체 집기
권장 첫 작업
작업 이름: 컵을 A 위치에서 B 위치로 옮기기

시작 상태:
- 컵은 왼쪽 표시선 안에 위치
- 로봇팔은 홈 위치
- 그리퍼는 열린 상태

종료 상태:
- 컵은 오른쪽 표시선 안에 위치
- 그리퍼는 열린 상태
- 로봇팔은 컵에서 떨어진 상태

작업 성공 조건을 명확히 해야 데이터 평가가 쉬워집니다.

12. 작업 환경 고정

모방학습에서는 환경 변화가 너무 크면 학습이 어려워집니다.

처음에는 다음을 고정합니다.

카메라 위치
카메라 각도
책상 높이
로봇 베이스 위치
조명
배경
컵 종류
목표 위치
작업 시작 자세

카메라 삼각대와 로봇 베이스 위치에 테이프 표시를 해두는 것이 좋습니다.

카메라 위치 표시
로봇 베이스 위치 표시
컵 시작 위치 표시
컵 목표 위치 표시
13. 3차 실습: 데이터 수집

공식 LeRobot 튜토리얼에서는 원격 조작 장치로 로봇을 움직이면서 움직임 궤적과 관측 데이터를 기록하도록 안내합니다.

개념적인 명령은 다음과 같습니다.

lerobot-record \
  --robot.type=omx_follower \
  --robot.port=/dev/ttyACM1 \
  --robot.id=my_omx_follower \
  --teleop.type=omx_leader \
  --teleop.port=/dev/ttyACM0 \
  --teleop.id=my_omx_leader \
  --robot.cameras="{front: {type: opencv, index_or_path: /dev/video0, width: 640, height: 480, fps: 30}}" \
  --dataset.repo_id=formakers/omx_cup_pick_place \
  --dataset.num_episodes=20 \
  --dataset.single_task="Pick up the cup and place it on the target."

정확한 옵션명은 버전별로 달라질 수 있으므로 먼저 확인합니다.

lerobot-record --help
에피소드 길이

첫 실습은 약 15~30초 정도가 적당합니다.

예:

0~3초: 로봇이 컵에 접근
3~6초: 그리퍼 위치 조정
6~9초: 컵 잡기
9~13초: 컵 들어 올리기
13~18초: 목표 위치로 이동
18~21초: 컵 내려놓기
21~24초: 그리퍼 열기
24~27초: 로봇 후퇴

너무 길면 불필요한 정지 구간이 많아지고, 너무 짧으면 움직임이 급해질 수 있습니다.

첫 데이터 수집 권장량

테스트 단계:

5개 에피소드

파이프라인 확인 단계:

10~20개 에피소드

첫 학습 단계:

30~50개 에피소드

성능 개선 단계:

50~100개 이상

정확한 필요량은 작업 난이도, 시범의 일관성, 카메라 구성, 모델 종류에 따라 달라집니다. 처음부터 많은 데이터를 기록하기보다 5개를 기록하고 영상과 관절 데이터를 확인한 후 본격적으로 수집하는 것이 안전합니다.

14. 좋은 데이터와 나쁜 데이터
좋은 데이터
동작 속도가 일정함
컵을 확실하게 잡음
불필요한 흔들림이 적음
시작과 종료 상태가 일정함
카메라에서 로봇과 물체가 잘 보임
목표 행동만 포함됨
나쁜 데이터
컵을 놓침
팔이 충돌함
그리퍼가 물체 옆을 잡음
사람의 손이 카메라를 계속 가림
작업 도중 오래 멈춤
에피소드 중간에 물체를 손으로 옮김
실패했는데 성공 데이터로 저장함

실패한 시범은 가능하면 다시 기록하는 것이 좋습니다.

15. 반복 동작은 괜찮은가?

같은 동작을 여러 번 반복하는 것은 모방학습에서 필요합니다.

다만 모든 에피소드가 기계적으로 완전히 같으면 모델이 환경 변화에 약해질 수 있습니다.

예를 들어 컵 시작 위치를 조금씩 변경합니다.

1회: 기준 위치
2회: 왼쪽으로 1cm
3회: 오른쪽으로 1cm
4회: 앞쪽으로 1cm
5회: 뒤쪽으로 1cm

초기에는 변화 범위를 작게 유지합니다.

초기 학습: ±1~2cm
성능 개선: ±3~5cm
고급 학습: 위치, 방향, 조명을 함께 변화
16. 데이터셋에 저장되는 내용

각 프레임에는 일반적으로 다음 정보가 기록됩니다.

프레임 번호
에피소드 번호
시간
카메라 영상
현재 관절 상태
사람이 명령한 목표 관절값
작업 설명
다음 프레임과의 관계

개념적으로는 다음과 같습니다.

sample = {
    "observation.images.front": image,
    "observation.state": current_joint_positions,
    "action": target_joint_positions,
    "task": "Pick up the cup and place it on the target.",
    "episode_index": 3,
    "frame_index": 127,
    "timestamp": 4.23
}

LeRobotDataset은 이런 로봇 시계열 데이터와 영상을 학습에 적합한 구조로 관리합니다.

17. Hugging Face 로그인

Hugging Face Hub에 데이터셋과 모델을 업로드하려면 로그인합니다.

huggingface-cli login

또는 토큰을 직접 지정할 수 있습니다.

huggingface-cli login \
  --token YOUR_HUGGINGFACE_TOKEN \
  --add-to-git-credential

공식 실물 로봇 시작 문서에서도 쓰기 권한이 있는 토큰으로 CLI 로그인하는 방법을 안내합니다.

로그인 확인:

huggingface-cli whoami

예상 결과:

formakers

토큰은 GitHub, 유튜브 영상, 블로그, 화면 녹화에 공개하지 마십시오.

18. 데이터셋 확인

데이터 기록이 끝난 후 바로 학습하지 말고 먼저 확인합니다.

확인 항목
영상이 정상적으로 저장됐는가?
영상과 관절 움직임이 시간상 일치하는가?
프레임이 지나치게 누락되지 않았는가?
에피소드 수가 맞는가?
실패한 에피소드가 포함됐는가?
카메라 영상이 너무 어둡거나 흐리지 않은가?
기록 속도 경고

포메이커스님의 기존 환경에서는 다음과 같은 메시지가 발생했습니다.

Record loop is running slower (6.6 Hz)
than the target FPS (30.0 Hz)

이 의미는 목표는 초당 30프레임인데 실제 데이터 처리 반복은 초당 약 6.6회만 실행됐다는 뜻입니다.

주요 원인:

카메라 입력 지연
영상 압축 부하
CPU 사용량 증가
OBS와 카메라 동시 사용
USB 대역폭 부족
디스크 저장 속도
Rerun 동시 실행
개선 방법

OBS와 다른 카메라 프로그램을 종료합니다.

pkill obs
pkill guvcview

카메라 해상도를 낮춥니다.

1280×720 → 640×480

처음에는 목표 FPS를 낮춰 테스트합니다.

30 FPS → 15 FPS

사용하지 않는 카메라를 제거합니다.

그리고 다음 명령으로 자원 사용량을 확인합니다.

htop

GPU 확인:

watch -n 1 nvidia-smi
19. 정책 모델 선택

LeRobot은 실제 로봇 학습을 위한 여러 정책 구현을 제공합니다.

초기에는 다음 순서가 좋습니다.

ACT

ACT는 하나의 동작만 예측하는 대신, 앞으로 실행할 여러 동작을 묶어 예측하는 방식입니다.

영상 + 현재 관절값
→ 앞으로 실행할 관절 명령 묶음

장점:

로봇팔 조작 학습에 많이 사용됨
동작이 비교적 부드러움
작은 데이터셋으로 실험하기 좋음
리더-팔로워 모방학습에 적합

포메이커스님의 첫 컵 이동 실습에는 ACT가 좋은 출발점입니다.

Diffusion Policy

확산 모델을 이용해 로봇 동작을 생성합니다.

장점:

복잡한 동작 분포를 표현할 수 있음
여러 가능한 행동을 학습하기 좋음

단점:

학습과 추론 계산량이 상대적으로 큼
GPU 메모리 요구량이 커질 수 있음
초기 설정이 ACT보다 복잡할 수 있음
VLA 계열

영상과 자연어 명령을 함께 처리합니다.

예:

"빨간 컵을 들어 상자에 넣어라"

매우 강력하지만 첫 실습에는 학습 비용과 시스템 복잡도가 큽니다.

20. 4차 실습: ACT 정책 학습

개념적인 학습 명령은 다음과 같습니다.

lerobot-train \
  --dataset.repo_id=formakers/omx_cup_pick_place \
  --policy.type=act \
  --output_dir=outputs/train/omx_cup_act \
  --job_name=omx_cup_act \
  --policy.device=cuda \
  --steps=10000

설치된 버전의 실제 옵션 확인:

lerobot-train --help

최근 배포판에서는 기본 설치에 학습 의존성이 전부 포함되지 않을 수 있으므로, 설치 버전에 따라 학습용 추가 패키지가 필요할 수 있습니다. 공식 릴리스 안내에서는 필요한 기능에 맞춰 lerobot[training] 같은 추가 설치 옵션을 사용하도록 설명합니다.

예:

pip install -e ".[training]"

소스 저장소가 아닌 일반 pip 설치 환경에서는:

pip install "lerobot[training]"
작은 테스트 학습

처음부터 긴 학습을 하지 말고 1,000스텝으로 파이프라인을 확인합니다.

lerobot-train \
  --dataset.repo_id=formakers/omx_cup_pick_place \
  --policy.type=act \
  --output_dir=outputs/train/omx_cup_act_test \
  --job_name=omx_cup_act_test \
  --policy.device=cuda \
  --steps=1000

확인할 내용:

데이터셋 로딩 성공
카메라 입력 크기 인식
관절 차원 인식
CUDA 사용 여부
손실값 출력
체크포인트 저장
오류 없이 학습 종료
21. 학습 중 확인해야 할 값
Loss

Loss는 AI의 예측과 실제 시범 동작 사이의 오차입니다.

Loss가 매우 큼
→ 예측이 실제 동작과 많이 다름

Loss가 감소함
→ 시범 동작을 점점 학습하고 있음

예:

step 100: loss 1.85
step 500: loss 0.92
step 1000: loss 0.48

하지만 Loss가 낮다고 실제 로봇 성공률이 반드시 높은 것은 아닙니다.

데이터를 외웠을 수 있음
카메라 위치 변화에 약할 수 있음
새로운 컵 위치에서 실패할 수 있음
실제 제어 지연이 반영되지 않을 수 있음

따라서 실제 로봇 평가가 반드시 필요합니다.

22. GPU 확인
nvidia-smi

Python에서 확인:

python - <<'PY'
import torch

print("PyTorch:", torch.__version__)
print("CUDA 사용 가능:", torch.cuda.is_available())

if torch.cuda.is_available():
    print("GPU:", torch.cuda.get_device_name(0))
PY

T600 GPU는 소규모 ACT 실험에는 사용할 수 있지만, 해상도와 배치 크기가 크면 메모리 부족이 발생할 수 있습니다.

메모리 부족 오류:

CUDA out of memory

이 경우:

배치 크기 축소
카메라 해상도 축소
카메라 수 축소
모델 크기 축소
CPU에서 먼저 파이프라인 테스트
23. 체크포인트

학습 중 모델 상태는 체크포인트로 저장됩니다.

예:

outputs/train/omx_cup_act/
├── checkpoints/
│   ├── 001000/
│   ├── 005000/
│   └── 010000/
├── logs/
└── config.yaml

체크포인트에는 보통 다음 정보가 포함됩니다.

모델 가중치
정책 설정
데이터 정규화 정보
학습 단계
프로세서 설정

최근 LeRobot에서는 정규화 처리가 모델 내부 가중치와 분리되는 등 호환성 변경이 있었으므로, 체크포인트 일부만 임의로 복사하기보다 전체 정책 디렉터리와 관련 설정을 함께 보관하는 것이 안전합니다.

24. 5차 실습: 학습된 모델 실행

실행 전 안전 준비:

작업 공간에서 사람 손 제거
비상 정지 방법 준비
로봇 속도 제한
그리퍼 힘 제한
관절 범위 확인
첫 실행은 컵 없이 진행

최근 공식 문서 기준 개념적인 실행 명령은 다음과 같습니다.

lerobot-rollout \
  --robot.type=omx_follower \
  --robot.port=/dev/ttyACM1 \
  --robot.id=my_omx_follower \
  --robot.cameras="{front: {type: opencv, index_or_path: /dev/video0, width: 640, height: 480, fps: 30}}" \
  --policy.path=outputs/train/omx_cup_act/checkpoints/last/pretrained_model \
  --dataset.repo_id=formakers/eval_omx_cup_act \
  --dataset.single_task="Pick up the cup and place it on the target."

정확한 옵션 확인:

lerobot-rollout --help

lerobot-rollout은 학습된 정책을 로봇에 배포하고 실행 결과를 기록하는 용도로 공식 문서에서 안내됩니다.

25. 실제 실행 과정
1. 카메라 영상 입력
2. 현재 관절값 읽기
3. 영상과 관절값 전처리
4. 정책 모델 추론
5. 다음 행동 생성
6. 안전 제한 적용
7. 모터에 명령 전송
8. 새로운 상태 관측
9. 작업 종료까지 반복

개념적인 Python 구조는 다음과 같습니다.

while True:
    observation = robot.get_observation()

    action = policy.select_action(observation)

    safe_action = apply_safety_limits(action)

    robot.send_action(safe_action)
26. 안전 제한이 반드시 필요한 이유

AI 모델은 학습하지 않은 상황에서 예상하지 못한 행동을 출력할 수 있습니다.

따라서 다음 제한을 적용해야 합니다.

관절 위치 제한
joint1 = max(min(joint1, joint1_max), joint1_min)
한 프레임 이동량 제한
delta = target_position - current_position
delta = max(min(delta, max_delta), -max_delta)
safe_target = current_position + delta

예를 들어 한 번에 최대 0.05rad만 움직이도록 제한할 수 있습니다.

속도 제한
처음 실행: 정상 속도의 10~20%
안정화 후: 30~50%
충분한 검증 후: 필요 속도로 조정
워크스페이스 제한

로봇 끝단이 다음 범위를 벗어나지 못하게 합니다.

책상 아래
로봇 베이스 뒤쪽
카메라 방향
사람이 있는 영역
관절 특이점 근처
27. 평가 방법

한두 번 성공했다고 학습이 끝난 것은 아닙니다.

예를 들어 20회 실행합니다.

성공: 14회
실패: 6회

성공률:

14 ÷ 20 × 100 = 70%

평가표 예:

번호	컵 위치	결과	실패 원인
1	중앙	성공	-
2	왼쪽 1cm	성공	-
3	오른쪽 2cm	실패	그리퍼 위치 불량
4	앞쪽 1cm	실패	접근 높이 부족
5	중앙	성공	-
실패 유형 분류
물체를 발견하지 못함
접근 위치가 틀림
그리퍼를 너무 일찍 닫음
컵을 잡았지만 이동 중 놓침
목표 위치가 틀림
동작이 중간에 멈춤
관절 진동 발생
28. 실패 데이터 개선 방법
접근 위치 실패
컵 위치를 조금씩 바꾼 시범 데이터 추가
카메라 화질 개선
그리퍼가 컵 중앙으로 접근하도록 시범 수정
그리퍼가 너무 일찍 닫힘
접근과 그리퍼 닫기 사이의 동작을 천천히 기록
그리퍼가 컵에 도달한 뒤 닫는 시범 추가
이동 중 컵을 놓침
그리퍼 닫힘 범위 확인
모터 토크 확인
컵을 천천히 이동하는 시범 추가
급격한 방향 전환 제거
새로운 위치에서 실패
시작 위치 변화를 포함한 에피소드 추가
컵의 회전 방향 변화 추가
카메라 관측 범위 확인

LeRobot의 최신 방향은 단순히 한 번 학습하고 끝내는 것이 아니라, 실행 실패를 다시 데이터로 만들고 평가와 개선을 반복하는 폐쇄 루프에 초점을 두고 있습니다. 2026년 7월 공개된 v0.6.0도 정책 실행, 평가, 실패 데이터 활용과 같은 학습 루프 강화를 주요 내용으로 소개했습니다.

29. 포메이커스님에게 권장하는 실제 실습 순서
17-1. 연결 점검
리더 포트 확인
팔로워 포트 확인
카메라 확인
모터 ID 확인
캘리브레이션 확인
17-2. 원격 조작
리더를 움직여 팔로워가 따라가는지 확인
각 관절 방향 확인
그리퍼 확인
동작 제한 확인
17-3. 테스트 기록
640×480
15~30 FPS
20초 내외
5개 에피소드
17-4. 데이터 검수
영상 확인
관절값 확인
프레임 속도 확인
실패 에피소드 제거 또는 재기록
17-5. 본 데이터 수집
30~50개 에피소드
컵 위치 ±1~2cm 변화
동작 속도 일정하게 유지
17-6. 테스트 학습
ACT
1,000스텝
GPU와 데이터 파이프라인 확인
17-7. 본 학습
10,000~50,000스텝부터 실험
중간 체크포인트 저장
실제 성공률 비교
17-8. 안전 실행
컵 없이 1차 실행
낮은 속도로 2차 실행
부드러운 물체로 3차 실행
실제 컵으로 4차 실행
17-9. 실패 분석
성공률 기록
실패 유형 분류
부족한 상황의 데이터 추가
재학습
30. ROS 2·MoveIt과 LeRobot의 차이
MoveIt 2
목표 위치를 사람이 지정
기구학과 충돌 검사를 이용해 경로 생성
정확한 위치 제어에 강함
설명 가능한 경로 계획
LeRobot
사람의 시범 데이터를 학습
카메라 영상을 보고 행동 생성
환경 변화에 적응할 가능성
복잡한 행동을 데이터로 학습
결합 구조

두 시스템은 경쟁 관계가 아니라 결합할 수 있습니다.

카메라
  ↓
LeRobot 정책
  ↓
목표 자세 또는 목표 위치 생성
  ↓
MoveIt 2
  ↓
충돌 없는 경로 계획
  ↓
ros2_control
  ↓
OMX 로봇팔

예를 들면 LeRobot이 “컵을 잡을 목표 위치”를 결정하고, MoveIt 2가 그 위치까지 안전한 관절 경로를 계산하도록 구성할 수 있습니다.

31. 17단계 최종 실습 과제
과제 이름

LeRobot을 이용한 컵 이동 모방학습

목표
리더 로봇팔로 컵 이동 시범을 기록하고,
ACT 정책을 학습한 뒤,
팔로워 로봇팔이 카메라 영상을 보고
컵을 지정 위치로 옮기게 한다.
실습 조건
카메라: 1대
해상도: 640×480
리더: OMX Leader
팔로워: OMX Follower
에피소드: 30개 이상
작업 시간: 에피소드당 약 20초
정책: ACT
초기 테스트 학습: 1,000스텝
본 학습: 10,000스텝 이상
평가: 최소 20회
완료 기준
리더-팔로워 조작 정상
데이터셋 저장 정상
Hugging Face 업로드 정상
ACT 학습 정상
학습 모델 로드 정상
20회 평가 성공률 기록
실패 원인 분석 완료
32. 단계별 폴더 구성 예

GitHub 교육 자료에는 다음과 같이 구성하면 좋습니다.

robot-making/
├── 01_robot_basics.md
├── 02_electronics_basics.md
├── ...
├── 16_ai_pick_and_place.md
├── 17_lerobot.md
└── examples/
    └── stage17_lerobot/
        ├── README.md
        ├── teleoperate.sh
        ├── record_dataset.sh
        ├── train_act.sh
        ├── rollout.sh
        └── troubleshooting.md

17단계 핵심 흐름은 다음 한 줄로 정리할 수 있습니다.

사람이 시범을 보인다
→ LeRobot이 영상과 관절 데이터를 기록한다
→ 정책 모델이 행동을 학습한다
→ 실제 로봇이 스스로 작업한다
→ 실패 데이터를 추가해 다시 학습한다

이 단계부터 로봇은 단순히 미리 작성된 좌표대로 움직이는 장치를 넘어, 사람의 작업 경험을 데이터로 배우는 AI 로봇으로 발전합니다.
