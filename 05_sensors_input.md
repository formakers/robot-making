AI 로봇 제작 마스터 클래스
5단계 : 센서와 입력장치 (Sensors & Input Devices)

지금까지 학습한 내용

✅ 1단계 : 로봇의 기초 이해
✅ 2단계 : 전기와 전자 기초
✅ 3단계 : 아두이노 기초
✅ 4단계 : 모터의 종류와 제어

이번에는 로봇의 눈, 귀, 피부 역할을 하는 센서를 배우는 단계입니다.

5단계 목표

이번 단계가 끝나면 다음을 할 수 있습니다.

다양한 센서의 원리 이해
센서를 아두이노와 연결
센서값 읽기
여러 센서를 동시에 사용
센서값으로 모터 제어
AI 프로젝트의 입력장치 이해
센서(Sensor)란?

센서는

주변 환경을 전기신호로 바꾸는 장치

입니다.

예를 들어

사람은

눈
귀
피부
코

로 세상을 인식합니다.

로봇은

카메라
초음파센서
적외선센서
온도센서
자이로센서

등으로 세상을 인식합니다.

즉

센서는 로봇의 감각기관입니다.

입력(Input)이란?

아두이노는

입력(Input)

↓

처리(Process)

↓

출력(Output)

순서로 동작합니다.

예)

버튼 누름

↓

아두이노가 읽음

↓

LED 켜기

센서의 종류

대표적인 센서는 아래와 같습니다.

센서	용도
버튼	사용자 입력
가변저항	아날로그 입력
조도센서	밝기 측정
온도센서	온도 측정
습도센서	습도 측정
초음파센서	거리 측정
적외선센서	장애물 감지
컬러센서	색상 인식
자이로센서	기울기 측정
IMU	자세 측정
압력센서	힘 측정
로드셀	무게 측정
홀센서	자석 감지
엔코더	회전량 측정
GPS	위치 측정
카메라	영상 인식
디지털 센서

출력이

0

또는

1

만 존재합니다.

예)

버튼

안눌림 → LOW

눌림 → HIGH
버튼 연결
5V

 │

버튼

 │

D2

 │

10KΩ

 │

GND

또는

INPUT_PULLUP

사용

버튼 예제
const int button = 2;

void setup()
{
    pinMode(button, INPUT_PULLUP);
    Serial.begin(9600);
}

void loop()
{
    if(digitalRead(button)==LOW)
    {
        Serial.println("Button");
    }
}
아날로그 센서

출력이

0~1023

값으로 변합니다.

예)

가변저항

0

↓

523

↓

1023
가변저항 연결
5V

│

가변저항

│

A0

│

GND
예제
int value;

void setup()
{
    Serial.begin(9600);
}

void loop()
{
    value=analogRead(A0);

    Serial.println(value);

    delay(100);
}
조도센서(LDR)

빛을 측정합니다.

어두움

↓

저항 증가

↓

값 변화

활용

자동조명
야간로봇
스마트조명
초음파센서 HC-SR04

가장 많이 사용하는 거리센서입니다.

핀

VCC

Trig

Echo

GND

원리

초음파 발사

↓

반사

↓

시간측정

↓

거리계산

공식

거리(cm)

=

시간 ×0.034/2
예제
const int trig=9;
const int echo=10;

void setup()
{
    pinMode(trig,OUTPUT);
    pinMode(echo,INPUT);

    Serial.begin(9600);
}

void loop()
{
    digitalWrite(trig,LOW);
    delayMicroseconds(2);

    digitalWrite(trig,HIGH);
    delayMicroseconds(10);

    digitalWrite(trig,LOW);

    long duration=pulseIn(echo,HIGH);

    float distance=duration*0.034/2;

    Serial.println(distance);

    delay(100);
}
적외선 장애물 센서

원리

적외선 발사

↓

물체 반사

↓

감지

활용

장애물 회피
라인트레이서
자동문
엔코더

모터의 회전량을 측정합니다.

활용

위치제어
속도제어
PID제어
IMU 센서

측정

X축
Y축
Z축

가속도

자이로

자기장

활용

드론
휴머노이드
균형로봇
자율주행
카메라

가장 중요한 센서입니다.

카메라 하나로

얼굴인식
객체인식
사람추적
AI학습

모두 가능합니다.

여러분이 앞으로 배우게 될

OpenCV
YOLO
MediaPipe
LeRobot

모두 카메라를 사용합니다.

여러 센서 동시에 사용

예)

초음파

+

카메라

+

IMU

+

엔코더

+

GPS

↓

AI가 판단

↓

모터 제어

↓

자율주행

실습 1 : 버튼으로 LED 켜기

준비물

Arduino UNO
버튼
LED
220Ω 저항
브레드보드

목표

버튼을 누르면 LED가 켜지고, 떼면 꺼지도록 구현합니다.
실습 2 : 가변저항으로 LED 밝기 조절

준비물

Arduino UNO
가변저항(10kΩ)
LED
220Ω 저항

목표

analogRead()로 읽은 값을 analogWrite()에 적용하여 LED 밝기를 부드럽게 조절합니다.
실습 3 : 초음파 센서로 거리 측정

준비물

Arduino UNO
HC-SR04 초음파 센서

목표

시리얼 모니터에 거리를 출력하고, 일정 거리 이내에서는 LED를 켜거나 부저를 울립니다.
실습 4 : 장애물 회피 자동차 기초

구성

초음파 센서
DC 모터
모터 드라이버(L298N 등)

동작 순서

거리 측정

↓

20cm 이하인가?

↓

예 → 정지 또는 회전

↓

아니오 → 전진
실습 5 : AI 로봇의 센서 융합 개념

여러 센서를 함께 사용하는 기본 흐름입니다.

카메라(YOLO)
        │
        ▼
객체 위치 인식
        │
        ├─────────────┐
        ▼             ▼
초음파 센서      엔코더
거리 확인        이동 거리 확인
        │             │
        └──────┬──────┘
               ▼
        아두이노/ROS2 판단
               ▼
          모터 제어

이 구조는 앞으로 배우게 될 OpenCV, YOLO, ROS 2, MoveIt, LeRobot 프로젝트의 기초가 됩니다.

이번 단계 핵심 정리
센서는 로봇의 감각기관이다.
디지털 센서는 ON/OFF 상태를 읽는다.
아날로그 센서는 연속적인 값을 읽는다.
초음파 센서는 거리를 측정한다.
IMU는 자세와 기울기를 측정한다.
엔코더는 모터의 회전량을 측정한다.
카메라는 AI 로봇의 핵심 입력장치이다.
여러 센서를 함께 사용하는 센서 융합(Sensor Fusion) 이 자율주행과 지능형 로봇의 핵심 기술이다.
다음 단계 (6단계)

Python 프로그래밍

다음 단계에서는 Python 문법부터 시작하여 아두이노와의 시리얼 통신, OpenCV와 AI 비전 프로젝트의 기반이 되는 프로그래밍을 체계적으로 학습합니다.
