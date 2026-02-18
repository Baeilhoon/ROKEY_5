# 🤖 Project 2: SLAM 기반 자율주행 로봇 시스템

> **MCU 펌웨어 상태 관리 & 제어 흐름 안정성 검증**  
> SLAM-based Mobile Robot with Gripper Control  

## 📋 개요

SLAM 기반 자율주행 로봇 시스템에서 **그리퍼 동작을 담당하는 MCU 펌웨어**를  
직접 설계·구현한 프로젝트입니다.

`PC → TurtleBot3 → Arduino` 로 이어지는 **제어 흐름**에서 그리퍼 서보 모터의  
상태 관리와 예외 처리를 중점적으로 해결했습니다.

---

## 🎯 핵심 성과

| 성과 | 설명 |
|------|------|
| **상태 기반 펌웨어** | State-driven 구조로 재초기화 버그 제거 |
| **EEPROM 활용** | 전원 재시작 후에도 마지막 상태 복원 |
| **안정적 상태 전환** | `OPEN / MOVING / CLOSED / ERROR` 상태머신 |
| **명시적 제어 흐름** | 명확한 명령-응답 프로토콜 구현 |

---

## 🔧 내가 맡은 것

### 1️⃣ Arduino 그리퍼 펌웨어 설계
- 서보 모터 제어 (`OPEN / CLOSE / POSITION_CONTROL`)
- 상태 기반 동작 구조 (State-Driven Architecture)
- 예외 상황 감지 및 자동 복구

### 2️⃣ PC → TurtleBot3 → Arduino 제어 흐름 정의
- 상위 계층의 high-level 명령을 하위 하드웨어 명령으로 변환
- 각 계층 간 동기화 메커니즘
- 통신 프로토콜 정의

### 3️⃣ 상태 동기화 로직 구현
```
[ OPEN ]
   ↓ 집기 명령 수신
[ MOVING ]
   ↓ 목표 각도 도달
[ CLOSED ]
   ↓ 해제 명령 수신
[ MOVING ]
   ↓ 초기 위치 복귀
[ OPEN ]
```

### 4️⃣ EEPROM 기반 상태 보존
- 마지막 서보 위치 저장
- 전원 재시작 시 자동 복원
- 스택 오버플로우 방지

---

## 🐛 발생한 문제 & 해결

### 1️⃣ 상태 전환 중 의도치 않게 `reset()` 재호출

**문제**: 작업 중 그리퍼가 초기 위치로 복귀함  
**원인**: 초기화 로직이 `loop()` 함수에 위치하여 매 루프마다 호출

**해결**:
```cpp
// 잘못된 구조 (Before)
void setup() {
  // ...
}

void loop() {
  initialize_gripper();  // ❌ 매 루프 호출!
  execute_command();
}

// 올바른 구조 (After)
volatile bool initialized = false;

void setup() {
  initialize_gripper();
  initialized = true;
}

void loop() {
  if (initialized) {
    execute_command();
  }
}
```

### 2️⃣ 그리퍼가 작업 중 초기 위치로 복귀

**문제**: 상태 전환 시 마다 초기화 함수 재호출  
**원인**: 상태 플래그 확인 없이 무조건 함수 호출

**해결**: State-Driven 아키텍처 도입
```cpp
enum GripperState {
  OPEN = 0,
  MOVING = 1,
  CLOSED = 2,
  ERROR = 3
};

GripperState current_state = OPEN;

void update_state(GripperState new_state) {
  // 명시적 상태 전환만 처리
  if (is_valid_transition(current_state, new_state)) {
    current_state = new_state;
    // 상태별 처리만 수행
  } else {
    set_error_state();
  }
}
```

### 3️⃣ 전원 재시작 시 이전 상태 유실

**문제**: 로봇 리부팅 후 그리퍼 위치 미파악  
**원인**: 상태가 RAM에만 저장되어 전원 차단 시 초기화

**해결**: EEPROM 활용한 상태 보존
```cpp
// EEPROM 주소 정의
#define EEPROM_STATE_ADDR 0
#define EEPROM_POSITION_ADDR 1

void save_state_to_eeprom(GripperState state, uint8_t position) {
  EEPROM.write(EEPROM_STATE_ADDR, state);
  EEPROM.write(EEPROM_POSITION_ADDR, position);
}

GripperState load_state_from_eeprom() {
  return (GripperState)EEPROM.read(EEPROM_STATE_ADDR);
}

void setup() {
  // 부팅 시 이전 상태 복원
  GripperState last_state = load_state_from_eeprom();
  uint8_t last_position = EEPROM.read(EEPROM_POSITION_ADDR);
  
  servo.write(last_position);  // 마지막 위치로 이동
  current_state = last_state;
}
```

---

## 🛠️ 기술 스택

```
언어:         C++ (Arduino)  ·  Python  ·  C++17
프레임워크:   ROS2 Humble
보드:         TurtleBot3 Burger (Raspberry Pi, Arduino)
센서/하드웨어: Servo Motor  ·  EEPROM  ·  Arduino Nano
로봇:         TurtleBot3
OS:           Ubuntu 22.04 LTS  ·  Raspberry Pi OS
알고리즘:     SLAM (Simultaneous Localization and Mapping)
```

---

## 📊 아키텍처

```
ROS2 Navigation Stack (PC)
  ↓ (TurtleBot3 제어 명령)
TurtleBot3 (Raspberry Pi)
  ↓ (USB 직렬 통신)
Arduino Nano
  ├── Servo Motor PWM Control
  ├── State Management (RAM + EEPROM)
  └── Command Processing
  
상태 복원 흐름:
[전원 차단]
    ↓
[전원 재시작]
    ↓
[EEPROM에서 마지막 상태 로드]
    ↓
[서보 모터 이전 위치로 이동]
```

---

## 📝 통신 프로토콜

### 명령 형식
```
PC → TurtleBot3 → Arduino

명령: [CMD][PARAM1][PARAM2]

CMD:
  0x01: OPEN (매개변수 없음)
  0x02: CLOSE (매개변수 없음)
  0x03: MOVE_TO_POSITION (PARAM1: 각도)
  0x04: GET_STATE (상태 요청)

응답: [STATE][POSITION][ERROR_CODE]
```

---

## 🎓 배운 것

1. **임베디드 시스템 상태 관리의 중요성**
   - 상태 머신이 없으면 예상 못한 동작 발생
   - State-Driven 설계의 필수성

2. **하드웨어 계층 추상화**
   - 저수준 센서 제어와 고수준 명령 간 분리
   - 계층 간 명확한 인터페이스 정의

3. **비휘발성 메모리 활용**
   - RAM만으로는 영구 상태 유지 불가
   - EEPROM/Flash의 적절한 활용으로 robustness 확보

4. **디버깅 복잡도 관리**
   - 상태 로깅, 타임스탬프 기록으로 추후 분석 용이
   - 하드웨어 동작 검증과 소프트웨어 로직 검증 분리

---

## 📁 주요 파일 구조

```
project2_slam_robot/
├── firmware/
│   ├── gripper_control.ino
│   ├── state_machine.h
│   └── eeprom_manager.h
├── ros_nodes/
│   └── gripper_bridge.py
└── config/
    └── parameters.yaml
```

---

## 📝 참고 자료

- **TurtleBot3 공식 문서**: https://emanual.robotis.com/docs/en/platform/turtlebot3/
- **ROS2 Navigation**: https://navigation.ros.org/
- **Arduino EEPROM Guide**: https://www.arduino.cc/reference/en/libraries/eeprom/
