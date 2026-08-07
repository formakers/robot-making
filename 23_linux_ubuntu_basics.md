리눅스(Linux)와 우분투(Ubuntu)는 서로 경쟁하는 별개의 운영체제라기보다, 큰 범주와 그 안에 속하는 제품의 관계로 이해하면 쉽습니다.

Linux = 운영체제의 핵심인 Linux 커널 + 이를 기반으로 한 운영체제 계열

Ubuntu = Linux를 기반으로 만들어진 여러 배포판 중 하나

쉽게 말하면 Linux가 큰 가족이라면 Ubuntu는 그 가족의 한 구성원입니다.

1. Linux란?

Linux는 1991년 리누스 토르발스(Linus Torvalds)가 개발을 시작한 Linux 커널에서 출발했습니다.

엄밀히 말하면 Linux 자체는 운영체제 전체라기보다 커널(Kernel)을 가리킵니다.

컴퓨터 구조를 아주 단순화하면 다음과 같습니다.

사용자
 ↓
프로그램
Python / ROS 2 / Chrome / VS Code
 ↓
운영체제
 ↓
Linux Kernel
 ↓
하드웨어
CPU / RAM / USB / GPU / 네트워크 / 모터 컨트롤러

커널은 프로그램과 실제 하드웨어 사이에서 CPU, 메모리, 파일, USB 장치, 네트워크 등을 관리합니다.

예를 들어 터미널에서

ls /dev/ttyACM*

를 실행했을 때

/dev/ttyACM0
/dev/ttyACM1

같은 장치가 나타나는 것도 Linux의 장치 관리 체계와 관련되어 있습니다.

2. Linux 배포판이란?

Linux 커널만 가지고 일반 사용자가 바로 컴퓨터를 사용하기는 어렵습니다.

그래서 Linux 커널에 각종 프로그램, 패키지 관리 시스템, 데스크톱 환경, 설치 프로그램 등을 조합해서 완성된 운영체제 형태로 만든 것이 Linux 배포판(Distribution, Distro)입니다.

대표적으로 다음과 같은 것들이 있습니다.

Linux
│
├── Ubuntu
├── Debian
├── Fedora
├── Arch Linux
├── Linux Mint
├── Rocky Linux
└── Kali Linux

즉,

Linux = 기반
Ubuntu = Linux 기반 배포판

이라고 이해하면 됩니다.

3. Ubuntu란?

Ubuntu는 Linux 배포판 중 하나입니다.

Ubuntu는 Canonical이 개발·지원하며, Debian 계열을 기반으로 만들어졌습니다.

관계를 단순화하면

Linux Kernel
     ↓
   Debian
     ↓
   Ubuntu

정도로 이해할 수 있습니다.

Ubuntu는 일반 사용자가 Linux를 비교적 쉽게 설치하고 사용할 수 있도록 데스크톱 환경, 패키지 관리, 각종 기본 프로그램 등을 제공합니다.

4. Linux와 Ubuntu의 가장 중요한 차이
구분	Linux	Ubuntu
의미	커널 및 Linux 계열 전체를 가리키는 말	Linux 배포판 중 하나
범위	매우 넓음	Linux에 포함됨
개발 시작	Linus Torvalds	Canonical
종류	하나의 기반/생태계	특정 배포판
패키지 관리	배포판마다 다름	주로 APT
초보자 접근성	배포판에 따라 다름	비교적 쉬움
ROS 2/로봇 개발	많이 사용	특히 많이 사용

그래서 "Linux와 Ubuntu 중 어떤 것을 설치해야 하나?"라는 질문은 엄밀하게는 조금 이상합니다.

Ubuntu를 설치하면 이미 Linux를 사용하고 있기 때문입니다.

Ubuntu ⊂ Linux

라고 생각하면 됩니다.

5. 자동차로 비유하면

조금 단순화한 비유지만 이렇게 생각하면 이해하기 쉽습니다.

Linux
= 자동차의 기본 기술/플랫폼

Ubuntu
= 그 플랫폼을 이용해서 실제 사용하기 좋게 만든 특정 자동차 모델

그리고 다른 Linux 배포판들은 다른 모델에 해당합니다.

              Linux
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
    Ubuntu    Fedora    Arch
       │
       ↓
 Ubuntu 24.04
6. Ubuntu에서 자주 사용하는 명령어

Ubuntu를 사용하다 보면 다음과 같은 명령을 많이 보게 됩니다.

시스템 업데이트:

sudo apt update

프로그램 설치:

sudo apt install git

현재 Ubuntu 버전 확인:

lsb_release -a

Linux 커널 버전 확인:

uname -r

여기서도 Linux와 Ubuntu의 차이가 드러납니다.

lsb_release는 Ubuntu 배포판 버전을 확인하는 것이고,

uname -r

은 그 Ubuntu가 사용하고 있는 Linux 커널 버전을 확인하는 것입니다.

예를 들어 개념적으로는

Ubuntu 24.04 LTS
       │
       └── Linux Kernel 6.x

처럼 Ubuntu 안에서 Linux 커널이 동작합니다.

7. ROS 2와 로봇에서는 왜 Ubuntu를 많이 사용할까?

로봇 개발에서는 Ubuntu가 특히 많이 등장합니다.

예를 들어 다음과 같은 환경입니다.

PC
 ↓
Ubuntu
 ↓
Linux Kernel
 ↓
ROS 2
 ↓
MoveIt 2
 ↓
ROBOTIS / DYNAMIXEL / OpenManipulator
 ↓
실제 로봇

ROS 2, MoveIt 2, Gazebo 같은 로봇 소프트웨어의 Linux 지원이 강하고, Ubuntu는 관련 설치 문서와 패키지가 풍부하기 때문에 로봇 개발 환경으로 널리 쓰입니다.

따라서 ROS 2를 Ubuntu에서 사용하고 있다면 구조를 다음처럼 생각하면 좋습니다.

┌────────────────────────────┐
│ ROS 2 / MoveIt / Python    │
├────────────────────────────┤
│ Ubuntu                     │
├────────────────────────────┤
│ Linux Kernel               │
├────────────────────────────┤
│ PC Hardware                │
│ USB / CPU / RAM / GPU      │
└────────────────────────────┘
한 문장으로 정리

Linux는 Ubuntu의 기반이며, Ubuntu는 Linux를 일반 사용자와 개발자가 편리하게 사용할 수 있도록 구성한 Linux 배포판 중 하나입니다.

그래서 "나는 Ubuntu를 사용한다" = "나는 Linux 계열 운영체제를 사용한다"라고 보면 거의 맞습니다.
