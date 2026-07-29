전체 구성
ROS 2 / MoveIt
      ↓
FollowJointTrajectory 액션
      ↓
Python ROS 2 액션 서버
      ↓
USB 시리얼 통신
      ↓
Arduino UNO
      ↓
CNC Shield V3
      ↓
스텝모터

현재 설정:

운영체제: Ubuntu 24.04
ROS 2: Jazzy
Arduino 포트: /dev/ttyACM2
시리얼 속도: 115200
모터 기본 스텝: 200 step/rev
마이크로스텝: 32분주
관절 이름: joint1
액션 이름: /arm_controller/follow_joint_trajectory
1단계: 하드웨어 연결

CNC Shield X축 기준 핀:

STEP   → Arduino D2
DIR    → Arduino D5
ENABLE → Arduino D8

모터 전원은 Arduino USB 전원이 아니라 CNC Shield의 모터 전원 단자에 별도로 연결합니다.

전원 + → CNC Shield VMOT 또는 +
전원 - → CNC Shield GND 또는 -

스텝모터 드라이버의 마이크로스텝 점퍼는 현재 32분주 설정입니다.

기본 모터 스텝: 200
마이크로스텝: 32
한 바퀴 스텝: 200 × 32 = 6400
2단계: Arduino 포트 확인

Arduino를 USB로 연결한 후 터미널에서 실행합니다.

ls -l /dev/ttyACM*

현재 Arduino 포트는 다음입니다.

/dev/ttyACM2

포트 번호는 USB를 다시 연결하면 바뀔 수 있으므로 실행 전에 항상 확인하는 것이 좋습니다.

3단계: Arduino 포트 권한 설정
sudo chmod 666 /dev/ttyACM2

권한 확인:

ls -l /dev/ttyACM2

정상적인 예:

crw-rw-rw- 1 root dialout ...
4단계: Arduino 코드 업로드

Arduino IDE를 실행한 뒤 새 스케치를 만들고 기존 코드를 모두 지웁니다.

다음 전체 코드를 붙여넣습니다.

/*
 * ROS 2 + Arduino UNO + CNC Shield V3
 * 스텝모터 절대 위치 제어
 *
 * 명령:
 * ENABLE
 * DISABLE
 * MOVE 목표스텝 이동시간ms
 * ZERO
 * STOP
 * STATUS
 */

const uint8_t STEP_PIN = 2;
const uint8_t DIR_PIN = 5;
const uint8_t ENABLE_PIN = 8;

const uint8_t ENABLE_ACTIVE_LEVEL = LOW;
const uint8_t ENABLE_INACTIVE_LEVEL = HIGH;

const uint8_t POSITIVE_DIRECTION_LEVEL = HIGH;
const uint8_t NEGATIVE_DIRECTION_LEVEL = LOW;

const unsigned int STEP_PULSE_WIDTH_US = 5;

/*
 * 탈조 방지 설정
 *
 * 속도가 너무 빠르거나 가속도가 너무 높으면
 * 모터가 위치를 놓칠 수 있습니다.
 */
const float MAX_STEP_SPEED = 500.0;
const float START_STEP_SPEED = 80.0;
const float ACCELERATION = 200.0;

long currentSteps = 0;
long targetSteps = 0;

long totalMoveSteps = 0;
long completedMoveSteps = 0;

bool moving = false;
bool driverEnabled = false;
bool positiveDirection = true;

float currentStepSpeed = 0.0;

unsigned long requestedDurationMs = 0;
unsigned long lastStepTimeUs = 0;
unsigned long stepIntervalUs = 1000000UL;

String inputLine = "";

void setup()
{
  pinMode(STEP_PIN, OUTPUT);
  pinMode(DIR_PIN, OUTPUT);
  pinMode(ENABLE_PIN, OUTPUT);

  digitalWrite(STEP_PIN, LOW);
  digitalWrite(DIR_PIN, POSITIVE_DIRECTION_LEVEL);
  digitalWrite(ENABLE_PIN, ENABLE_INACTIVE_LEVEL);

  Serial.begin(115200);

  inputLine.reserve(80);

  delay(500);

  Serial.println("READY");
}

void loop()
{
  readSerialCommands();
  updateMotor();
}

