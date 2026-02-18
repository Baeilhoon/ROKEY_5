# 🚀 Project 4: 디지털트윈 기반 TIAgo 자율배달로봇

> **ROS2 Nav2 + Vision + 시뮬레이터 통합 시스템**  
> Digital Twin-based Autonomous Delivery Robot with TIAGo  

## 📋 개요

`Nav2 · Perception · Manipulation` 동시 실행 환경에서  
**CPU 자원 병목으로 인한 주행 안정성 저하 문제**를 해결한 자율배달 로봇입니다.

시뮬레이터 환경(Gazebo)에서 디지털트윈을 구현하고,  
실제 배달 시나리오를 검증했습니다.

---

## 🎯 핵심 성과

| 성과 | 설명 |
|------|------|
| **CPU 병목 해결** | 기능 선택적 구동으로 시스템 안정성 30% 개선 |
| **Nav2 통합** | AMCL + Local Costmap + DWB 완전 구현 |
| **QR 기반 배달** | QR 인식 후 목표 위치 자동 노비게이션 |
| **시뮬레이션 검증** | Gazebo 환경에서 실제처럼 동작 가능 |

---

## 🔧 내가 맡은 것

### 1️⃣ ROS2 Nav2 기반 자율주행

**Path Planning & Navigation**:
- AMCL (Adaptive Monte Carlo Localization) 기반 자체 위치 파악
- Costmap 기반 장애물 회피
- DWB (Dynamic Window Approach) 로컬 플래닝

```
Global Planner (Nav2)
  ├── Start Position
  ├── Goal Position
  └── Global Path 생성 (Dijkstra / A*)
        ↓
Local Planner (DWB)
  ├── Velocity 계산
  ├── 장애물 회피
  └── 실시간 궤적 조정
```

### 2️⃣ YOLO 기반 객체 및 QR 인식

- 실시간 객체 감지 (사람, 박스, 가구 등)
- QR 코드 감지 및 위치 파악
- 배달 목표 자동 인식

```python
# QR 감지 후 목표 위치로 변환
qr_position = detect_qr_code(frame)  # (x, y, angle)
goal = transform_to_global_coords(qr_position)
send_goal_to_nav2(goal)
```

### 3️⃣ 시스템 실행 구조 분리 및 단계화

**문제**: Nav2 + Perception + LLM 동시 실행 시 CPU 과부하

**해결**: 선택적 기능 구동 구조

```yaml
# operating_mode.yaml
navigation_enabled: true      # 항상 활성
object_detection_enabled: true
qr_detection_enabled: true
llm_inference_enabled: false  # 선택적

# CPU 자원 분배
nav2_threads: 4
perception_threads: 2
```

---

## 🐛 발생한 문제 & 해결

### 1️⃣ Nav2 + Perception 동시 실행 시 FPS 저하

**문제**: 주행 중 갑자기 속도 저하, 방향 불안정  
**원인**: CPU 과부하로 인한 프레임 드롭

실시간 모니터링:
```
| Time | CPU (%) | FPS | Nav2 Latency |
|------|---------|-----|--------------|
| T0   | 85%     | 30  | 50ms ✓      |
| T1   | 120%    | 10  | 150ms ✗     |  ← 현상 발생
| T2   | 95%     | 28  | 60ms ✓      |
```

**해결**: 기능 선택적 구동 + 우선순위 큐

```python
class ResourceManager:
    def __init__(self):
        self.cpu_limit = 0.85  # 85% 넘으면 기능 비활성화
        self.priority_queue = [
            ("nav2", 1),              # 최우선
            ("qr_detection", 2),
            ("object_detection", 3),
            ("llm_inference", 4),     # 최하위
        ]
    
    def update_operating_mode(self):
        """CPU 사용률에 따라 실행 모드 결정"""
        cpu_usage = self.get_cpu_usage()
        
        if cpu_usage > self.cpu_limit:
            # 우선순위 낮은 것부터 비활성화
            self.disable_feature("llm_inference")
            if cpu_usage > 0.95:
                self.disable_feature("object_detection")
        else:
            # 리소스 여유되면 기능 활성화
            self.enable_feature("object_detection")
            if cpu_usage < 0.70:
                self.enable_feature("llm_inference")
```

### 2️⃣ Nav2 Global Planner 경로 최적화 부족

**문제**: 이상한 경로 생성 (불필요한 우회)  
**원인**: 기본 Dijkstra가 비용 함수를 고려하지 않음

**해결**: NavFn 플래너 + 가중치 조정

```yaml
# nav2_params.yaml
global_costmap:
  plugins:
    - static_layer
    - obstacle_layer

obstacles_layer:
  * inflation: 0.5  # 장애물 주변 50cm 위험 영역
  
nav2_planner:
  planner: nav2_navfn_planner/NavfnPlanner
  NavfnPlanner:
    tolerance: 0.1
    use_astar: true
    allow_unknown: true
```

### 3️⃣ QR 인식 각도 불안정

**문제**: QR 감지 후 로봇이 목표 위치에서 어긋난 방향으로 정지  
**원인**: QR의 회전 정보를 무시함

**해결**: QR의 orientation 추출 및 반영

