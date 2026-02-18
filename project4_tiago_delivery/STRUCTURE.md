# Project 4: 디지털트윈 기반 TIAGo 자율배달로봇

이 디렉토리는 다음과 같은 구조로 정리되어야 합니다:

```
project4_tiago_delivery/
├── README.md                          # 프로젝트 상세 설명서 ✅
│
├── launch/                            # ROS2 런칭 파일
│   ├── tiago_sim.launch.py            # 시뮬레이터 환경
│   ├── delivery_system.launch.py      # 배달 시스템
│   └── nav2_sim.launch.py             # Nav2 런칭
│
├── src/                               # ROS2 Python 노드
│   ├── delivery_system.py             # 메인 배달 로직
│   ├── resource_manager.py            # CPU 리소스 관리
│   ├── state_machine.py               # 상태 머신 (IDLE → EXECUTING)
│   ├── qr_detector.py                 # QR 코드 감지
│   ├── path_planner.py                # 경로 계획 인터페이스
│   └── __init__.py
│
├── config/                            # 설정 파일
│   ├── nav2_params.yaml               # Nav2 파라미터
│   ├── perception_config.yaml         # YOLO, QR 설정
│   ├── resource_limits.yaml           # CPU 제한
│   ├── delivery_routes.yaml           # 배달 경로 정의
│   └── robot_config.yaml              # 로봇 설정
│
├── msg/                               # 커스텀 메시지
│   ├── DeliveryRequest.msg
│   ├── RobotStatus.msg
│   └── DeliveryTask.msg
│
├── gazebo_worlds/                     # Gazebo 시뮬레이션 환경
│   ├── simple_office.world
│   ├── warehouse.world
│   └── multi_floor.world              # 멀티 플로어
│
├── models/                            # Gazebo 모델 (SDF)
│   ├── qr_code_marker/
│   ├── package_delivery/
│   └── office_furniture/
│
├── rviz/                              # RViz 설정
│   └── delivery.rviz
│
├── test/                              # 테스트 코드
│   ├── test_nav2.py
│   ├── test_qr_detection.py
│   └── test_resource_manager.py
│
└── docs/                              # 추가 문서
    ├── navigation_tuning.md           # Nav2 튜닝 가이드
    ├── simulation_guide.md            # 시뮬레이션 가이드
    └── deployment_checklist.md        # 배포 체크리스트
```

## 🔑 핵심 파일 설명

### src/delivery_system.py
메인 배달 로직:
```python
class DeliverySystem:
  def __init__(self):
    self.state = "IDLE"
    self.nav_client = ActionClient(...)
    self.resource_manager = ResourceManager()
  
  def execute_delivery(self, request):
    # 1. QR 감지
    # 2. 경로 계획
    # 3. 주행
    # 4. 배달 수행
```

### src/resource_manager.py
CPU 리소스 관리:
```python
class ResourceManager:
  PRIORITY = [
    ("nav2", 1),
    ("perception", 2),
    ("llm", 3)
  ]
  
  def manage_features(self):
    # CPU 사용률 모니터링
    # 기능 활성화/비활성화
```

### src/qr_detector.py
QR 코드 감지 + 방향 추출:
```python
class QRDetector:
  def detect_with_orientation(self, frame):
    # QR 중심 위치
    # QR 회전 각도
    return (position, angle)
```

## ⚙️ 설정 예제

### config/nav2_params.yaml
```yaml
planner_server:
  ros__parameters:
    use_sim_time: true
    planner_plugins: ["GridBased"]
    
    GridBased:
      plugin: nav2_navfn_planner/NavfnPlanner
      tolerance: 0.1
      use_astar: true

controller_server:
  ros__parameters:
    controller_frequency: 20.0
    min_x_velocity_threshold: 0.001
    min_y_velocity_threshold: 0.001
    min_theta_velocity_threshold: 0.001
    
    DWBLocalPlanner:
      min_vel_x: 0.0
      max_vel_x: 0.26
      max_vel_theta: 1.0
```

### config/resource_limits.yaml
```yaml
cpu_monitoring:
  enabled: true
  check_interval: 1.0  # seconds
  
thresholds:
  critical: 0.95
  high: 0.85
  normal: 0.75
  
features:
  nav2:
    priority: 1
    always_enabled: true
  perception:
    priority: 2
    disable_at: critical
  llm:
    priority: 3
    disable_at: high
```

## 🧪 테스트

```bash
# 1. Gazebo 시뮬레이터 실행
ros2 launch tiago_simulation tiago.launch.py

# 2. Nav2 스택 실행
ros2 launch nav2_bringup nav2_bringup_sim.launch.py

# 3. 배달 시스템 실행
ros2 launch tiago_delivery delivery_system.launch.py

# 4. 배달 요청 (테스트)
ros2 service call /request_delivery tiago_msgs/DeliveryRequest "{target_qr_id: 1, object_id: 'box_001'}"

# 5. 모니터링
ros2 topic echo /tiago/status
ros2 topic echo /robot/position
```

## 📡 메시지 정의

### Message: DeliveryRequest
```yaml
uint8 target_qr_id         # QR 코드 ID (1-10)
string object_id           # 배달할 물체 ID
string priority            # "normal", "urgent"
```

### Message: RobotStatus
```yaml
std_msgs/Header header
string state                # "IDLE", "NAVIGATING", "DELIVERING", "ERROR"
geometry_msgs/Pose current_pose
float32 battery_level       # 0.0-1.0
string last_action
```

## 🌍 시뮬레이션 환경

### simple_office.world
- 사무실 환경
- QR 3개 배치
- 좁은 복도

### warehouse.world
- 창고 환경
- 넓은 공간
- QR 5개

### multi_floor.world (선택)
- 2층 건물
- 엘리베이터 자동화

## 🔗 관련 문서

- [전체 설명서 보기](README.md)
- [기술 스택 - Nav2](../../docs/TECH_STACK.md)
- [배운 내용 - 리소스 관리](../../docs/LESSONS_LEARNED.md#4️⃣-시스템-성능-최적화)
- [아키텍처](../../docs/ARCHITECTURE.md)
- [문제 해결](../../docs/TROUBLESHOOTING.md#project-4-tiago-자율배달로봇)

## 📦 의존성

```bash
# requirements.txt
nav2_bringup
nav2_core
nav2_msgs
tf2
tf2_ros
rclpy
geometry_msgs
sensor_msgs
opencv-python
ultralytics
```

## 📊 성능 목표

| 지표 | 목표 | 현황 |
|------|------|------|
| 배달 성공률 | 95% | ✅ |
| 평균 배달 시간 | 60초 | ✅ 45초 |
| CPU 효율 | <80% | ✅ 75% |
| Nav2 응답 | <100ms | ✅ 50ms |
