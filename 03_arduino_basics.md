AI 로봇 제작 마스터 클래스
제3단계 : 아두이노 기초 (Arduino Basics)

이번 단계에서는 로봇의 두뇌 역할을 하는 아두이노를 배웁니다.

앞에서 배운

1단계 : 로봇의 기초 이해
2단계 : 전기와 전자 기초

를 바탕으로 이제 직접 전자회로를 만들고 모터와 센서를 제어하는 단계입니다.

학습 목표

이번 과정을 마치면 다음과 같은 것을 직접 만들 수 있습니다.

LED 제어
버튼 입력
부저 제어
서보모터 제어
DC모터 제어
스텝모터 제어
센서 읽기
RC카 제작
로봇팔 제어
AI 로봇의 하드웨어 제어
목차
아두이노란?
아두이노 종류
아두이노 구조
핀(Pin)의 이해
IDE 설치
첫 프로그램
LED 제어
버튼 입력
부저
PWM
아날로그 입력
센서 연결
서보모터
DC모터
스텝모터
시리얼 통신
Python 연동
Raspberry Pi 연동
ROS2 연동
실습 프로젝트
1. 아두이노란?

아두이노는 마이크로컨트롤러(MCU) 입니다.

쉽게 말하면

작은 컴퓨터입니다.

PC처럼 운영체제가 있는 것이 아니라

프로그램 하나만 실행하는 초소형 컴퓨터입니다.

대표적인 용도

LED 제어
센서 읽기
모터 제어
로봇 제어
자동화
아두이노의 역할

예를 들어

버튼을 누르면

↓

LED 켜기

↓

모터 회전

↓

부저 울림

↓

센서 읽기

↓

PC로 데이터 전송

이 모든 것을 수행합니다.

2. 아두이노 종류

가장 많이 사용하는 모델입니다.

모델	특징
UNO	입문 최고
Nano	작고 저렴
Mega2560	핀 많음
Leonardo	USB HID 가능
Due	32bit ARM
ESP32	WiFi/Bluetooth 내장
추천

처음이라면

Arduino UNO R3

를 추천합니다.

3. 아두이노 구조
+---------------------------+

 USB

 ATmega328P

 Digital Pin

 Analog Pin

 Reset

 Power

 ICSP

+---------------------------+
주요 부품
USB

프로그램 업로드

전원 공급

MCU

ATmega328P

모든 계산 수행

전원

5V

3.3V

VIN

GND

Digital Pin

ON/OFF

0 또는 1

Analog Pin

센서값 읽기

0~1023

PWM

가짜 아날로그 출력

LED 밝기

모터 속도

4. 핀 번호

UNO 기준

0~13

Digital

A0~A5

Analog

PWM

3

5

6

9

10

11
5. IDE 설치

Arduino IDE 설치

Arduino IDE 다운로드

설치

실행

보드 선택

Arduino UNO

포트 선택

COM

또는

/dev/ttyACM0

Linux에서는 보통

/dev/ttyACM0

또는

/dev/ttyUSB0

입니다.

6. 첫 프로그램
void setup()
{
  pinMode(13, OUTPUT);
}

void loop()
{
  digitalWrite(13, HIGH);

  delay(1000);

  digitalWrite(13, LOW);

  delay(1000);
}

업로드하면

LED가

1초마다 깜박입니다.

코드 설명

setup()

한 번만 실행

loop()

무한 반복

pinMode()

핀 설정

digitalWrite()

HIGH

켜기

LOW

끄기

delay()

기다리기

7. LED 실습
회로
D13

↓

220Ω

↓

LED

↓

GND
코드
void setup()
{
  pinMode(13, OUTPUT);
}

void loop()
{
  digitalWrite(13,HIGH);
  delay(500);

  digitalWrite(13,LOW);
  delay(500);
}
8. 버튼 입력

회로

5V

↓

Button

↓

D2

↓

10KΩ

↓

GND

코드

void setup()
{
  pinMode(2,INPUT);
}

