4단계 : 모터의 종류와 제어
학습 목표

이번 단계에서는 로봇을 실제로 움직이게 만드는 핵심 부품인 모터(Motor)를 배웁니다.

이 단계를 마치면 다음과 같은 것을 할 수 있습니다.

DC 모터와 스텝모터의 차이 이해
서보모터의 동작 원리 이해
BLDC 모터의 특징 이해
모터 드라이버 사용법
PWM 제어
속도 제어
방향 제어
위치 제어
Arduino로 모터 제어
왜 모터를 먼저 배워야 할까?

센서는 정보를 입력합니다.

모터는 로봇을 움직입니다.

즉

센서 = 눈

CPU = 뇌

모터 = 근육

입니다.

AI가 아무리 똑똑해도

모터를 움직이지 못하면

로봇은 아무 일도 하지 못합니다.

로봇에서 사용하는 대표 모터
모터
 │
 ├─ DC Motor
 │
 ├─ Servo Motor
 │
 ├─ Step Motor
 │
 └─ BLDC Motor

이 네 가지를 반드시 알아야 합니다.

1. DC 모터

가장 단순한 모터입니다.

특징
계속 회전
방향 변경 가능
속도 조절 가능
위치 제어 불가능

예

장난감 자동차

선풍기

환풍기

RC카
동작 원리

전압을 인가하면

계속 회전합니다.

+ ----- Motor ---- -


전압을 반대로 연결하면

- ----- Motor ---- +


반대 방향으로 회전합니다.

속도 조절

PWM으로 합니다.

PWM 30%

느리게

PWM 80%

빠르게
PWM(Pulse Width Modulation)

모터 속도 제어의 핵심입니다.

████    ████    ████

ON OFF ON OFF

듀티비가 높을수록

평균 전압이 높아집니다.

20%

█░░░░░░░░

50%

████░░░░

80%

████████░

PWM은

Arduino의

analogWrite()


또는

ESP32의

ledcWrite()


를 사용합니다.

DC 모터 제어 회로

Arduino는 모터를 직접 구동하지 못합니다.

반드시

모터 드라이버가 필요합니다.

대표적으로

L298N

TB6612FNG

DRV8833
연결
Arduino

↓

L298N

↓

Motor
Arduino 예제
int ENA=5;
int IN1=8;
int IN2=9;

void setup()
{
 pinMode(ENA,OUTPUT);
 pinMode(IN1,OUTPUT);
 pinMode(IN2,OUTPUT);
}

void loop()
{
 digitalWrite(IN1,HIGH);
 digitalWrite(IN2,LOW);

 analogWrite(ENA,180);

 delay(3000);

 analogWrite(ENA,0);

 delay(2000);
}
2. Servo Motor

가장 많이 사용하는 모터입니다.

특징

각도 제어
위치 유지
내부 기어
내부 제어기 포함
회전 범위

대부분

0°

↓

180°

입니다.

PWM 신호

보통

50Hz

입니다.

펄스폭

1ms

↓

0°

1.5ms

↓

90°

2ms

↓

180°
Arduino 코드
#include <Servo.h>

Servo servo;

void setup()
{
 servo.attach(9);
}

void loop()
{
 servo.write(0);
 delay(1000);

 servo.write(90);
 delay(1000);

 servo.write(180);
 delay(1000);
}
어디에 사용할까?
로봇팔

RC조향

카메라 짐벌

자동문

집게
3. 스텝모터

로봇 제작에서 가장 중요한 모터입니다.

왜냐하면

정확한 위치 제어가 가능합니다.

원리

한 번에 조금씩 움직입니다.

Step

↓

Step

↓

Step

↓

Step
예
1.8°

↓

1 Step

즉

200 Step

↓

1바퀴
마이크로스텝

DRV8825

A4988

에서는

1

1/2

1/4

1/8

1/16

1/32

까지 가능합니다.

장점

매우 정밀

Arduino 제어

대표 라이브러리

AccelStepper

예제

#include <AccelStepper.h>

AccelStepper stepper(1,2,3);

void setup()
{
 stepper.setMaxSpeed(1000);
}

void loop()
{
 stepper.moveTo(1000);
 stepper.run();
}
어디에 사용할까?
3D 프린터

CNC

XY테이블

자동화 장비

카메라 슬라이더
4. BLDC 모터

Brushless DC Motor

브러시가 없는 모터입니다.

장점
매우 빠름
조용함
긴 수명
높은 효율
사용 분야
드론

전기차

로봇

산업용 장비
제어

ESC(Electronic Speed Controller)가 필요합니다.

Arduino

↓

ESC

↓

BLDC
모터 비교
종류	속도	위치제어	정밀도	대표 용도
DC	★★★★★	❌	낮음	자동차
Servo	★★★	⭕	높음	로봇팔
Stepper	★★	⭕	매우 높음	CNC
BLDC	★★★★★	△(제어기 필요)	높음	드론, 전기차
모터 드라이버

모터는 전류가 많이 필요합니다.

Arduino 출력은 약 20mA 정도에 불과하므로 모터를 직접 연결하면 보드가 손상될 수 있습니다. 따라서 반드시 모터 드라이버를 사용해야 합니다.

대표 드라이버

L298N

TB6612FNG

DRV8825

A4988

TMC2209
실제 로봇에서의 사용 예
이동 로봇
DC 모터

또는

BLDC
3D 프린터
Stepper
CNC
Stepper
휴머노이드
Servo

BLDC
산업용 로봇팔
Servo

BLDC
이번 단계 실습
실습 1
DC 모터를 L298N으로 정방향/역방향 회전시키기
실습 2
PWM으로 DC 모터 속도 제어하기
실습 3
SG90 서보모터를 0° → 90° → 180°로 움직이기
실습 4
A4988 또는 DRV8825를 이용해 스텝모터를 일정 스텝만큼 회전시키기
실습 5
가변저항을 연결하여 서보모터 또는 스텝모터의 위치를 제어하기
이번 단계 핵심 정리
모터는 로봇의 구동 장치(근육)이다.
DC 모터는 연속 회전에 적합하며 PWM으로 속도를 제어한다.
서보모터는 각도 제어가 가능하여 로봇팔과 조향 장치에 많이 사용된다.
스텝모터는 정밀한 위치 제어가 가능하여 CNC, 3D 프린터, 자동화 장비에 적합하다.
BLDC 모터는 고효율·고속 특성으로 드론, 전기차, 산업용 로봇 등에 널리 사용된다.
모터를 안전하게 구동하려면 반드시 모터 드라이버를 사용해야 한다.
GitHub 학습 파일 구성

이번 단계는 아래와 같이 저장하면 됩니다.

robot-making/
├── README.md
├── 01_robot_basics.md
├── 02_electronics_basics.md
├── 03_arduino_basics.md
└── 04_motor_basics.md

다음 5단계에서는 "센서와 입력장치"를 배우며, 초음파 센서, 적외선 센서, 엔코더, IMU, 카메라 등을 활용해 로봇이 주변 환경을 인식하는 방법을 실습과 함께 자세히 학습하겠습니다.
