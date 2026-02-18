# 📚 기술 스택 가이드

ROKEY 부트캠프 5기 프로젝트에서 사용한 모든 기술을 정리한 문서입니다.

---

## 🛠️ 프로그래밍 언어

### C++ (C++17)
**사용 이유**: 성능, 실시간 제어, 하드웨어 추상화

**주요 사용처**:
- ROS2 노드 작성
- 고성능 연산 필요 부분
- 시스템 통합

**학습 자료**:
- [cppreference.com](https://en.cppreference.com/)
- [ROS2 C++ 튜토리얼](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Publisher-And-Subscriber.html)

---

### Python (3.10)
**사용 이유**: 빠른 프로토타이핑, AI/Vision 라이브러리 풍부

**주요 사용처**:
- Vision 파이프라인 (YOLO, OpenCV)
- ROS2 노드
- 데이터 처리 및 분석

**필수 라이브러리**:
```bash
pip install opencv-python
pip install ultralytics  # YOLO
pip install numpy
pip install rclpy  # ROS2 Python 클라이언트
```

---

### C (Arduino)
**사용 이유**: 마이크로컨트롤러 펌웨어 개발

**주요 사용처**:
- Arduino 기반 하드웨어 제어
- 센서 신호 처리
- 모터 제어

---

## 🤖 ROS2 & 로봇 프레임워크

### ROS2 Humble
**바전**: Ubuntu 22.04 LTS 공식 지원

**주요 개념**:
- **노드 (Node)**: 독립적인 ROS 프로그램
- **토픽 (Topic)**: 발행-구독 메시징
- **서비스 (Service)**: 요청-응답 통신
- **애션 (Action)**: 장시간 작업 (피드백 포함)

**설치**:
```bash
curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update
sudo apt install ros-humble-desktop
```

**학습 자료**:
- [ROS2 공식 문서](https://docs.ros.org/en/humble/)
- [ROS2 튜토리얼](https://docs.ros.org/en/humble/Tutorials.html)

---

### ROS2 Nav2
**용도**: 자율주행 로봇의 경로 계획 및 제어

**주요 컴포넌트**:
- **AMCL**: Adaptive Monte Carlo Localization (위치 파악)
- **Global Planner**: 전역 경로 계획 (Dijkstra, A*)
- **Local Planner**: 실시간 경로 조정 (DWB)
- **Costmap**: 장애물 정보

**설치**:
```bash
sudo apt install ros-humble-navigation2 ros-humble-nav2-bringup
```

---

## 🧠 AI & Vision

### YOLO v8 (Ultralytics)
**용도**: 실시간 객체 감지

**특징**:
- 빠른 추론 속도 (10-30ms)
- 높은 정확도 (mAP 기준)
- 다양한 프리트레인 모델 (nano, small, medium, large, xlarge)

**설치**:
```bash
pip install ultralytics
```

**기본 사용**:
```python
from ultralytics import YOLO

model = YOLO('yolov8s.pt')  # nano, small, medium, large, xlarge
results = model(image)
```

**학습 자료**:
- [YOLO 공식 문서](https://docs.ultralytics.com/)

---

### OpenCV
**용도**: 이미지 전처리, 처리, 표시

**주요 기능**:
- 이미지 읽기/쓰기
- 필터링 (가우시안, 모폴로지 등)
- 히스토그램 균등화
- 특징 검출 (SIFT, ORB 등)

**설치**:
```bash
pip install opencv-python
pip install opencv-contrib-python  # 추가 기능
```

**예제** - 이미지 정규화:
```python
import cv2
import numpy as np

img = cv2.imread('image.jpg')

# 히스토그램 균등화
lab = cv2.cvtColor(img, cv2.COLOR_BGR2LAB)
l, a, b = cv2.split(lab)
l = cv2.equalizeHist(l)
img = cv2.cvtColor(cv2.merge([l, a, b]), cv2.COLOR_LAB2BGR)

# CLAHE (Contrast Limited Adaptive Histogram Equalization)
clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
img_enhanced = clahe.apply(cv2.cvtColor(img, cv2.COLOR_BGR2GRAY))
```

---

## 🤖 로봇 플랫폼

### Arduino
**보드**: UNO, MEGA, Nano

**주요 기능**:
- GPIO 제어
- PWM (Pulse Width Modulation) - 모터 속도 제어
- 직렬 통신 (Serial)
- EEPROM - 비휘발성 메모리

**프로그래밍**:
```cpp
void setup() {
  Serial.begin(9600);
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(1000);
  digitalWrite(LED_PIN, LOW);
  delay(1000);
}
```

---

### TurtleBot3
**플랫폼**: 저비용 모바일 로봇

**구성**:
- Raspberry Pi 보드
- OpenCR 마이크로컨트롤러
- LiDAR 센서
- 차동 구동 (Differential Drive)

**SLAM 실행**:
```bash
# 터미널 1: TurtleBot3 드라이버
ros2 launch turtlebot3_bringup robot.launch.py

# 터미널 2: SLAM
ros2 launch turtlebot3_slam slam.launch.py
```

---

### TIAGo
**플랫형**: 모바일 매니퓨레이터 (이동형 로봇팔)

**특징**:
- 기저: 차동 구동
- 팔: 7-DOF 로봇팔
- 카메라, LiDAR, 그리퍼

---

### Doosan 협동로봇 (M0609)
**용도**: 인간-로봇 협업 (Human-Robot Collaboration)

**특징**:
- 6-DOF 관절형 로봇팔
- 최대 페이로드: 9kg
- 안전성 (힘 제한)

---

## 📊 시뮬레이션 환경

### Gazebo
**용도**: 로봇 시뮬레이션

**실행**:
```bash
# 기본 Gazebo 시작
gazebo

# ROS와 통합
ros2 launch gazebo_ros gazebo.launch.py
```

---

## 📡 통신 프로토콜

### Serial Communication (UART)
**속도**: 9600 ~ 115200 baud

**Arduino 코드**:
```cpp
void setup() {
  Serial.begin(9600);
}

void loop() {
  if (Serial.available()) {
    char cmd = Serial.read();
    Serial.println("Received: " + String(cmd));
  }
}
```

---

## 🔧 개발 도구

### VS Code
**확장 프로그램**:
- ROS (Microsoft)
- C/C++ (Microsoft)
- Python (Microsoft)
- Markdown All in One

---

### Git & GitHub
**기본 명령**:
```bash
git clone <repo-url>
git add .
git commit -m "message"
git push origin main
```

---

## 📚 참고 자료 모음

| 분야 | 자료 |
|------|------|
| ROS2 | [공식 문서](https://docs.ros.org/en/humble/) |
| Nav2 | [Navigation2](https://navigation.ros.org/) |
| YOLO | [Ultralytics](https://docs.ultralytics.com/) |
| OpenCV | [공식 튜토리얼](https://docs.opencv.org/) |
| Arduino | [공식 레퍼런스](https://www.arduino.cc/reference/) |
| Python | [Python 공식](https://www.python.org/) |
| C++ | [cppreference](https://en.cppreference.com/) |

