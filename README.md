# Reaction Wheel

[단국대학교 MAZE](https://maze.co.kr) 2020년 로봇전시회 Reaction Wheel Bike 개발 내용

개발 기간 : 2019. 12 ~ 2020. 05

## 프로젝트 개요

Reaction Wheel Bike는 Reaction Wheel의 회전 관성을 이용해 로봇 스스로 균형을 잡고 서있을 수 있도록 하는 동시에, 카메라를 활용한 도로 자율주행 알고리즘에 의해 맵을 주행 할 수 있는 로봇입니다.
자율주행을 자동차에만 한정시키는 것이 아니라 이동체(Mobility)로의 확장을 고려해서 이륜 이동체에 카메라 기반의 LKS 및 신호 인식을 구현했습니다. 

## 프로젝트 결과

[개발완료 보고서 PDF]

[전시회 영상](https://www.youtube.com/watch?v=4kkZjeZvWrI)

## 팀원 구성

|팀원|맡은 역할|
|:---------:|:---:|
|김원석| 하드웨어 제작, 제어기 설계, 영상처리(정)|
|[이진호](https://github.com/StylishPanther)|MCU 개발환경 설정, 센서 데이터 파싱| 

## 프로젝트 목표    

1. 리액션 휠의 회전관성을 이용한 Inverted Pendulum의 구현

2. OpenCV 라이브러리를 사용한 영상처리로 자율주행 시스템 구축

3. 이륜차의 균형 유지와 주행의 자율화의 융합 가능성을 확인.

## 프로젝트 내용 
<br>
<p align="center"><img src="./Images/disturbancecontrol.gif" width="360px" ></p>  
<p align="center"> < 외부 충격 반응 속도 테스트 ></p>  

<p align="center"><img src="./Images/RemoteControlMovement.gif" width="360px"></p>  
<p align="center"> < 원격 조종을 이용한 주행 테스트></p>

<p align="center"><img src="./Images/CurvedWayDriving.gif" width="360px"></p>  
<p align="center"> < 자율 곡선 주행 테스트></p>

### 하드웨어 품목  

|Hardware Type|Model Name|Datasheet|  
|:---:|:---:|:---:|
|MCU|STM32F407VGT6|[PDF](./PDF/stm32f405_407.pdf)|
|Vision Processor|Raspberry-Pi-4|[PDF]((./PDF/Raspberry-Pi-4-Product-Brief.pdf))| 
|Camera|PI-Cam|[PDF](./PDF/ST-1KLA.pdf)|  
|Geomagnetic Sensor|EBIMU-9DOFV5|[PDF](./PDF/EBIMU-9DOFV5_rev11.pdf)|
|Encodered Motor|RA20GM_04_EN26, RB35GM_09SQ_EN|[PDF](./EN-21-146.pdf), [PDF](rb35gm_09SQ_en.jpg)|
|Servo Motor|DM_S2006MD|[PDF](./PDF/DM_S2006MD.png)|  
|Motor Drive|TB67H420FTG|[PDF](./PDF/TB67H420FTG_datasheet.pdf)|  
|Regulator|LM2576|[PDF](./PDF/LM2576_datasheet.pdf)|

### 개발환경 

|Tool Name|Description|  
|:---:|:---:|  
|Atolic True studio |STMicroelectronics IDE|

### SDK

|SDK|Description|  
|:---:|:---:|  
|HAL Library|STMicroelectronics에서 제공하는 HAL 라이브러리

 

 ### 소스코드 
<!-->
크게 라이브러리를 이용한 펌웨어 설정 부분과 슬레이브 기능 구현을 위한 어플리케이션 부분으로 나뉘며 
Build.bat을 이용해 makefile을 동작시키는 방식으로 .hex파일을 생성했다.

 `DSP280x_***.c` : GPIO ,PWM, ADC, SCI, SPI, Clock, Timer 등의 기본 기능을 설정하는 파일로 Datasheet 기반으로 설정되었다. 
 Linker Script의 Peripherals 메모리 정보를 DSP280x_GlobalVariableDefs.c를 통해 불러와 주소를 지정해준다.
- - -

 ###  motor.c
`motor_ISR` : 500 마이크로초 마다 실행되는 타이머 인터럽트. Kp, Kd 계수에 의해 결정된 PWM 값으로 모터를 동작시키고 센서에 의해 결정되는 포지션에 따라서 좌회전, 우회전을 판단한다.

`Straight_PID()` : 포지션 값을 이용해 모터에 인가되는 PWM을 결정한다.
- - -
### sensor.c

ADC를 이용해 센서 값을 받기 위한 함수들로 구성되어있다.


`sen_vari_init()` : 센서와 관련된 변수들을 초기화 하는 함수.

`sensor_checking()` : VFD를 이용해서 로봇이 현재 받고있는 센서 값 출력

`make_position()` : 6개의 센서값과 6개 센서에 대한 포지션 테이블을 설정해 센서 값에 따른 위치 값을 결정. 위치 값이 0이 되도록 모터를 제어한다.

`position_enable()` : 포지션 값이 외란에 대해 강건성을 가지고 연속적으로 움직일 수 있도록 현재 보고 있는 위치에서 다음 값의 변화량이 너무 클 경우 무시한다. 

`maxmin_set()` : 적외선 센서의 최대 값과 최소 값을 127등분 해서 Threshold로 설정한 값 이상의 센서 값이 들어오면 그 곳을 추종 대상으로 생각한다. Threshold의 비율은 실험적으로 알아낸다.

`Handle()` : make_position()에 의해 결정된 포지션 값을 좌측 모터와 우측 모터의 PWM 인가 비율로 변환한다.



- - -
### Etc

`Rom.c` : SPI 통신을 이용해 이전에 저장해 놓은 각 센서 별 MaxValue, minvalue를 읽거나 저장한다.

`search.c` : 플래그를 통해 모터 타이머 인터럽트를 동작시킨다.

`menu.c` : 종합 디버깅 함수로 현재 상태의 센서 값, 포지션 값, 주행 결정 등을 선택 할 수 있게 함수 포인터로 구성되어있다.

 `variable.h` : 전역 변수로 설정된 변수들을 모아 놓은 헤더. 

 `struct.h` : 전역 변수로 설정된 구조체 변수들을 모아 놓은 헤더.