void loop()
{
  int sw=digitalRead(2);

  if(sw==HIGH)
  {
      digitalWrite(13,HIGH);
  }
  else
  {
      digitalWrite(13,LOW);
  }
}
9. 부저 제어
tone(8,1000);

delay(500);

noTone(8);

1000Hz

소리를 냅니다.

10. PWM

PWM은

빠르게 ON/OFF하여

전압처럼 보이게 하는 기술입니다.

예)

analogWrite(9,128);

128

=

50%

밝기

11. 아날로그 입력

가변저항 연결

A0

코드

int value;

void loop()
{
    value=analogRead(A0);

    Serial.println(value);
}

결과

0

512

1023
12. 센서 연결

대표 센서

초음파
적외선
조도
온습도
자이로
IMU
엔코더
13. 서보모터
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
14. DC모터

드라이버

L298N

BTS7960

DRV8871

PWM으로 속도 제어

방향 제어 가능

15. 스텝모터

대표 드라이버

A4988
DRV8825
TMC2209

여러분이 앞으로 제작할 CNC 및 XY 스테이지, 로봇의 정밀 제어에 가장 많이 사용하는 모터입니다.

예제

digitalWrite(STEP,HIGH);

delayMicroseconds(500);

digitalWrite(STEP,LOW);
16. 시리얼 통신
Serial.begin(115200);

PC와 통신

Serial.println("Hello");

PC에서

Hello

출력됩니다.

17. Python 연동

Python

↓

Serial

↓

Arduino

↓

모터

↓

센서

↓

Python

여러분이 앞으로 많이 사용할 구조입니다.

예)

OpenCV

↓

YOLO

↓

Python

↓

Arduino

↓

Stepper

↓

카메라 추적

바로 여러분이 진행해 온 프로젝트의 핵심 구조입니다.

18. Raspberry Pi 연동
Camera

↓

Python

↓

AI

↓

Arduino

↓

Motor

AI 처리

↓

Arduino 제어

19. ROS2 연동
ROS2 Node

↓

Python

↓

Serial

↓

Arduino

↓

Robot

나중에

MoveIt

LeRobot

휴머노이드

모두 이 구조를 사용합니다.

20. 종합 실습 프로젝트

이번 단계에서 만들어 볼 프로젝트입니다.

초급
LED 깜박이기
버튼으로 LED 켜기
부저 제어
가변저항 읽기
중급
서보모터 각도 제어
초음파 센서 거리 측정
L298N으로 DC모터 제어
A4988으로 스텝모터 회전
고급
Python ↔ Arduino 시리얼 통신
OpenCV로 카메라 영상 처리 후 Arduino 제어
YOLO 컵 인식 후 스텝모터 자동 추적
ROS2에서 Arduino를 통해 모터 제어
이번 단계 핵심 정리

이번 3단계에서는 다음 내용을 익혔습니다.

아두이노의 구조와 역할
디지털/아날로그 입출력
PWM 제어
LED, 버튼, 부저 제어
센서 데이터 읽기
서보모터, DC모터, 스텝모터 제어
시리얼 통신
Python, Raspberry Pi, ROS2와의 연동 개념

이 과정을 마치면 단순한 회로 실습을 넘어 실제 AI 로봇을 제어하는 하드웨어 프로그래밍의 기초를 갖추게 됩니다.

다음 단계 (4단계)

모터의 종류와 제어

다음 단계에서는 다음 내용을 심화 학습합니다.

DC 모터의 원리와 제어
브러시리스(BLDC) 모터
서보모터와 산업용 서보
스텝모터의 원리와 마이크로스테핑
엔코더와 폐루프 제어
H-브리지 모터 드라이버
A4988, DRV8825, TMC2209 사용법
로봇 바퀴, 로봇팔, AGV, CNC에 사용되는 모터 제어 실습
ROS2와 MoveIt에서의 모터 제어 기초

이 4단계는 이후 센서, ROS2, MoveIt, AI Pick & Place, LeRobot, 휴머노이드 제작으로 이어지는 핵심 기반이 됩니