void readSerialCommands()
{
  while (Serial.available() > 0)
  {
    char receivedChar = Serial.read();

    if (receivedChar == '\n')
    {
      inputLine.trim();

      if (inputLine.length() > 0)
      {
        processCommand(inputLine);
      }

      inputLine = "";
    }
    else
    {
      if (receivedChar != '\r')
      {
        inputLine += receivedChar;
      }

      if (inputLine.length() > 79)
      {
        inputLine = "";
        Serial.println("ERROR COMMAND_TOO_LONG");
      }
    }
  }
}

void processCommand(String command)
{
  command.trim();

  if (command == "ENABLE")
  {
    enableMotor();
    Serial.println("OK ENABLED");
    return;
  }

  if (command == "DISABLE")
  {
    stopMotor();
    disableMotor();

    Serial.println("OK DISABLED");
    return;
  }

  if (command == "ZERO")
  {
    if (moving)
    {
      Serial.println("ERROR MOTOR_MOVING");
      return;
    }

    currentSteps = 0;
    targetSteps = 0;

    Serial.println("OK ZERO");
    return;
  }

  if (command == "STOP")
  {
    stopMotor();

    Serial.print("STOPPED ");
    Serial.println(currentSteps);
    return;
  }

  if (command == "STATUS")
  {
    printStatus();
    return;
  }

  if (command.startsWith("MOVE "))
  {
    processMoveCommand(command);
    return;
  }

  Serial.print("ERROR UNKNOWN_COMMAND ");
  Serial.println(command);
}

void processMoveCommand(String command)
{
  int firstSpace = command.indexOf(' ');
  int secondSpace = command.indexOf(' ', firstSpace + 1);

  if (firstSpace < 0 || secondSpace < 0)
  {
    Serial.println("ERROR MOVE_FORMAT");
    return;
  }

  String targetText = command.substring(
    firstSpace + 1,
    secondSpace
  );

  String durationText = command.substring(
    secondSpace + 1
  );

  targetText.trim();
  durationText.trim();

  if (targetText.length() == 0 ||
      durationText.length() == 0)
  {
    Serial.println("ERROR MOVE_FORMAT");
    return;
  }

  long requestedTargetSteps = targetText.toInt();

  unsigned long durationMs =
    (unsigned long)durationText.toInt();

  if (durationMs < 1)
  {
    Serial.println("ERROR INVALID_DURATION");
    return;
  }

  startMove(
    requestedTargetSteps,
    durationMs
  );
}

void startMove(
  long requestedTargetSteps,
  unsigned long durationMs
)
{
  if (!driverEnabled)
  {
    enableMotor();
  }

  moving = false;

  targetSteps = requestedTargetSteps;

  long difference = targetSteps - currentSteps;

  totalMoveSteps = labs(difference);
  completedMoveSteps = 0;

  requestedDurationMs = durationMs;

  if (totalMoveSteps == 0)
  {
    Serial.print("DONE ");
    Serial.println(currentSteps);
    return;
  }

  positiveDirection = difference > 0;

  if (positiveDirection)
  {
    digitalWrite(
      DIR_PIN,
      POSITIVE_DIRECTION_LEVEL
    );
  }
  else
  {
    digitalWrite(
      DIR_PIN,
      NEGATIVE_DIRECTION_LEVEL
    );
  }

  float requestedAverageSpeed =
    ((float)totalMoveSteps * 1000.0)
    / (float)durationMs;

  float allowedSpeed =
    min(requestedAverageSpeed, MAX_STEP_SPEED);

  if (allowedSpeed < START_STEP_SPEED)
  {
    currentStepSpeed = allowedSpeed;
  }
  else
  {
    currentStepSpeed = START_STEP_SPEED;
  }

  if (currentStepSpeed < 1.0)
  {
    currentStepSpeed = 1.0;
  }

  updateStepInterval();

  lastStepTimeUs = micros();

  moving = true;

  Serial.print("MOVING ");
  Serial.print(currentSteps);
  Serial.print(" ");
  Serial.print(targetSteps);
  Serial.print(" ");
  Serial.print(totalMoveSteps);
  Serial.print(" ");
  Serial.println(durationMs);
}

