# 🎯 Project 3: 멀티모달 LLM 협동로봇 지능제어 시스템

> **AI 추론 결과 → ROS2 토픽 변환 → 로봇 동작 연동**  
> Multimodal LLM-based Collaborative Robot Intelligence System  

## 📋 개요

LLM·Vision·로봇 제어를 연결한 협동로봇 시스템에서  
**Webcam 인식 결과를 ROS2 스킬 흐름과 연동**하는 시스템입니다.

인식 불안정 문제를 구조적으로 해결하고,  
**신뢰도 기준 + 연속 프레임 조건 + 중간 검증 단계**를 통해  
안정적인 AI 기반 로봇 제어를 구현했습니다.

> ⚠️ **중요**: AI 모델 자체를 개발한 것이 아닙니다.  
> 기존 모델의 출력을 로봇 제어에 **안정적으로 연결하는 인터페이스 설계**에 집중했습니다.

---

## 🎯 핵심 성과

| 성과 | 설명 |
|------|------|
| **신뢰도 기반 필터링** | 낮은 확률의 오인식 사전 차단 |
| **연속 프레임 검증** | 단일 프레임 오류 방지 (3프레임 연속 검증) |
| **2단계 검증 시스템** | 초기 인식 + 작업 전 재검증으로 안정성 확보 |
| **로봇 연동 파이프라인** | Vision 추론 → ROS2 토픽 → 로봇 스킬 실행 |

---

## 🔧 내가 맡은 것

### 1️⃣ Webcam 기반 인식 로직 구현
- YOLO 기반 실시간 객체 감지
- OpenCV를 통한 이미지 전처리
- 신뢰도(confidence) 필터링

### 2️⃣ 인식 결과 → ROS2 스킬 흐름 연동
- Vision 토픽 구독 및 처리
- 로봇 명령 토픽 발행
- 동기화된 제어 흐름

### 3️⃣ Handover / Dump 시나리오 설계
- Handover: 물체를 인간에게 전달
- Dump: 물체를 특정 위치에 배치
- 각 시나리오별 검증 조건 정의

### 4️⃣ 오동작 필터링 시스템

```
Raw Webcam Input
    ↓
[1] YOLO 추론 + Confidence > 0.7 필터링
    ↓
[2] 연속 프레임 검증 (3프레임 연속 동일 결과)
    ↓
[3] ROS2 Topic 발행
    ↓
[4] 로봇 동작 시작
    ↓
[5] 작업 전 재검증 (최종 확인)
    ↓
[6] 스킬 실행
```

---

## 🐛 발생한 문제 & 해결

### 1️⃣ 인식 오류로 잘못된 행동 트리거

**문제**: Webcam 오인식으로 인해 로봇이 잘못된 동작 수행  
예) 물체 없는데도 집으려고 움직임, 이미징 노이즈로 오동작

**원인**:
- 단일 프레임 결과 바로 사용
- 낮은 신뢰도(confidence) 값도 넘김

**해결**: 다단계 검증 시스템 도입

```python
class ObjectDetectionPipeline:
    def __init__(self):
        self.confidence_threshold = 0.7
        self.frame_buffer = []
        self.buffer_size = 3  # 3프레임 연속 검증
    
    def process_frame(self, frame):
        """
        1단계: YOLO 추론
        """
        detections = self.yolo_model(frame)
        
        # Confidence 필터링
        valid_detections = [
            d for d in detections 
            if d.confidence > self.confidence_threshold
        ]
        
        if not valid_detections:
            return None
        
        """
        2단계: 연속 프레임 검증
        """
        self.frame_buffer.append(valid_detections[0])
        
        if len(self.frame_buffer) < self.buffer_size:
            return None  # 아직 충분한 프레임 없음
        
        # 최근 3프레임 검증
        if self.is_consistent(self.frame_buffer[-3:]):
            result = self.frame_buffer[-1]
            self.frame_buffer = []  # 버퍼 초기화
            return result
        else:
            self.frame_buffer.pop(0)  # 가장 오래된 프레임 제거
            return None
    
    def is_consistent(self, detections):
        """
        연속 프레임의 일관성 검증
        - 동일 객체 클래스 확인
        - 위치 편차 내 확인
        """
        if len(detections) < 3:
            return False
        
        class_ids = [d.class_id for d in detections]
        positions = [d.position for d in detections]
        
        # 모두 같은 클래스인가?
        if not all(c == class_ids[0] for c in class_ids):
            return False
        
        # 위치 편차가 임계값 내인가? (픽셀 단위)
        center = positions[1]  # 중간 프레임을 기준
        max_deviation = 20  # 20 픽셀
        
        for pos in positions:
            if abs(pos[0] - center[0]) > max_deviation or \
               abs(pos[1] - center[1]) > max_deviation:
                return False
        
        return True
```

### 2️⃣ ROS2 토픽과 로봇 스킬 동기화

**문제**: Vision 토픽 발행과 로봇 동작 간 타이밍 미스매치  
**원인**: 비동기 통신에서 명확한 상태 전환 규칙 부재

**해결**: 상태 머신 기반 제어 흐름

