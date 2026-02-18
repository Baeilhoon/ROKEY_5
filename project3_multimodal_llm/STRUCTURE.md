# Project 3: 멀티모달 LLM 협동로봇 지능제어 시스템

이 디렉토리는 다음과 같은 구조로 정리되어야 합니다:

```
project3_multimodal_llm/
├── README.md                          # 프로젝트 상세 설명서 ✅
│
├── src/                               # 소스 코드 (Python)
│   ├── vision_pipeline.py             # YOLO 기반 비전 파이프라인
│   ├── frame_validator.py             # 프레임 검증 (다단계)
│   ├── image_preprocessor.py          # 이미지 정규화
│   ├── robot_controller.py            # 로봇 제어 로직
│   ├── state_machine.py               # 상태 머신
│   ├── detection_node.py              # ROS2 노드
│   └── __init__.py
│
├── config/                            # 설정 파일
│   ├── yolo_config.yaml               # YOLO 모델 설정
│   ├── detection_thresholds.yaml      # 검증 임계값
│   ├── camera_config.yaml             # 카메라 설정
│   └── robot_skills.yaml              # 로봇 스킬 정의
│
├── launch/                            # ROS2 런칭 파일
│   ├── multimodal_system.launch.py    # 전체 시스템
│   └── vision_only.launch.py          # Vision만 테스트
│
├── msg/                               # 커스텀 메시지
│   ├── Detection.msg
│   ├── VerifiedDetection.msg
│   └── RobotSkillRequest.msg
│
├── models/                            # 학습된 모델 (큼)
│   └── yolov8s.pt                     # 또는 다운로드 링크
│
├── test/                              # 테스트 코드
│   ├── test_frame_validator.py
│   ├── test_image_preprocessor.py
│   └── test_videos/                   # 테스트 영상
│
└── docs/                              # 추가 문서
    ├── pipeline_flow.md               # 파이프라인 흐름
    ├── validation_strategy.md         # 검증 전략
    └── skill_definitions.md           # 스킬 정의
```

## 🔑 핵심 파일 설명

### src/vision_pipeline.py
```python
class ObjectDetectionPipeline:
  def __init__(self):
    self.yolo_model = YOLO('yolov8s.pt')
    self.frame_buffer = []
    self.preprocessor = ImagePreprocessor()
  
  def process_frame(self, frame):
    # 1. 전처리
    # 2. YOLO 추론
    # 3. Confidence 필터링
    # 4. 프레임 버퍼 관리
    # 5. 일관성 검증
```

### src/frame_validator.py
```python
class FrameValidator:
  def validate(self, detections, buffer):
    # 3프레임 일관성 검증
    # 위치 안정성 검증
    # 신뢰도 검증
    pass
```

### src/image_preprocessor.py
```python
class ImagePreprocessor:
  @staticmethod
  def preprocess(frame):
    # LAB 히스토그램 균등화
    # CLAHE 적용
    # 감마 보정
    pass
```

### src/robot_controller.py
상태 머신 기반 로봇 제어:
```python
class RobotController:
  def handle_detection(self, detection):
    if detection.action == "HANDOVER":
      self.execute_handover(detection)
    elif detection.action == "DUMP":
      self.execute_dump(detection)
```

## ⚙️ 설정 예제

### config/detection_thresholds.yaml
```yaml
confidence:
  initial_threshold: 0.7
  strict_threshold: 0.8

frame_buffer:
  buffer_size: 3
  consistency_mode: "strict"  # strict, normal, relaxed

position_stability:
  max_pixel_deviation: 20
  max_angle_deviation: 15

verification:
  final_check_enabled: true
  final_check_threshold: 0.75
```

## 🧪 테스트

```bash
# 1. 환경 설정
pip install -r requirements.txt
python -m pip install ultralytics opencv-python

# 2. 모델 다운로드
python -c "from ultralytics import YOLO; YOLO('yolov8s.pt')"

# 3. 테스트 (비전 파이프라인만)
python src/test_vision_pipeline.py

# 4. ROS2 실행
ros2 launch multimodal_system multimodal_system.launch.py

# 5. 토픽 모니터링
ros2 topic echo /vision/detection
ros2 topic echo /robot/skill_request
```

## 📡 메시지 정의

### Message: Detection
```yaml
std_msgs/Header header
string object_type        # "cup", "person", etc.
float32 confidence        # 0.0-1.0
geometry_msgs/Point position
geometry_msgs/Vector3 size
```

### Message: VerifiedDetection
```yaml
std_msgs/Header header
string action             # "HANDOVER", "DUMP"
geometry_msgs/Pose target_pose
float32 final_confidence  # 재검증 신뢰도
```

## 🔗 관련 문서

- [전체 설명서 보기](README.md)
- [기술 스택 - YOLO & OpenCV](../../docs/TECH_STACK.md)
- [배운 내용 - AI 검증](../../docs/LESSONS_LEARNED.md#3️⃣-ai-신뢰도의-현실성)
- [문제 해결](../../docs/TROUBLESHOOTING.md#project-3-llm-협동로봇)

## 📦 의존성

```bash
# requirements.txt
ultralytics>=8.0.0
opencv-python>=4.8.0
numpy>=1.24.0
rclpy>=0.13.0
sensor_msgs
geometry_msgs
```