void updateMotor()
{
  if (!moving)
  {
    return;
  }

  unsigned long nowUs = micros();

  if ((unsigned long)(nowUs - lastStepTimeUs)
      < stepIntervalUs)
  {
    return;
  }

  lastStepTimeUs = nowUs;

  generateStepPulse();

  if (!moving)
  {
    return;
  }

  updateAcceleration();
}

void generateStepPulse()
{
  digitalWrite(STEP_PIN, HIGH);
  delayMicroseconds(STEP_PULSE_WIDTH_US);
  digitalWrite(STEP_PIN, LOW);

  if (positiveDirection)
  {
    currentSteps++;
  }
  else
  {
    currentSteps--;
  }

  completedMoveSteps++;

  if (completedMoveSteps >= totalMoveSteps)
  {
    currentSteps = targetSteps;
    moving = false;
    currentStepSpeed = 0.0;

    Serial.print("DONE ");
    Serial.println(currentSteps);
  }
}

void updateAcceleration()
{
  long remainingSteps =
    totalMoveSteps - completedMoveSteps;

  if (remainingSteps <= 0)
  {
    return;
  }

  float brakingDistance =
    (currentStepSpeed * currentStepSpeed)
    / (2.0 * ACCELERATION);

  float deltaTime =
    (float)stepIntervalUs / 1000000.0;

  float speedChange =
    ACCELERATION * deltaTime;

  if ((float)remainingSteps <= brakingDistance)
  {
    currentStepSpeed -= speedChange;

    if (currentStepSpeed < START_STEP_SPEED)
    {
      currentStepSpeed = START_STEP_SPEED;
    }
  }
  else
  {
    currentStepSpeed += speedChange;

    float requestedAverageSpeed =
      ((float)totalMoveSteps * 1000.0)
      / (float)requestedDurationMs;

    float targetSpeed =
      min(requestedAverageSpeed, MAX_STEP_SPEED);

    if (targetSpeed < START_STEP_SPEED)
    {
      targetSpeed = START_STEP_SPEED;
    }

    if (currentStepSpeed > targetSpeed)
    {
      currentStepSpeed = targetSpeed;
    }
  }

  updateStepInterval();
}

void updateStepInterval()
{
  if (currentStepSpeed < 1.0)
  {
    currentStepSpeed = 1.0;
  }

  stepIntervalUs =
    (unsigned long)(
      1000000.0 / currentStepSpeed
    );

  const unsigned long ABSOLUTE_MIN_INTERVAL_US = 1000;

  if (stepIntervalUs < ABSOLUTE_MIN_INTERVAL_US)
  {
    stepIntervalUs =
      ABSOLUTE_MIN_INTERVAL_US;
  }
}

void stopMotor()
{
  moving = false;

  targetSteps = currentSteps;
  totalMoveSteps = 0;
  completedMoveSteps = 0;
  currentStepSpeed = 0.0;

  digitalWrite(STEP_PIN, LOW);
}

void enableMotor()
{
  digitalWrite(
    ENABLE_PIN,
    ENABLE_ACTIVE_LEVEL
  );

  driverEnabled = true;

  delay(5);
}

void disableMotor()
{
  digitalWrite(
    ENABLE_PIN,
    ENABLE_INACTIVE_LEVEL
  );

  driverEnabled = false;
}

void printStatus()
{
  Serial.print("STATUS ");
  Serial.print(currentSteps);
  Serial.print(" ");
  Serial.print(targetSteps);
  Serial.print(" ");

  if (moving)
  {
    Serial.print("MOVING");
  }
  else
  {
    Serial.print("IDLE");
  }

  Serial.print(" ");

  if (driverEnabled)
  {
    Serial.print("ENABLED");
  }
  else
  {
    Serial.print("DISABLED");
  }

  Serial.print(" ");

  Serial.print(currentStepSpeed, 2);

  Serial.println();
}

Arduino IDE에서 다음 설정을 확인합니다.

보드: Arduino UNO
포트: /dev/ttyACM2
보드레이트: 115200

업로드 버튼을 누릅니다.

5단계: Arduino 단독 테스트

Arduino IDE의 시리얼 모니터를 엽니다.

시리얼 모니터 설정:

보드레이트: 115200
줄바꿈: Newline

상태 확인:

STATUS

