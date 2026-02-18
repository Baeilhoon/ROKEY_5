# Project 1: 컨베이어벨트 자동화 공정 제어 시스템

이 디렉토리는 다음과 같은 구조로 정리되어야 합니다:

```
project1_conveyor_belt/
├── README.md                          # 프로젝트 상세 설명서 ✅
│
├── firmware/                          # Arduino 펌웨어
│   ├── conveyor_control/
│   │   └── conveyor_control.ino
│   ├── libraries/
│   │   └── (필요한 라이브러리 헤더)
│   └── config/
│       └── config.h                   # 핀 설정, 상수 정의
│
├── src/                               # ROS2 C++ 노드
│   ├── conveyor_bridge_node.cpp       # Arduino ↔ ROS2 브릿지
│   ├── state_machine.cpp              # 상태 머신 로직
│   ├── CMakeLists.txt
│   └── package.xml
│
├── launch/                            # ROS2 런칭 파일
│   └── conveyor_system.launch.py
│
├── config/                            # 설정 파일
│   ├── parameters.yaml                # ROS2 파라미터
│   └── state_config.yaml              # 상태 정의
│
└── docs/                              # 추가 문서
    ├── state_diagram.md               # 상태 다이어그램
    └── protocol.md                    # 통신 프로토콜 명세
```

## 🔑 핵심 파일 설명

### firmware/conveyor_control.ino
- IR 센서 신호 처리
- 스텝 모터 제어
- 상태 플래그 관리
- Serial 통신

### src/conveyor_bridge_node.cpp
- Arduino와 ROS2 간 메시지 변환
- `/conveyor/state` 토픽 발행
- `/conveyor/command` 서비스 제공
- 직렬 포트 관리

### launch/conveyor_system.launch.py
```python
ros2 launch conveyor_system conveyor_system.launch.py
```

## ⚙️ 설정 예제

### config/parameters.yaml
```yaml
conveyor:
  serial_port: "/dev/ttyUSB0"
  baud_rate: 115200
  max_speed: 255
  min_speed: 0
  debounce_ms: 50
  
state_machine:
  idle_timeout: 30
  emergency_stop_enable: true
```

## 🧪 테스트

```bash
# 1. Arduino 업로드
cd firmware/conveyor_control
platformio upload

# 2. ROS2 빌드
cd ../..
colcon build --packages-select conveyor_system

# 3. 실행
ros2 launch conveyor_system conveyor_system.launch.py

# 4. 테스트
ros2 service call /conveyor/command "{ command: 100, direction: 1 }"
ros2 topic echo /conveyor/state
```

## 📡 메시지 정의

### Topic: /conveyor/state
```
std_msgs/Header header
int32 current_state          # 0:IDLE, 1:MOVING, 2:WAITING, 3:EMERGENCY
int32 current_speed
geometry_msgs/Timestamp last_update
```

### Service: /conveyor/command
```
Request:
  int32 command              # 0:STOP, 1:START, 2:RESUME
  int32 speed                # 0-255
Response:
  bool success
  string message
```

## 🔗 관련 문서

- [전체 설명서 보기](README.md)
- [기술 스택](../../docs/TECH_STACK.md)
- [문제 해결](../../docs/TROUBLESHOOTING.md#project-1-컨베이어벨트-자동화)
