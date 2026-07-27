6단계. Python 프로그래밍 기초 (로봇 제어를 위한 필수 과정)

목표

Python을 이용하여 로봇을 제어할 수 있는 프로그래밍 능력을 기른다.

앞으로 배우게 될

OpenCV
YOLO
ROS2
MoveIt
LeRobot
AI Pick & Place

모두 Python을 사용합니다.

이번 단계에서 배우는 내용
1. Python이란?

2. Python 설치

3. 변수

4. 자료형

5. 입력과 출력

6. 조건문(if)

7. 반복문(for, while)

8. 함수

9. 리스트

10. 딕셔너리

11. 파일 저장

12. 모듈

13. 객체지향

14. 예외처리

15. 실습 프로젝트
1. Python이란?

Python은 가장 많이 사용하는 AI 언어입니다.

특징

배우기 쉽다.
코드가 짧다.
AI 라이브러리가 많다.
로봇 제어에 적합하다.

예)

print("안녕하세요")

실행하면

안녕하세요
2. 변수

변수는 데이터를 저장하는 공간입니다.

예)

name = "홍길동"
age = 20

print(name)
print(age)

결과

홍길동
20
3. 숫자 계산
a = 10
b = 3

print(a+b)
print(a-b)
print(a*b)
print(a/b)
4. 문자열
robot = "OpenManipulator"

print(robot)

print(robot + " X")
5. 입력받기
name = input("이름 입력 : ")

print("안녕하세요", name)
6. 조건문
temp = 30

if temp >= 30:
    print("덥습니다.")
else:
    print("괜찮습니다.")
7. 반복문

for문

for i in range(5):
    print(i)

결과

0
1
2
3
4

while문

count = 0

while count < 5:
    print(count)
    count += 1
8. 함수
def hello():

    print("안녕하세요")

hello()

매개변수

def add(a,b):

    return a+b

print(add(3,5))
9. 리스트
motors = [1,2,3,4]

print(motors[0])

print(motors[3])

반복

for motor in motors:

    print(motor)
10. 딕셔너리
robot = {

"name":"OMX",

"joint":5,

"gripper":True

}

print(robot["name"])
11. 파일 저장
f = open("test.txt","w")

f.write("Hello Robot")

f.close()
12. 모듈
import math

print(math.pi)

print(math.sqrt(16))
13. 객체지향
class Robot:

    def hello(self):

        print("Robot Start")

r = Robot()

r.hello()
14. 예외처리
try:

    num = 10/0

except:

    print("오류 발생")
15. 실습 프로젝트
실습 1 : LED 제어

Python → Arduino

Python

↓

Serial 통신

↓

Arduino

↓

LED ON/OFF
실습 2 : 스텝모터 제어
Python

↓

Serial

↓

Arduino

↓

Stepper Motor
실습 3 : 카메라 화면 보기
import cv2

cap = cv2.VideoCapture(0)

while True:

    ret, frame = cap.read()

    cv2.imshow("camera", frame)

    if cv2.waitKey(1)==27:
        break

cap.release()

cv2.destroyAllWindows()
실습 4 : 이미지 저장
import cv2

cap = cv2.VideoCapture(0)

ret, frame = cap.read()

cv2.imwrite("photo.jpg", frame)

cap.release()
이번 단계 최종 프로젝트

Python으로 다음 기능을 구현합니다.

콘솔 프로그램 작성
파일 저장
카메라 영상 보기
이미지 저장
Arduino와 시리얼 통신
LED 제어
스텝모터 제어
간단한 로봇 제어 프로그램 작성
학습 목표

6단계를 완료하면 다음을 할 수 있습니다.

✅ Python 문법 이해

✅ 함수 작성

✅ 클래스 작성

✅ OpenCV 사용 준비

✅ Arduino 연동

✅ ROS2 Python 노드 이해

✅ YOLO 프로그램 분석

✅ MoveIt Python API 사용 준비

다음 단계 예고 (7단계)

7단계는 OpenCV 영상처리입니다.

여기서는 다음 내용을 배우게 됩니다.

OpenCV 설치
이미지 처리
영상 처리
웹캠 제어
색상 검출
윤곽선 검출
얼굴 인식
객체 추적
ROI
필터
마우스 이벤트
OpenCV를 이용한 로봇 비전 기초

이 단계를 마치면 이후의 YOLO 객체 인식과 AI Pick &Place를 훨씬 쉽게 이해할 수 있습니다