정상 응답:

STATUS 0 0 IDLE DISABLED 0.00

모터 드라이버 활성화:

ENABLE

정상 응답:

OK ENABLED

509스텝을 5초 동안 이동:

MOVE 509 5000

정상 응답:

MOVING 0 509 509 5000
DONE 509

원점 복귀:

MOVE 0 5000

정상 응답:

MOVING 509 0 509 5000
DONE 0

현재 위치를 0으로 초기화:

ZERO

즉시 정지:

STOP

모터 드라이버 비활성화:

DISABLE

Arduino 단독 테스트가 끝나면 시리얼 모니터를 반드시 닫습니다.

6단계: 포트 점유 확인

Arduino IDE의 시리얼 모니터가 닫힌 상태에서 실행합니다.

sudo lsof /dev/ttyACM2

아무 내용도 출력되지 않으면 정상입니다.

기존 ROS 2 서버가 실행 중이면 종료합니다.

pkill -f stepper_trajectory_server 2>/dev/null || true
7단계: ROS 2 작업공간 확인

현재 작업공간으로 이동합니다.

cd ~/cnc_moveit_ws

폴더 확인:

ls

소스 파일 확인:

find ~/cnc_moveit_ws/src -maxdepth 4 -type f | sort

현재 패키지 이름:

cnc_stepper_controller

현재 실행 파일 이름:

stepper_trajectory_server
8단계: 필요한 ROS 2 패키지 설치
sudo apt update
sudo apt install -y \
  python3-serial \
  ros-jazzy-control-msgs \
  ros-jazzy-sensor-msgs \
  python3-colcon-common-extensions
9단계: ROS 2 패키지 빌드

ROS 2 Jazzy 환경을 불러옵니다.

source /opt/ros/jazzy/setup.bash

작업공간으로 이동합니다.

cd ~/cnc_moveit_ws

기존 빌드 결과 삭제:

rm -rf build install log

패키지 빌드:

colcon build \
  --symlink-install \
  --packages-select cnc_stepper_controller

정상 결과:

Summary: 1 package finished

빌드한 작업공간 환경을 불러옵니다.

source ~/cnc_moveit_ws/install/setup.bash

실행 파일 확인:

ros2 pkg executables cnc_stepper_controller

정상 결과:

cnc_stepper_controller stepper_trajectory_server
10단계: 터미널 1에서 액션 서버 실행

새 터미널을 열고 다음 명령을 순서대로 실행합니다.

기존 서버 종료:

pkill -f stepper_trajectory_server 2>/dev/null || true

포트 권한 설정:

sudo chmod 666 /dev/ttyACM2

ROS 2 환경 설정:

source /opt/ros/jazzy/setup.bash

작업공간 환경 설정:

source ~/cnc_moveit_ws/install/setup.bash

액션 서버 실행:

ros2 run cnc_stepper_controller \
  stepper_trajectory_server \
  --ros-args \
  -p serial_port:=/dev/ttyACM2 \
  -p baud_rate:=115200 \
  -p joint_name:=joint1 \
  -p motor_steps_per_revolution:=200.0 \
  -p microsteps:=32.0 \
  -p gear_ratio:=1.0 \
  -p direction:=1.0 \
  -p arduino_timeout_sec:=30.0

정상 로그의 예:

Arduino 연결 성공: /dev/ttyACM2
Arduino TX: ENABLE
Arduino RX: OK ENABLED
액션 서버 준비 완료

이 터미널은 계속 열어둡니다.

11단계: 터미널 2에서 액션 서버 확인

새 터미널을 하나 더 엽니다.

ROS 2 환경 설정:

source /opt/ros/jazzy/setup.bash

작업공간 환경 설정:

source ~/cnc_moveit_ws/install/setup.bash

액션 목록 확인:

ros2 action list

정상적으로 다음 액션이 보여야 합니다.

/arm_controller/follow_joint_trajectory

액션 타입 확인:

ros2 action type \
  /arm_controller/follow_joint_trajectory

정상 결과:

control_msgs/action/FollowJointTrajectory

액션 서버 정보 확인:

ros2 action info \
  /arm_controller/follow_joint_trajectory
12단계: 0.2라디안 이동

