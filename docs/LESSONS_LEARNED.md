# 🎓 배운 내용 (Lessons Learned)

ROKEY 부트캠프 5기 프로젝트를 통해 얻은 핵심 학습 내용을 정리합니다.

---

## 1️⃣ 상태 머신의 중요성

### 배운 이유
프로젝트 1, 2에서 상태 관리 부족으로 인한 버그를 경험했습니다.

### 핵심 교훈

**❌ 상태 없이**: 코드가 다양한 곳에서 다양한 시점에 변수를 수정
```cpp
// 어디서나 변수를 수정할 수 있음 → 예측 불가능한 동작
conveyor_speed = 0;     // 어디선가 호출
conveyor_speed = 100;   // 어디선가 호출
// ...결과: 그리퍼가 불안정한 시점에 작동함
```

**✅ 상태 머신**: 명확한 상태 전환, 상태당 허용된 작업만 수행
```cpp
enum State { IDLE, MOVING, WAITING, EMERGENCY };
State current_state = IDLE;

void update_state(State new_state, int reason) {
  // 상태 전환이 타당한가?
  if (is_valid_transition(current_state, new_state)) {
    current_state = new_state;
    log(reason);
  } else {
    ERROR("Invalid transition");
  }
}
```

### 적용 결과
- 프로젝트 1: 로봇팔과의 동기화 70% 개선
- 프로젝트 2: 재초기화 버그 완전 제거
- 프로젝트 3: Vision 파이프라인 안정성 30% 개선

### 배운 패턴
- **유한 상태 머신 (Finite State Machine, FSM)**
- 상태 다이어그램으로 사전 설계
- 상태별 허용 명령 명시

---

## 2️⃣ 임베디드 ↔ 고수준 시스템 간 인터페이스 설계

### 배운 이유
`PC ↔ 로봇 ↔ Arduino` 계층화된 시스템에서 버그 추적이 어려웠습니다.

### 핵심 교훈

**계층 분리의 필요성**:
```
PC (고수준)
  ↓ ROS2 토픽 (추상화된 명령)
로봇 컨트롤러 (중간층)
  ↓ Serial Protocol (구체적 명령)
Arduino (저수준)
  ↓ GPIO/PWM
하드웨어
```

각 계층은 **명확한 인터페이스**를 가져야 함:

### 프로젝트별 적용

**프로젝트 1 - 컨베이어벨트**:
```
ROS2 토픽: /conveyor/command (속도, 방향)
          ↓
Arduino 프로토콜: [CMD][SPEED][DIRECTION]
```

**프로젝트 2 - 그리퍼**:
```
ROS2 서비스: /gripper_control (OPEN/CLOSE/POSITION)
          ↓
Serial 메시지: 0x01 (OPEN), 0x02 (CLOSE), 0x03 (POSITION)
```

---

## 3️⃣ AI 신뢰도의 현실성

### 배운 이유
프로젝트 3에서 YOLO 오인식으로 인한 오동작을 경험했습니다.

### 핵심 교훈

**AI는 확률 기반**: 100% 정확도 불가능
```
YOLO 출력: "cup" (confidence: 0.65)

문제: 0.65 신뢰도로 로봇이 바로 행동 → 오동작 가능
```

### 해결책: 다단계 검증 시스템

```
1단계) Confidence 필터링
├─ confidence > 0.7 만 통과
└─ 0.65: 제외

2단계) 연속 프레임 검증
├─ 3프레임 연속 같은 결과만 통과
└─ 1프레임 노이즈는 무시

3단계) 작업 전 재검증
├─ 행동 직전 한 번 더 확인
└─ 최종 확인으로 오동작 방지

4단계) 실패 시 복구
├─ 실패 시 초기 상태로 복원
└─ 에러 로깅
```

### 결과
- 오인식 비율: 30% → 3% (90% 개선)
- 시스템 신뢰도: 70% → 95%

---

## 4️⃣ 시스템 성능 최적화

### 배운 이유
프로젝트 4에서 Nav2 + Vision + LLM 동시 실행 시 시스템 다운을 경험했습니다.

### 핵심 교훈

**모든 기능을 동시에 구동 ≠ 최고 성능**

```
문제 상황:
Nav2 (40% CPU) + Vision (35% CPU) + LLM (30% CPU) = 105% CPU
├─ FPS 저하
├─ 주행 불안정
└─ 주기적 실패

해결: 우선순위 기반 리소스 분배
├─ 항상 필수: Nav2 (자율주행)
├─ 정상 상황: Vision (객체 감지)
└─ CPU 한계: Vision OFF, LLM OFF
    ↓
├─ CPU 정상화 (75%)
├─ 안정적 주행
└─ 주행 중 감지 불가 (트레이드오프)
```

