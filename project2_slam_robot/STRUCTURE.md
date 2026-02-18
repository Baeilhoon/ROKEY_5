# Project 2: SLAM 기반 자율주행 로봇 시스템

이 디렉토리는 다음과 같은 구조로 정리되어야 합니다:

```
project2_slam_robot/
├── README.md                          # 프로젝트 상세 설명서 ✅
│
├── firmware/                          # Arduino 그리퍼 제어 펌웨어
│   ├── gripper_control/
│   │   └── gripper_control.ino
│   ├── libraries/
│   │   ├── StateManager.h
│   │   ├── EEPROMManager.h
│   │   └── ServoController.h
│   └── config/
│       ├── pins.h                     # 핀 정의
│       └── state_config.h             # 상태 정의
│
├── ros_nodes/                         # ROS2 노드 (Python)
│   ├── gripper_bridge_node.py         # Arduino ↔ ROS2 브릿지
│   ├── gripper_control_node.py        # 그리퍼 제어 로직
│   └── CMakeLists.txt
│
├── launch/                            # ROS2 런칭 파일
│   └── slam_robot.launch.py
│
├── config/                            # 설정 파일
│   ├── parameters.yaml
│   └── servo_config.yaml              # 서보 모터 설정
│
├── msg/                               # 커스텀 메시지
│   └── GripperState.msg
│
└── docs/                              # 추가 문서
    ├── state_machine.md               # 상태 머신 설계
    └── eeprom_mapping.md              # EEPROM 맵핑
```

## 🔑 핵심 파일 설명

### firmware/gripper_control.ino
- State-driven 아키텍처
- EEPROM 상태 저장/복원
- 서보 모터 PWM 제어
- 상태 전환 검증

**주요 상태**:
```cpp
enum GripperState {
  OPEN = 0,
  MOVING = 1,
  CLOSED = 2,
  ERROR = 3
};
```

### libraries/EEPROMManager.h
```cpp
class EEPROMManager {
public:
  void save_state(GripperState state, uint8_t position);
  GripperState load_state();
  uint8_t load_position();
};
```

### ros_nodes/gripper_bridge_node.py
- /gripper/state 토픽 발행
- /gripper/control 서비스 제공
- 상태 동기화

## ⚙️ EEPROM 맵핑

| 주소 | 크기 | 내용 | 기본값 |
|------|------|------|--------|
| 0 | 1 byte | 상태 | 0 (OPEN) |
| 1 | 1 byte | 위치 | 90 |
| 2 | 1 byte | 버전 | 1 |
| 3-255 | - | 예약 | - |

## 🧪 테스트

```bash
# 1. Arduino 업로드
cd firmware/gripper_control
arduino-cli compile --fqbn arduino:avr:nano gripper_control

# 2. ROS2 빌드
colcon build --packages-select slam_robot

# 3. 실행
ros2 launch slam_robot slam_robot.launch.py

# 4. 그리퍼 테스트
# 열기
ros2 service call /gripper/control "command: 'open'"

# 닫기
ros2 service call /gripper/control "command: 'close'"

# 모니터링
ros2 topic echo /gripper/state
```

## 📡 메시지 정의

### Message: GripperState
```yaml
std_msgs/Header header
uint8 state              # 0:OPEN, 1:MOVING, 2:CLOSED, 3:ERROR
uint8 position          # 0-180 (서보 각도)
bool ready              # 명령 수행 가능 여부
```

### Service: /gripper/control
```
Request:
  string command          # "open", "close", "position"
  uint8 position          # (position 명령일 때만)
Response:
  bool success
  string status
```

## 🔗 관련 문서

- [전체 설명서 보기](README.md)
- [기술 스택](../../docs/TECH_STACK.md)
- [배운 내용 - EEPROM](../../docs/LESSONS_LEARNED.md#6️⃣-임베디드-메모리-관리)
- [문제 해결](../../docs/TROUBLESHOOTING.md#project-2-slam-자율주행-로봇)
