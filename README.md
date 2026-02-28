# 사용자 추적 기반 지향성 음향 시스템
## 초음파 파라메트릭 스피커와 AI 객체 추적을 활용한 1:1 개인화 청각 환경 구현

---

## 프로젝트 개요 (Background)

기존 스피커는 소리를 사방으로 확산시키는 구조로 설계되어 있어,  
특정 사용자에게만 음성을 전달하기 어렵고 주변에 불필요한 소음이 발생하는 문제가 있습니다.

본 프로젝트는 **초음파 기반 지향성 음향 기술(Parametric Speaker)** 과  
**AI 기반 객체 추적 시스템(HuskyLens)**,  
그리고 **2축 병렬 PID 제어 알고리즘**을 결합하여

> 특정 사용자에게만 소리를 전달하고  
> 사용자의 움직임을 실시간으로 추적하며  
> 스피커 방향을 자동으로 제어하는  
> 1:1 개인화 청각 환경 시스템을 구현하는 것을 목표로 합니다.

---

## 주요 기능

- 40kHz 초음파 기반 지향성 음향 송출
- HuskyLens 기반 실시간 얼굴 추적
- X/Y 2축 서보모터 제어
- 병렬 PID 제어 구조 적용
- Deadband 기반 진동 억제
- 거리 기반 가중치 제어 (Gain Scheduling)
- Lost Mode 상태 머신 설계

---

## 시스템 구성 (System Overview)

### 전체 신호 흐름

HuskyLens (AI Tracking)  
      ↓ I2C  
MCU (PID Controller)  
      ↓ PWM  
Dual Servo Motor (X/Y)  
      ↓  
Parametric Speaker (40kHz Array)

---

### 동작 설명

- 카메라 모듈(HuskyLens)이 사용자 위치를 실시간 추적
- MCU에서 위치 오차 계산 및 PID 제어 수행
- 서보모터를 통해 스피커 방향 자동 조정
- 초음파 자가복조(Self-Demodulation)를 통해 가청음 생성

---

## 제어 흐름도 (Control Flow Diagram)

시스템은 전원 인가 후 초기화를 수행하고,  
객체 인식 여부에 따라 Tracking Mode와 Lost Mode로 분기하는  
상태 머신(State Machine) 구조로 설계되었습니다.

![Control Flow](assets/control_flow.png)

### 동작 과정

1. 시스템 전원 ON → 마이크로프로세서 초기화
2. 서보모터 기준 각도(Center) 설정
3. 객체 인식 여부 판단
   - 인식 성공 → Tracking Mode 진입
   - 인식 실패 → Lost Mode 진입
4. Tracking Mode
   - 위치 오차 계산
   - PID 제어 수행
   - PWM 신호 생성
   - 서보모터 구동
   - 루프 반복
5. Lost Mode
   - 일정 시간 동안 마지막 위치 유지
   - 임계 시간 초과 시 안전 복귀

---

## 2축 PID 제어 블록도 (Dual-Axis PID Block Diagram)

![PID Block Diagram](assets/pid_block.png)

### 제어 구조

입력:
- 목표 위치 (X_ref, Y_ref)
- 측정 위치 (X_meas, Y_meas)

오차 계산:


error = reference - measurement


PID 제어식:


u(t) = Kp·e(t) + Ki∫e(t)dt + Kd·de(t)/dt


출력:
- 출력 제한 로직 (Limiter)
- PWM 변환
- 서보모터 각도 제어 (θ_x, θ_y)

HuskyLens에서 측정된 좌표는 실시간으로 피드백되어  
폐루프 제어(Closed-loop control)를 형성합니다.

---

## 지향성 음향 원리 (Parametric Speaker Principle)

40kHz 초음파 캐리어를 활용하여 공기 중 비선형 상호작용을 유도합니다.

두 초음파가 중첩되면 다음과 같은 성분이 생성됩니다:


cos(a)·cos(b) = 1/2 [cos(a+b) + cos(a-b)]


예시:
- 40kHz + 41kHz → 81kHz (합 주파수)
- 41kHz - 40kHz → 1kHz (차 주파수 → 가청음)

이를 통해 특정 방향으로만 소리가 전달되는 음향 빔(Beam)을 형성합니다.

---

## 객체 추적 시스템 (AI Tracking)

- 모듈: HuskyLens
- 해상도: 320 × 240
- 출력값: (X_meas, Y_meas), width, height, ID
- 통신 방식: I2C

CNN 기반 객체 인식을 통해 얼굴 중심 좌표를 추출하고  
이를 제어 입력으로 사용합니다.

---

## 하드웨어 구성

- 40kHz 초음파 트랜스듀서 × 50
- Arduino 기반 MCU
- HuskyLens AI Vision Sensor
- 서보모터 × 2 (X/Y)
- 초음파 구동 회로 (오실레이터 + 필터 + 드라이버 IC)

---

## 저장소 구조 (Repository Structure)


```bash

├── README.md
├── assets/
│   ├── control_flow.png
│   └── pid_block.png
├── code/
│   ├── x_axis_tracking/
│   └── xy_axis_pid_tracking/
└── docs/
    └── Poster.png
    └── presentation meterial
```

---

## 시연 영상

(여기에 유튜브 링크 추가)

---

## 기대 효과 및 활용 분야

- 공공장소 소음 최소화
- 박물관/전시장 맞춤형 음성 안내
- 병원·복지시설 개인 알림 시스템
- 스마트홈 개인 음성 전달
- 접근성 보조 시스템 확장 가능

---

## 향후 개선 방향

- 밀폐 공간 내 반사 문제 개선
- 흡음재 및 차폐 구조 설계
- 제어 정밀도 향상
- 초음파 배열 최적화

---

## 프로젝트 정보

인천대학교 전기공학과  
2025-2학기 캡스톤 디자인  

팀장 : 이기현