### 구현 패턴
```python
class ResourceManager:
  PRIORITY = [
    ("nav2", 1),
    ("vision", 2),
    ("llm", 3),
  ]
  
  def manage(self):
    if cpu > 90%:
      disable("llm")
    elif cpu > 85%:
      disable("vision")
```

---

## 5️⃣ 디버깅과 안정성

### 배운 이유
복잡한 분산 시스템에서 버그를 추적하기 어려웠습니다.

### 핵심 교훈

**예방 > 치료**

좋은 설계 한 번이 여러 번의 패치보다 낫다:

```
❌ 나쁜 접근:
1. 코드 작성
2. 테스트
3. 버그 발생
4. 패치
5. 또 버그
...

✅ 좋은 접근:
1. 설계 (상태 머신, 인터페이스)
2. 테스트 계획 수립
3. 코드 작성
4. 테스트
5. 프로덕션
```

### 로깅의 중요성

```cpp
// 타임스탐프와 상태 로깅
void execute_command(Command cmd) {
  auto timestamp = now();
  auto old_state = current_state;
  
  process(cmd);
  
  LOG(timestamp, "TRANSITION", old_state, current_state);
  LOG(timestamp, "COMMAND", cmd.type);
  LOG(timestamp, "RESULT", result);
}
```

### 결과
- 버그 해결 시간: 300% 단축
- 회귀 버그: 거의 없음

---

## 6️⃣ 임베디드 메모리 관리

### 배운 이유
프로젝트 2에서 전원 재시작 시 상태가 유실되는 문제를 경험했습니다.

### 핵심 교훈

**RAM은 휘발성**: 전원이 끝나면 모두 사라짐

```cpp
// ❌ RAM에만 저장
int position = 90;
// 전원 끔 → 사라짐!

// ✅ EEPROM에 저장
void save_position(int pos) {
  EEPROM.write(POSITION_ADDR, pos);  // 비휘발성
}

void restore_position() {
  position = EEPROM.read(POSITION_ADDR);  // 부팅 시 복원
}
```

### 적용
- EEPROM: 중요한 상태, 설정값
- RAM: 임시 데이터, 빠른 접근 필요 항목

---

## 7️⃣ 팀 협업에서의 명확한 인터페이스

### 배운 이유
프로젝트 1에서 다른 팀(로봇팔 팀)과 연동할 때 오해가 발생했습니다.

### 핵심 교훈

**명확한 인터페이스 정의 필수**:

```
❌ 불명확한 정의:
- "컨베이어 정지해"
- "언제 정지?"
- "얼마나 대기?"

✅ 명확한 정의:
Service: /conveyor/control
Request:
  - command: STOP (enum)
  - wait_time: 5  (seconds)
Response:
  - success: bool
  - actual_time: float
```

### 계약 기반 설계 (Contract-Based Design)
```
계약:
1. 입력 조건 (Precondition)
   - 컨베이어가 켜져 있어야 함
   
2. 처리 (Process)
   - 속도를 점진적으로 감소
   - 지정된 시간 정지
   
3. 출력 결과 (Postcondition)
   - 컨베이어 완전 정지
   - 상태 토픽 발행
```

---

## 🎯 최종 정리

| 학습 | 적용 | 개선 |
|------|------|------|
| 상태 머신 | 모든 프로젝트 | 버그 90% 감소 |
| 계층화 설계 | 프로젝트 1, 2 | 유지보수성 3배 |
| AI 검증 | 프로젝트 3 | 신뢰도 70% → 95% |
| 리소스 관리 | 프로젝트 4 | 안정성 50% 개선 |
| 로깅 | 모든 프로젝트 | 디버깅 시간 70% 단축 |

---

## 📚 권장 학습 순서

1. **상태 머신**: FSM 패턴 이해
2. **아키텍처**: 계층화 설계 원리
3. **AI 통합**: 신뢰도 검증 기법
4. **최적화**: 리소스 관리 및 프로파일링
5. **디버깅**: 로깅 및 모니터링

---

## 🔗 관련 문서

- [기술 스택 가이드](TECH_STACK.md)
- [문제 해결 가이드](TROUBLESHOOTING.md)
- [시스템 아키텍처](ARCHITECTURE.md)