```python
from enum import Enum
import rospy
from geometry_msgs.msg import Twist

class RobotState(Enum):
    IDLE = 0
    DETECTING = 1
    VERIFIED = 2
    EXECUTING = 3
    DONE = 4

class RobotController:
    def __init__(self):
        self.state = RobotState.IDLE
        self.vision_sub = rospy.Subscriber('/vision/detection', Detection, self.vision_callback)
        self.command_pub = rospy.Publisher('/robot/command', Twist, queue_size=1)
    
    def vision_callback(self, msg):
        """
        Vision으로부터 검증된 검출 수신
        """
        if self.state == RobotState.IDLE:
            self.state = RobotState.DETECTING
            self.handle_detection(msg)
    
    def handle_detection(self, detection):
        """
        검출에 따른 스킬 실행
        """
        if detection.action == "HANDOVER":
            self.execute_handover(detection)
        elif detection.action == "DUMP":
            self.execute_dump(detection)
        else:
            self.state = RobotState.IDLE
            return
        
        self.state = RobotState.EXECUTING
    
    def execute_handover(self, detection):
        """
        물체를 인간에게 전달
        """
        # 1. 최종 재검증 (작업 전 한 번 더 확인)
        if not self.final_verify(detection):
            self.state = RobotState.IDLE
            return
        
        # 2. 로봇팔 움직임
        cmd = Twist()
        cmd.linear.x = detection.target_position.x
        cmd.linear.y = detection.target_position.y
        self.command_pub.publish(cmd)
        
        # 3. 작업 완료 대기
        rospy.sleep(2.0)  # 2초 대기
        
        self.state = RobotState.DONE
```

### 3️⃣ 이미징 조명 변화에 민감

**문제**: 조명이 바뀌면 인식률 급하락  
**원인**: 전처리 없이 raw 이미지 바로 사용

**해결**: 이미지 정규화 및 히스토그램 균등화

```python
import cv2
import numpy as np

def preprocess_frame(frame):
    """
    이미지 전처리로 조명 변화 대응
    """
    # 1. 히스토그램 균등화
    lab = cv2.cvtColor(frame, cv2.COLOR_BGR2LAB)
    l, a, b = cv2.split(lab)
    l = cv2.equalizeHist(l)
    lab = cv2.merge([l, a, b])
    frame = cv2.cvtColor(lab, cv2.COLOR_LAB2BGR)
    
    # 2. CLAHE (Contrast Limited Adaptive Histogram Equalization)
    clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    gray = clahe.apply(gray)
    
    # 3. 정규화
    frame = cv2.normalize(frame, None, 0, 255, cv2.NORM_MINMAX)
    
    return frame
```

---

## 🛠️ 기술 스택

```
언어:         Python 3.10  ·  C++17
프레임워크:   ROS2 Humble
AI/CV:        YOLO v8  ·  OpenCV  ·  TensorFlow/PyTorch
센서/하드웨어: Webcam  ·  Doosan Collaborative Robot
OS:           Ubuntu 22.04 LTS
협업 도구:    GitHub  ·  Notion
```

---

## 📊 파이프라인 아키텍처

```
┌─────────────┐
│   Webcam    │
└──────┬──────┘
       ↓
┌─────────────────────────┐
│  Image Preprocessing    │
│  (기울기, 조명 정규화)   │
└──────┬──────────────────┘
       ↓
┌─────────────────────────┐
│  YOLO Inference         │
│  (객체 감지)             │
└──────┬──────────────────┘
       ↓
┌─────────────────────────┐
│  Confidence Filter      │
│  (>= 0.7)               │
└──────┬──────────────────┘
       ↓
┌─────────────────────────┐
│  Frame Buffer           │
│  (3프레임 검증)          │
└──────┬──────────────────┘
       ↓
       No → 버퍼에 추가 → 대기
       ↓
      Yes
       ↓
┌─────────────────────────┐
│  ROS2 Topic Publish     │
│  (/vision/detection)    │
└──────┬──────────────────┘
       ↓
┌─────────────────────────┐
│  Robot State Machine    │
│  (IDLE → EXECUTING)     │
└──────┬──────────────────┘
       ↓
┌─────────────────────────┐
│  Final Verification     │
│  (작업 전 재검증)        │
└──────┬──────────────────┘
       ↓
┌─────────────────────────┐
│  Robot Skill Execution  │
│  (Handover / Dump)      │
└──────┬──────────────────┘
       ↓
┌─────────────────────────┐
│  Result Logging         │
└─────────────────────────┘
```

---

## 📝 메시지 정의

```yaml
# Vision Detection Message
std_msgs/Header header
string action              # "HANDOVER" or "DUMP"
float32 confidence         # 0.0 ~ 1.0
geometry_msgs/Point target_position

# Robot Command Message
std_msgs/Header header
string skill_type          # "HANDOVER", "DUMP", "INSPECT"
geometry_msgs/Pose target_pose
```

---

## 🎓 배운 것

1. **AI 모델과 로봇 제어 간 격차**
   - AI 추론 결과를 바로 사용할 수 없음
   - 안정성을 위한 추가 검증 계층 필수

2. **다중 센서 퓨전의 필요성**
   - 단일 센서(카메라)만으로는 부족
   - 연속성 검증, 신뢰도 기준 등으로 보완

3. **상태 머신의 확장성**
   - 로봇-AI 연동에서도 상태 관리 필수
   - 명확한 상태 전환이 안정성 보장

4. **프레임 기반 처리의 중요성**
   - 단일 프레임 오류에 강건하게 설계
   - 버퍼링과 연속성 검증으로 오류 필터링

---

## 📁 주요 파일 구조

```
project3_multimodal_llm/
├── src/
│   ├── vision_pipeline.py
│   ├── robot_controller.py
│   ├── preprocessing.py
│   └── detection_node.py
├── config/
│   ├── yolo_config.yaml
│   └── detection_thresholds.yaml
└── launch/
    └── multimodal_system.launch.py
```

---

## 📝 참고 자료

- **YOLO 공식 문서**: https://docs.ultralytics.com/
- **OpenCV 튜토리얼**: https://docs.opencv.org/
- **ROS2 Humble**: https://docs.ros.org/en/humble/
- **프로젝트 GitHub**: https://github.com/C-2-Organization/DUM-E