작은 각도로 먼저 시험합니다.

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.2], time_from_start: {sec: 5, nanosec: 0}}]}}" \
  --feedback

32분주 기준으로 0.2라디안은 약 204스텝입니다.

0.2 × 6400 ÷ 2π
약 204스텝

정상 결과:

Goal accepted
Goal finished with status: SUCCEEDED

서버 터미널에는 다음과 비슷하게 표시됩니다.

목표=0.2000 rad
스텝=204
시간=5000 ms
Arduino TX: MOVE 204 5000
Arduino RX: MOVING ...
Arduino RX: DONE 204
13단계: 원점 복귀
ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.0], time_from_start: {sec: 5, nanosec: 0}}]}}" \
  --feedback

Arduino에는 다음 명령이 전달됩니다.

MOVE 0 5000
14단계: 0.5라디안 이동
ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.5], time_from_start: {sec: 7, nanosec: 0}}]}}" \
  --feedback

32분주 기준으로 약 509스텝입니다.

0.5 × 6400 ÷ 2π
약 509스텝

원점 복귀:

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.0], time_from_start: {sec: 7, nanosec: 0}}]}}" \
  --feedback
15단계: 반대 방향 이동
ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [-0.5], time_from_start: {sec: 7, nanosec: 0}}]}}" \
  --feedback
16단계: 여러 지점 순차 이동
ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [
  {positions: [0.2], time_from_start: {sec: 3, nanosec: 0}},
  {positions: [0.5], time_from_start: {sec: 7, nanosec: 0}},
  {positions: [0.0], time_from_start: {sec: 12, nanosec: 0}}
  ]}}" \
  --feedback

이동 순서:

현재 위치
→ 0.2 rad
→ 0.5 rad
→ 0.0 rad
17단계: 왕복 테스트
source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.5], time_from_start: {sec: 7, nanosec: 0}}]}}" \
  --feedback

sleep 2

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.0], time_from_start: {sec: 7, nanosec: 0}}]}}" \
  --feedback
18단계: 탈조가 발생하면 속도 낮추기

Arduino 코드 상단의 다음 부분을 찾습니다.

const float MAX_STEP_SPEED = 500.0;
const float START_STEP_SPEED = 80.0;
const float ACCELERATION = 200.0;

더 안정적으로 설정:

const float MAX_STEP_SPEED = 300.0;
const float START_STEP_SPEED = 40.0;
const float ACCELERATION = 100.0;

코드 수정 후 Arduino에 다시 업로드합니다.

ROS 2 이동 시간도 늘립니다.

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.5], time_from_start: {sec: 10, nanosec: 0}}]}}" \
  --feedback
19단계: 회전 방향 변경

현재 서버 설정:

-p direction:=1.0

반대로 회전시키려면 액션 서버 실행 명령에서 변경합니다.

-p direction:=-1.0

전체 실행:

pkill -f stepper_trajectory_server 2>/dev/null || true

sudo chmod 666 /dev/ttyACM2

source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

ros2 run cnc_stepper_controller \
  stepper_trajectory_server \
  --ros-args \
  -p serial_port:=/dev/ttyACM2 \
  -p baud_rate:=115200 \
  -p joint_name:=joint1 \
  -p motor_steps_per_revolution:=200.0 \
  -p microsteps:=32.0 \
  -p gear_ratio:=1.0 \
  -p direction:=-1.0 \
  -p arduino_timeout_sec:=30.0
20단계: Arduino 직접 통신 테스트

ROS 2 액션 서버를 먼저 종료합니다.

pkill -f stepper_trajectory_server 2>/dev/null || true

포트 권한 설정:

sudo chmod 666 /dev/ttyACM2

직접 통신 테스트:

python3 - <<'PY'
import time
import serial

PORT = "/dev/ttyACM2"
BAUD = 115200

ser = serial.Serial(
    port=PORT,
    baudrate=BAUD,
    timeout=0.5,
    write_timeout=1.0
)

print("포트 연결:", PORT)
print("Arduino 재시작 대기")

time.sleep(3)

ser.reset_input_buffer()

def send(command):
    print("TX:", command)
    ser.write((command + "\r\n").encode())
    ser.flush()

