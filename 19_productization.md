AI 로봇 제작 마스터 클래스
제19단계 : 로봇 제품화 (Product Development)

지금까지의 과정에서는 동작하는 로봇을 만들었다면, 19단계는 판매할 수 있는 로봇으로 발전시키는 단계입니다.

많은 사람들이 로봇을 만들 수는 있지만 제품으로 만드는 방법은 잘 모릅니다.

제품화에서는

"실험용 로봇" → "신뢰성 있는 제품"

으로 바꾸는 과정입니다.

19단계 목표

다음과 같은 수준까지 완성합니다.

YOLO
   ↓
ROS2
   ↓
MoveIt
   ↓
AI
   ↓
실제 로봇
   ↓
PCB
   ↓
외관
   ↓
안전성
   ↓
제품 테스트
   ↓
판매 가능한 로봇
전체 과정

19단계는 크게 15개의 과정으로 나눌 수 있습니다.

1. 제품 기획
2. 요구사항 작성
3. 하드웨어 설계
4. PCB 제작
5. 전원 설계
6. 케이스 제작
7. 소프트웨어 구조
8. GUI 제작
9. 업데이트 기능
10. 오류 처리
11. 안전기능
12. 신뢰성 시험
13. 양산 준비
14. 인증 준비
15. 제품 완성
1. 제품 기획

가장 먼저 해야 하는 일입니다.

예를 들어

AI Pick & Place Robot

기능

컵 인식
물체 집기
자동 분류
컨베이어 연동

사용자

학교
연구소
기업

가격

300만원
500만원
1000만원
2. 요구사항 작성

제품은 문서부터 시작됩니다.

예

작업반경

500mm

반복정밀도

±0.2mm

최대속도

300mm/s

적재하중

500g

전원

24V
3. 하드웨어 설계

기존

Arduino
USB
브레드보드

제품

PCB

↓

MCU

↓

Motor Driver

↓

Encoder

↓

Sensor

↓

Power

↓

CAN

↓

Ethernet
4. PCB 제작

브레드보드는 사용하지 않습니다.

제품은

PCB

↓

회로설계

↓

Gerber

↓

PCB 제작

↓

부품실장

↓

테스트

사용 프로그램

KiCad
Altium
EasyEDA
PCB 예
MCU

STM32

↓

Motor Driver

↓

Encoder

↓

Power

↓

USB

↓

CAN

↓

IO
5. 전원 설계

제품에서 가장 중요합니다.

예

220V

↓

SMPS

24V

↓

DC/DC

12V

↓

DC/DC

5V

↓

3.3V
6. 케이스 제작

초기

3D 프린터

↓

제품

알루미늄

+

판금

+

사출

+

가공
좋은 제품의 조건
튼튼하다

예쁘다

가볍다

조립이 쉽다

수리하기 쉽다
7. 소프트웨어 구조

프로그램도 제품 구조로 변경합니다.

예

Robot App

↓

Motion

↓

Vision

↓

Planning

↓

Driver

↓

Hardware
ROS2 패키지 구성
robot_driver

robot_msgs

robot_description

robot_moveit

robot_navigation

robot_bringup

robot_gui
8. GUI 제작

사용자는 터미널을 사용하지 않습니다.

GUI 예

Robot Status

Joint

Camera

Start

Stop

Emergency

Home

Speed

Log

제작

Qt
PySide6
Electron(Web 기반)
9. 자동 업데이트

제품은 업데이트가 가능합니다.

Version 확인

↓

다운로드

↓

설치

↓

재부팅
10. 로그 시스템

모든 오류를 저장합니다.

시간

↓

오류내용

↓

모터상태

↓

센서

↓

카메라

예

2026-07-20

Joint2 Over Current

Motor Stop
11. 안전기능

필수입니다.

E-Stop

비상정지 버튼

Torque Limit

과부하 방지

Collision Detection

충돌 감지

Speed Limit

속도 제한

Soft Limit

관절 제한

Watchdog

프로그램 멈춤 감지

12. 신뢰성 시험

제품은 오래 동작해야 합니다.

예

24시간

72시간

1주일

1개월


시험 항목

발열
진동
충격
반복동작
전원 OFF/ON
USB 제거
LAN 제거
13. 양산 준비

부품을 정리합니다.

BOM

모터

감속기

PCB

센서

볼트

베어링

케이블
BOM 예
품목	수량
STM32	1
BLDC Driver	6
Encoder	6
Power	1
Bearing	12
Bolt	80
14. 인증 준비

제품을 판매하려면 인증이 필요합니다.

대표적으로 다음과 같은 인증을 검토해야 합니다(제품과 판매 국가에 따라 달라짐).

KC 인증(대한민국)
전자파 적합성(EMC)
전기안전 관련 인증
무선 기능이 있는 경우 무선 관련 인증
15. 제품 완성

최종 제품은 다음과 같은 형태가 됩니다.

제품 박스

↓

전원 연결

↓

GUI 실행

↓

Home

↓

AI 실행

↓

자동 작업

↓

로그 저장

↓

안전 종료
실제 산업용 AI 로봇 구조
Camera
    │
    ▼
YOLO 객체 인식
    │
    ▼
Depth Camera
    │
    ▼
3D 좌표 계산
    │
    ▼
MoveIt 2 경로 계획
    │
    ▼
ROS 2 Control
    │
    ▼
Servo Driver
    │
    ▼
Robot Arm
    │
    ▼
Gripper
포메이커 프로젝트에 적용하기

현재까지 진행한 내용을 기준으로 하면 다음과 같은 제품화 로드맵을 추천합니다.

OMX 기반 로봇팔을 표준 하드웨어로 확정
ROS 2 Jazzy + MoveIt 2 + YOLO + Depth Camera를 기본 소프트웨어 플랫폼으로 통합
PySide6 GUI를 제작하여 터미널 없이 로봇을 제어
AI Pick & Place 기능을 안정화하고 반복 테스트
로봇 본체를 알루미늄 프레임과 전용 외장으로 개선
제어기를 전용 PCB로 통합하고 배선을 단순화
사용자 매뉴얼과 설치 프로그램을 작성하여 누구나 설치 가능한 제품으로 완성
이번 단계 실습 프로젝트
프로젝트 1
PySide6 기반 로봇 제어 GUI 만들기
프로젝트 2
ROS 2 Bringup 자동 실행 시스템 구축
프로젝트 3
AI Pick & Place를 장시간 반복 테스트하여 안정성 검증
프로젝트 4
전용 제어 PCB 설계 및 제작
프로젝트 5
제품 외관(케이스) 설계 및 조립
학습 체크리스트
✅ 제품 요구사항 정의
✅ 하드웨어/PCB 설계 이해
✅ 전원 시스템 설계
✅ GUI 개발
✅ 로그 및 오류 처리
✅ 안전 기능 구현
✅ 신뢰성 시험 수행
✅ BOM 작성
✅ 인증 준비
✅ 판매 가능한 AI 로봇 제품 완성

이 19단계를 마치면 단순히 "동작하는 로봇"이 아니라 실제 고객이 사용할 수 있는 수준의 AI 로봇 제품을 설계하고 제작하는 역량을 갖추게 됩니다. 다음 20단계(창업 및 양산)에서는 사업 계획, 생산 공정 구축, 원가 계산, 마케팅, 판매 및 A/S 체계를 포함한 로봇 기업 운영 전반을 다루게 됩니다.
