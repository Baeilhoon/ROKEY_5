# 🏭 Project 1: 컨베이어벨트 자동화 공정 제어 시스템

> **ROS2 기반 로봇팔 연동 자동화 시스템**  
> Conveyor Belt Automation with Robot Arm Integration  

## 📋 개요

컨베이어벨트를 중심으로 **물체 감지 → 이송 → 정지 → 재개** 공정 흐름을  
하나의 상태 기반 제어 구조로 통합한 시스템입니다.

ROS2 기반 로봇팔 시스템과 연동되어 공정 전체를 단일 상태 머신으로 제어하며,  
**상태 관리와 동기화 메커니즘**을 중심으로 설계했습니다.

---

## 🎯 핵심 성과

| 성과 | 설명 |
|------|------|
| **상태 기반 제어** | `IDLE / MOVING / WAITING / EMERGENCY` 4단계 상태머신 구현 |
| **IR 센서 노이즈 제거** | 디바운싱 로직으로 이벤트 중복 제거 |
| **동기화 안정화** | 우선순위 ACK 구조로 제어 명령 충돌 해결 |
| **실시간 모니터링** | Firebase 기반 공정 상태 가시화 |

---

## 🔧 내가 맡은 것

### 1️⃣ Arduino 펌웨어 개발
- 컨베이어벨트 스텝 모터 제어 (`START / STOP / RESUME / ESTOP`)
- IR 센서 신호 처리 및 이벤트 생성
- 상태 플래그 관리 및 예외 처리

### 2️⃣ ROS2 브릿지 노드 구현
- Arduino 신호 ↔ ROS2 토픽 변환 노드
- `conveyor_state` 토픽과 `conveyor_command` 서비스 정의
- ROS2-Arduino 간 직렬 통신 레이어 구현

### 3️⃣ 상태 머신 설계
```
[ IDLE ] 
   ↓ 물체 감지
[ MOVING ]
   ↓ 목표 위치 도달
[ WAITING ] 
   ↓ 로봇팔 작업 완료
[ IDLE ]
```

### 4️⃣ 동기화 메커니즘
- 로봇팔과 컨베이어 간 우선순위 ACK 구조
- 컨베이어 상태를 로봇팔 제어 유닛으로 발행
- 요청 중복 방지 플래그 관리

---

## 🐛 발생한 문제 & 해결

### 1️⃣ IR 센서 노이즈로 인한 이벤트 중복

**문제**: 동일 물체에 감지 이벤트가 여러 번 발생  
**원인**: IR 센서의 아날로그 신호 노이즈

**해결**:
```cpp
// 디바운싱 로직
if (currentState != lastState) {
  delayMicroseconds(DEBOUNCE_TIME);
  if (analogRead(SENSOR_PIN) > THRESHOLD) {
    publishEvent("OBJECT_DETECTED");
    lastState = currentState;
  }
}
```

### 2️⃣ 컨베이어-로봇팔 동작 겹침으로 동기화 깨짐

**문제**: 컨베이어 이송 중 로봇팔이 물체 잡으려 하거나, 반대의 경우 발생  
**원인**: 제어 명령 간 우선순위 없음

**해결**:
- `IDLE / MOVING / WAITING / EMERGENCY` 4단계 상태 정의
- 상태별 허용 작업 명시
- 우선순위 ACK 구조: 로봇팔 요청 → 컨베이어 상태 확인 → 승인/거부

```cpp
// Arduino 상태 확인
if (state == WAITING) {
  // 로봇팔 요청 허용
  digitalWrite(ACK_PIN, HIGH);
} else {
  // 요청 거부
  digitalWrite(ACK_PIN, LOW);
}
```

### 3️⃣ ROS2-Arduino 통신 지연으로 명령 중복 실행

**문제**: 한 번의 명령이 여러 번 실행됨  
**원인**: 비동기 타이밍 문제, 컨베이어 노드와 다른 로봇팔 노드 간 메시지 충돌

**해결**:
- 컨베이어 전용 ROS2 브릿지 노드 분리
- 명령 큐(Queue) 구조로 순차 처리
- 명령 완료 확인 후 다음 명령 실행

```cpp
// ROS2 콜백에서 즉시 실행 대신 큐에 추가
void command_callback(const Twist& msg) {
  command_queue.push(msg);
}

// 별도 스레드에서 순차 처리
while (!command_queue.empty()) {
  Command cmd = command_queue.front();
  execute_command(cmd);  // 완료 대기
  command_queue.pop();
}
```

---

## 🛠️ 기술 스택

```
언어:         C++ (Arduino)  ·  Python  ·  C++17
프레임워크:   ROS2 Humble
센서/하드웨어: IR Sensor  ·  Step Motor  ·  Arduino UNO/MEGA
OS:           Ubuntu 22.04 LTS
협업 도구:    Flask (모니터링)  ·  Firebase (데이터 저장)
```

---

## 📊 아키텍처

```
PC (ROS2 Master)
  ↓
[ROS2 Bridge Node]
  ↓ (직렬 통신)
Arduino
  ├── IR Sensor Input
  ├── Step Motor Control
  └── State Management
  
로봇팔 시스템 (ROS2)
  ↓ (conveyor_state 토픽)
[ROS2 Bridge Node]
  ✓ 우선순위 검증
  ✓ ACK 응답
```

---

## 🎓 배운 것

1. **상태 머신의 중요성**
   - 단순 제어보다 상태 기반 설계가 복잡도 관리에 효과적
   
2. **임베디드 ↔ 고수준 시스템 간 인터페이스 설계**
   - 토픽/서비스로 추상화하여 확장성 확보
   
3. **디버깅과 안정성**
   - 노이즈 처리, 타이밍 문제는 단순 로직 개선이 아닌 아키텍처 설계에서 해결
   
4. **팀 협업에서의 명확한 인터페이스 정의**
   - 로봇팔 팀과의 연동을 위해 명확한 토픽/서비스 스펙 필수

---

## 📝 참고 자료

- **ROS2 공식 문서**: https://docs.ros.org/en/humble/
- **Arduino Reference**: https://www.arduino.cc/reference/
- **프로젝트 GitHub**: https://github.com/taesla/doosan_rokey_collabo1