def read_for(seconds):
    end_time = time.time() + seconds

    while time.time() < end_time:
        line = ser.readline().decode(
            errors="replace"
        ).strip()

        if line:
            print("RX:", line)

send("STATUS")
read_for(2)

send("ENABLE")
read_for(2)

send("MOVE 509 5000")

done = False
end_time = time.time() + 15

while time.time() < end_time:
    line = ser.readline().decode(
        errors="replace"
    ).strip()

    if not line:
        continue

    print("RX:", line)

    if line.startswith("DONE"):
        done = True
        break

print("결과:", "성공" if done else "DONE 응답 없음")

ser.close()
PY

정상 결과:

TX: STATUS
RX: STATUS ...

TX: ENABLE
RX: OK ENABLED

TX: MOVE 509 5000
RX: MOVING ...
RX: DONE 509

결과: 성공
21단계: 서버 종료

액션 서버가 실행 중인 터미널에서:

Ctrl + C

다른 터미널에서 강제 종료:

pkill -f stepper_trajectory_server

포트가 해제됐는지 확인:

sudo lsof /dev/ttyACM2

아무 내용도 나오지 않으면 정상입니다.

매번 사용할 핵심 명령
터미널 1: 액션 서버 실행
pkill -f stepper_trajectory_server 2>/dev/null || true

sudo chmod 666 /dev/ttyACM2

source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

ros2 run cnc_stepper_controller \
  stepper_trajectory_server \
  --ros-args \
  -p serial_port:=/dev/ttyACM2 \
  -p baud_rate:=115200 \
  -p joint_name:=joint1 \
  -p motor_steps_per_revolution:=200.0 \
  -p microsteps:=32.0 \
  -p gear_ratio:=1.0 \
  -p direction:=1.0 \
  -p arduino_timeout_sec:=30.0
터미널 2: 0.5라디안 이동
source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.5], time_from_start: {sec: 7, nanosec: 0}}]}}" \
  --feedback
터미널 2: 원점 복귀
source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.0], time_from_start: {sec: 7, nanosec: 0}}]}}" \
  --feedback

현재까지 완성된 핵심은 ROS 2의 관절 목표값을 Arduino 시리얼 명령으로 변환하고, CNC Shield의 스텝모터를 가속·감속 방식으로 움직이는 것입니다.

현재 설정에서는 360도 = 2π rad = 약 6.283185 rad입니다.

32분주 기준으로 모터 한 바퀴는 다음과 같습니다.

200 × 32 = 6400스텝

아래 명령은 현재 위치 0 rad를 기준으로:

0 rad → +6.283185 rad : 정방향 360도
+6.283185 rad → 0 rad : 역방향 360도

로 움직입니다.

터미널 1: 액션 서버 실행
pkill -f stepper_trajectory_server 2>/dev/null || true

sudo chmod 666 /dev/ttyACM2

source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

ros2 run cnc_stepper_controller \
  stepper_trajectory_server \
  --ros-args \
  -p serial_port:=/dev/ttyACM2 \
  -p baud_rate:=115200 \
  -p joint_name:=joint1 \
  -p motor_steps_per_revolution:=200.0 \
  -p microsteps:=32.0 \
  -p gear_ratio:=1.0 \
  -p direction:=1.0 \
  -p arduino_timeout_sec:=30.0

터미널 1은 실행 상태로 그대로 둡니다.

터미널 2: 정방향 360도

처음에는 탈조를 방지하기 위해 15초 정도로 천천히 시험합니다.

source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [6.283185307], time_from_start: {sec: 15, nanosec: 0}}]}}" \
  --feedback

서버에서는 약 6400스텝으로 변환됩니다.

목표 = 6.283185 rad
스텝 = 6400
Arduino 명령 = MOVE 6400 15000
터미널 2: 역방향으로 원점 복귀

정방향으로 360도 회전한 다음 0 rad로 돌아오면 역방향으로 360도 움직입니다.

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.0], time_from_start: {sec: 15, nanosec: 0}}]}}" \
  --feedback

Arduino 명령은 다음과 같이 전달됩니다.

MOVE 0 15000
정방향 360도 후 역방향 360도 자동 실행

아래 명령 하나를 복사하면 정방향 360도 회전하고, 2초 멈춘 뒤 역방향으로 원점에 복귀합니다.