```python
def detect_qr_with_orientation(frame):
    """
    QR 코드 감지 및 orientation 추출
    """
    qr_detector = cv2.QRCodeDetector()
    retval, decoded_info, points, _ = qr_detector.detectAndDecodeMulti(frame)
    
    if retval:
        qr_points = points[0]  # 첫 번째 QR의 코너점
        
        # QR의 방향 계산 (top-left에서 top-right 방향)
        top_left = qr_points[0]
        top_right = qr_points[1]
        
        delta_x = top_right[0] - top_left[0]
        delta_y = top_right[1] - top_left[1]
        
        # 각도 계산 (라디안)
        qr_angle = np.arctan2(delta_y, delta_x)
        
        # 목표 위치 및 각도 설정
        goal = GoalHandle()
        goal.pose.position.x = qr_center[0]
        goal.pose.position.y = qr_center[1]
        goal.pose.orientation = \
            quaternion_from_euler(0, 0, qr_angle)
        
        return goal
    
    return None
```

---

## 🛠️ 기술 스택

```
언어:         Python 3.10  ·  C++17
프레임워크:   ROS2 Humble  ·  ROS2 Nav2
로봇:         TIAGo (Mobile Manipulation Platform)
시뮬레이터:   Gazebo 11
센서:         LiDAR  ·  Depth Camera  ·  Webcam
OS:           Ubuntu 22.04 LTS
AI/CV:        YOLO v8  ·  OpenCV  ·  QR Code Detection
맵:           SLAM 기반 생성 또는 사전 배포된 맵
```

---

## 📊 시스템 아키텍처

```
┌─────────────────────────────────────┐
│      TIAGo (Mobile + Arm)           │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │  Hardware Interface             │ │
│ │  (Motor Control, Sensors)       │ │
│ └──────────┬──────────────────────┘ │
└───────────┼──────────────────────────┘
            │
    ┌───────┴────────────┐
    ↓                    ↓
┌─────────────┐    ┌──────────────┐
│  Nav2 Stack │    │  Perception  │
├─────────────┤    │  Pipeline    │
│ AMCL        │    ├──────────────┤
│ Global Plan │    │ Object Detect│
│ Local Plan  │    │ QR Detect    │
└─────────────┘    └──────────────┘
    ↓                    ↓
    └───────────┬────────┘
                ↓
         ┌──────────────┐
         │ Resource     │
         │ Manager      │
         │ (우선순위)   │
         └──────────────┘
                ↓
        ┌───────────────┐
        │ State Machine │
        │ (IDLE → NAV   │
        │  → DELIVERY)  │
        └───────────────┘
```

---

## 📊 시뮬레이션 환경 (Gazebo)

```yaml
# gazebo_world.launch.py

# 3가지 환경 제공
environments:
  - simple_office      # 사무실 (좁은 통로, QR 3개)
  - warehouse          # 창고 (넓은 공간, 복잡한 배치)
  - multi_floor        # 멀티 플로어 (엘리베이터 시뮬레이션)

# 각 환경에서 배달 경로 테스트
test_scenarios:
  - scenario 1: Start → QR1 → Pick → QR2 → Drop
  - scenario 2: 장애물 회피
  - scenario 3: 연속 배달 (QR1→QR2→...→QR5)
```

---

## 🚀 사용 방법

### 1. 시뮬레이션 환경 시작
```bash
# Gazebo + TIAGo 로드
ros2 launch tiago_gazebo tiago.launch.py

# Nav2 시작
ros2 launch nav2_bringup nav2_bringup_sim.launch.py

# 커스텀 배달 노드
ros2 run tiago_delivery delivery_system.py
```

### 2. 배달 요청
```bash
ros2 service call /request_delivery tiago_msgs/DeliveryRequest {
  target_qr_id: 1,
  object_id: 'box_001'
}
```

### 3. 결과 모니터링
```bash
# RViz에서 시각화
rviz2 -d ./config/delivery.rviz
```

---

## 📈 성능 지표

| 지표 | 값 | 개선도 |
|------|-----|--------|
| 배달 성공률 | 95% | — |
| 평균 배달 시간 | 45초 | — |
| CPU 사용률 | 75% | ↓ 30% |
| Nav2 응답 속도 | 50ms | — |
| 경로 최적성 | 90% | — |

---

## 🎓 배운 것

1. **시뮬레이션과 실제의 격차**
   - Gazebo 시뮬레이션과 실제 로봇은 다름
   - 시뮬레이션에서 동작 ≠ 실제 환경에서 동작

2. **다중 기능 동시 실행의 어려움**
   - CPU/메모리 관리 필수
   - 우선순위 기반 리소스 분배 필수

3. **Navigation Stack 튜닝의 중요성**
   - Nav2는 기본 파라미터로 최적화 불가
   - 환경과 로봇 특성에 맞는 파라미터 조정 필수

4. **엔드-투-엔드 시스템 검증**
   - 각 컴포넌트 개별 테스트와 통합 테스트 분리 필수
   - 상태 머신으로 복잡한 흐름 관리

---

## 📁 주요 파일 구조

```
project4_tiago_delivery/
├── launch/
│   ├── tiago_sim.launch.py
│   └── delivery_system.launch.py
├── src/
│   ├── delivery_system.py
│   ├── qr_detector.py
│   ├── resource_manager.py
│   └── state_machine.py
├── config/
│   ├── nav2_params.yaml
│   ├── perception_config.yaml
│   └── gazebo_worlds/
│       ├── simple_office.world
│       ├── warehouse.world
│       └── multi_floor.world
└── msg/
    └── DeliveryRequest.msg
```

---

## 📝 참고 자료

- **Nav2 공식 문서**: https://navigation.ros.org/
- **TIAGo 로봇**: https://robots.pal-robotics.com/
- **Gazebo 공식**: https://gazebosim.org/
- **YOLO**: https://docs.ultralytics.com/
- **프로젝트 GitHub**: https://github.com/C-2-Organization/tiago-delivery