source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

echo "정방향 360도 회전"

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [6.283185307], time_from_start: {sec: 15, nanosec: 0}}]}}" \
  --feedback

sleep 2

echo "역방향 360도 회전"

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.0], time_from_start: {sec: 15, nanosec: 0}}]}}" \
  --feedback
현재 위치에서 역방향 360도

현재 위치가 0 rad일 때 바로 역방향으로 한 바퀴 이동하려면 -2π rad를 보냅니다.

source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [-6.283185307], time_from_start: {sec: 15, nanosec: 0}}]}}" \
  --feedback

원점으로 복귀:

ros2 action send_goal \
  /arm_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.0], time_from_start: {sec: 15, nanosec: 0}}]}}" \
  --feedback
정방향과 역방향을 반복 실행

아래 코드는 정방향 360도와 역방향 360도를 3회 반복합니다.

source /opt/ros/jazzy/setup.bash
source ~/cnc_moveit_ws/install/setup.bash

for i in 1 2 3
do
  echo "===== ${i}회차: 정방향 360도 ====="

  ros2 action send_goal \
    /arm_controller/follow_joint_trajectory \
    control_msgs/action/FollowJointTrajectory \
    "{trajectory: {joint_names: ['joint1'], points: [{positions: [6.283185307], time_from_start: {sec: 15, nanosec: 0}}]}}" \
    --feedback

  sleep 2

  echo "===== ${i}회차: 역방향 360도 ====="

  ros2 action send_goal \
    /arm_controller/follow_joint_trajectory \
    control_msgs/action/FollowJointTrajectory \
    "{trajectory: {joint_names: ['joint1'], points: [{positions: [0.0], time_from_start: {sec: 15, nanosec: 0}}]}}" \
    --feedback

  sleep 2
done
Arduino에 직접 360도 명령 보내기

ROS 2를 사용하지 않고 Arduino만 직접 시험할 때 사용합니다.

액션 서버를 먼저 종료합니다.

pkill -f stepper_trajectory_server 2>/dev/null || true

포트 권한을 설정합니다.

sudo chmod 666 /dev/ttyACM2

정방향 6400스텝 후 역방향 원점 복귀:

python3 - <<'PY'
import time
import serial

PORT = "/dev/ttyACM2"
BAUD = 115200

ser = serial.Serial(
    PORT,
    BAUD,
    timeout=0.5,
    write_timeout=1.0
)

print("Arduino 재시작 대기")
time.sleep(3)
ser.reset_input_buffer()

def send(command):
    print("TX:", command)
    ser.write((command + "\r\n").encode())
    ser.flush()

def wait_done(timeout_seconds):
    end_time = time.time() + timeout_seconds

    while time.time() < end_time:
        line = ser.readline().decode(
            errors="replace"
        ).strip()

        if not line:
            continue

        print("RX:", line)

        if line.startswith("DONE"):
            return True

    return False

send("ENABLE")
time.sleep(1)

print("정방향 360도")
send("MOVE 6400 15000")

if not wait_done(25):
    print("정방향 이동 시간 초과")
    ser.close()
    raise SystemExit(1)

time.sleep(2)

print("역방향 360도")
send("MOVE 0 15000")

if not wait_done(25):
    print("역방향 이동 시간 초과")
    ser.close()
    raise SystemExit(1)

print("왕복 이동 완료")

ser.close()
PY
중요한 점

현재 Arduino 명령은 절대 위치 방식입니다.

MOVE 6400 15000

이 명령은 현재 위치에서 6400스텝을 더 움직이라는 뜻이 아니라, 절대 위치 6400스텝으로 이동하라는 뜻입니다.

따라서 같은 명령을 두 번 보내면 두 번째에는 움직이지 않습니다.

현재 위치 0      → MOVE 6400 → 한 바퀴 이동
현재 위치 6400   → MOVE 6400 → 이동 없음

두 바퀴를 연속해서 정방향으로 돌리려면 목표 위치를 계속 증가시켜야 합니다.

1바퀴: 6400
2바퀴: 12800
3바퀴: 19200

ROS 2에서는 다음과 같습니다.

1바퀴: 6.283185 rad
2바퀴: 12.566371 rad
3바퀴: 18.849556 rad


