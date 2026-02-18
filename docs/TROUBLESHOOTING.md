# 🔧 문제 해결 가이드 (Troubleshooting)

프로젝트 진행 중 발생한 주요 문제와 해결 방법을 정리합니다.

---

## Project 1: 컨베이어벨트 자동화

### ❌ 문제 1: IR 센서 노이즈로 중복 이벤트 발생

**증상**:
- 물체가 한 번 지나가는데 이벤트가 3~5번 발생
- 로봇팔이 여러 번 움직이려고 시도

**원인**:
- IR 센서의 아날로그 신호가 임계값 근처에서 진동
- 직렬 통신 지연으로 여러 신호 누적

**해결**:
```cpp
// 디바운싱: 신호 변화 후 일정 시간 무시
#define DEBOUNCE_MS 50

bool debounced_read(int pin) {
  if (digitalRead(pin) != last_state) {
    delay(DEBOUNCE_MS);
    if (digitalRead(pin) != last_state) {
      last_state = digitalRead(pin);
      return true;  // 상태 변화 확정
    }
  }
  return false;
}
```

**테스트**:
```bash
# 물체를 천천히 통과시킴
# 이벤트가 1번만 발생하는지 확인
```

---

### ❌ 문제 2: 컨베이어-로봇팔 동기화 깨짐

**증상**:
- 로봇팔이 움직이는 중에 컨베이어도 움직임
- 물체가 로봇팔의 그리퍼 범위를 벗어남

**원인**:
- 두 제어 시스템 간 우선순위 없음
- 비동기 메시징으로 순서 보장 불가능

**해결**:
```cpp
// 상태 기반 제어: 상태별 허용 작업 정의
enum State { IDLE, MOVING, WAITING, EMERGENCY };

bool can_move_arm() {
  return (state == IDLE || state == WAITING);
}

bool can_move_conveyor() {
  return (state == IDLE);  // 로봇팔이 작동 중면 금지
}
```

**ROS2 인터페이스**:
```
Service: /conveyor/status_check
Request: -
Response:
  - state: int (0:IDLE, 1:MOVING, ...)
  - can_accept_gripper: bool
```

**테스트**:
```bash
# 로봇팔이 물체를 집는 동안 컨베이어 정지 확인
ros2 service call /conveyor/control "command: 100" &
ros2 service call /gripper/grab "speed: 50"
# 동시 실행 시 컨베이어가 정지하는지 확인
```

---

### ❌ 문제 3: ROS2-Arduino 통신 지연

**증상**:
- 한 번의 명령이 여러 번 실행됨
- 500ms 이상의 지연 발생

**원인**:
- 직렬 포트 버퍼 오버플로우
- 여러 노드가 동시에 명령을 보냄
- 응답 확인 메커니즘 부재

**해결 (브릿지 노드 분리)**:
```python
# conveyor_bridge.py - Arduino와 직렬 통신하는 전용 노드
class ConveyorBridge:
  def __init__(self):
    self.serial_port = serial.Serial('/dev/ttyUSB0', 115200)
    self.command_queue = Queue()
    self.processing = False
  
  def command_callback(self, msg):
    # 명령을 큐에 추가 (즉시 실행 X)
    self.command_queue.put(msg)
  
  def process_queue(self):
    while True:
      if not self.command_queue.empty() and not self.processing:
        cmd = self.command_queue.get()
        self.execute_command(cmd)
      time.sleep(0.1)
  
  def execute_command(self, cmd):
    # 명령 실행
    self.processing = True
    self.serial_port.write(cmd.data)
    
    # 응답 대기 (최대 1초)
    start = time.time()
    while time.time() - start < 1.0:
      if self.serial_port.in_waiting:
        response = self.serial_port.read()
        if response == b'OK':
          break
    
    self.processing = False
```

**테스트**:
```bash
# Arduino SIM 환경
python conveyor_bridge.py
# 동일 명령 여러 번 전송
ros2 service call /conveyor/control "command: 100"
ros2 service call /conveyor/control "command: 100"
ros2 service call /conveyor/control "command: 100"
# 큐 처리로 순차 실행 확인
```

---

## Project 2: SLAM 자율주행 로봇

### ❌ 문제 1: 상태 전환 중 reset() 재호출

**증상**:
- 그리퍼가 작업 중에 초기 위치로 복귀
- 불규칙한 동작 반복

**원인**:
```cpp
// ❌ 잘못된 코드
void setup() {
  initialize_gripper();
}

void loop() {
  initialize_gripper();  // 매 루프마다 실행!
  handle_command();
}
```

**해결**:
```cpp
// ✅ 올바른 코드
volatile bool initialized = false;

void setup() {
  initialize_gripper();
  initialized = true;
}

void loop() {
  if (initialized) {
    handle_command();  // 초기화 스킵
  }
}
```

---

### ❌ 문제 2: 상태 플래그 관리 부족

**증상**:
- 그리퍼가 열린 상태인데 닫으려고 시도
- 오류 복구 불가능

**해결**:
```cpp
// State-Driven 아키텍처
enum GripperState {
  OPEN = 0,
  MOVING = 1,
  CLOSED = 2,
  ERROR = 3
};

GripperState current_state = OPEN;

bool request_state_change(GripperState new_state) {
  // 유효한 전환인가?
  bool valid = false;
  
  switch(current_state) {
    case OPEN:
      valid = (new_state == MOVING || new_state == ERROR);
      break;
    case MOVING:
      valid = (new_state == CLOSED || new_state == OPEN || 
               new_state == ERROR);
      break;
    case CLOSED:
      valid = (new_state == MOVING || new_state == ERROR);
      break;
    case ERROR:
      valid = (new_state == OPEN);  // 오류 복구: OPEN으로만
      break;
  }
  
  if (valid) {
    current_state = new_state;
    return true;
  }
  return false;
}
```

---

### ❌ 문제 3: 전원 재시작 후 상태 유실

**증상**:
- 로봇 부팅 후 그리퍼 위치 미파악
- 초기 명령 실패

**원인**:
```cpp
// ❌ RAM에만 저장
int position = 90;  // 전원 끄면 사라짐!
```

**해결 (EEPROM)**:
```cpp
#include <EEPROM.h>

#define EEPROM_STATE_ADDR 0
#define EEPROM_POSITION_ADDR 1

void save_state() {
  EEPROM.write(EEPROM_STATE_ADDR, (byte)current_state);
  EEPROM.write(EEPROM_POSITION_ADDR, current_position);
}

void restore_state_on_boot() {
  GripperState saved_state = 
    (GripperState)EEPROM.read(EEPROM_STATE_ADDR);
  int saved_position = EEPROM.read(EEPROM_POSITION_ADDR);
  
  // 부트 시 복원
  current_state = saved_state;
  servo.write(saved_position);
}

void setup() {
  restore_state_on_boot();
}
```

**테스트**:
```bash
# 1. 그리퍼 위치 설정
# 2. 로봇 전원 끔
# 3. 로봇 재부팅
# 4. 그리퍼가 이전 위치로 복구되는지 확인
```

---

## Project 3: LLM 협동로봇

### ❌ 문제 1: 인식 오류로 오동작

**증상**:
- 물체가 없는데도 액션 실행
- 손 트래킹 오인식으로 우발적 작동

**원인**:
```python
# ❌ 단일 프레임 바로 사용
result = yolo_model(frame)
if result.confidence > 0.5:  # 임계값 낮음
  execute_action(result)  # 즉시 실행!
```

**해결 (다단계 검증)**:
```python
class SafeVisionPipeline:
  def __init__(self):
    self.frame_buffer = []
    self.confidence_threshold = 0.7
    self.buffer_size = 3
    self.max_position_deviation = 20  # pixels
  
  def validate_detection(self, frame):
    # 1단계: Confidence 필터링
    result = self.yolo_model(frame)
    if result.confidence <= self.confidence_threshold:
      return None
    
    # 2단계: 프레임 버퍼에 추가
    self.frame_buffer.append(result)
    if len(self.frame_buffer) > self.buffer_size:
      self.frame_buffer.pop(0)
    
    # 3단계: 버퍼가 가득 찼나?
    if len(self.frame_buffer) < self.buffer_size:
      return None
    
    # 4단계: 3프레임 일관성 검증
    if not self.check_consistency():
      return None
    
    # 5단계: 위치 편차 검증
    if not self.check_position_stability():
      return None
    
    return self.frame_buffer[-1]  # 안전한 결과 반환
  
  def check_consistency(self):
    # 모든 프레임이 같은 객체를 감지했나?
    first_class = self.frame_buffer[0].class_id
    for result in self.frame_buffer[1:]:
      if result.class_id != first_class:
        return False
    return True
  
  def check_position_stability(self):
    # 위치 변화가 임계값 내인가?
    positions = [r.position for r in self.frame_buffer]
    center = positions[1]  # 중간 프레임 기준
    
    for pos in positions:
      dx = abs(pos[0] - center[0])
      dy = abs(pos[1] - center[1])
      if dx > self.max_position_deviation or \
         dy > self.max_position_deviation:
        return False
    return True
```

---

### ❌ 문제 2: 조명 변화로 인식 실패

**증상**:
- 조명이 변하면 YOLO 신뢰도 급락
- 회사 실내 다양한 조도에서 동작 불안정

**원인**:
- 전처리 없이 raw 이미지 바로 사용
- 이미지 평형화 부재

**해결**:
```python
class ImagePreprocessor:
  @staticmethod
  def preprocess(frame):
    # 1. 조도 정규화 (LAB 색공간)
    lab = cv2.cvtColor(frame, cv2.COLOR_BGR2LAB)
    l_channel = lab[:, :, 0]
    
    # 히스토그램 균등화
    l_equalized = cv2.equalizeHist(l_channel)
    lab[:, :, 0] = l_equalized
    frame = cv2.cvtColor(lab, cv2.COLOR_LAB2BGR)
    
    # 2. CLAHE (Contrast Limited Adaptive Histogram)
    clahe = cv2.createCLAHE(clipLimit=2.0, 
                            tileGridSize=(8, 8))
    frame_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    frame_gray = clahe.apply(frame_gray)
    
    # 3. 감마 보정 (밝기 조정)
    gamma = 0.9
    inv_gamma = 1.0 / gamma
    table = np.array([((i / 255.0) ** inv_gamma) * 255 
                      for i in range(256)], dtype="uint8")
    frame = cv2.LUT(frame, table)
    
    return frame
```

**테스트**:
```bash
# 다양한 조도에서 촬영한 이미지로 테스트
python test_lighting_robustness.py
# 신뢰도가 0.7 이상 유지되는지 확인
```

---

## Project 4: TIAGo 자율배달로봇

### ❌ 문제 1: Nav2 + Vision 동시 실행 시 FPS 저하

**증상**:
- 주행 중 예리한 방향 변화 불가능
- 1초 이상의 응답 지연

**원인**:
```
Nav2 (40% CPU) + Vision (35% CPU) + 기타 (30% CPU) = 105% CPU 오버헤드
→ CPU 절약을 위한 우선순위 필요
```

**해결 (리소스 관리)**:
```python
class ResourceManager:
  FEATURES = {
    'nav2': (1, True),           # 우선순위, 항상 필요
    'vision': (2, True),          
    'qr_detection': (3, False),
    'llm_inference': (4, False),
  }
  
  def __init__(self):
    self.cpu_threshold = {
      'high': 0.95,      # CPU >95%
      'medium': 0.85,    # CPU >85%
      'normal': 0.80,    # CPU <80%
    }
    self.active_features = set(self.FEATURES.keys())
  
  def monitor_and_adjust(self):
    cpu_usage = psutil.cpu_percent(interval=1)
    
    if cpu_usage > self.cpu_threshold['high']:
      # 긴급: Nav2만 유지
      self.disable_feature('llm_inference')
      self.disable_feature('vision')
      self.disable_feature('qr_detection')
    
    elif cpu_usage > self.cpu_threshold['medium']:
      # 높음: Vision 우선
      self.disable_feature('llm_inference')
      self.enable_feature('vision')
    
    elif cpu_usage < self.cpu_threshold['normal']:
      # 정상: 모두 활성화
      self.enable_feature('llm_inference')
      self.enable_feature('vision')
  
  def disable_feature(self, feature):
    if feature in self.active_features:
      rospy.set_param(f'/node_{feature}/enabled', False)
      self.active_features.remove(feature)
      rospy.logwarn(f"Disabled: {feature}")
  
  def enable_feature(self, feature):
    if feature not in self.active_features:
      rospy.set_param(f'/node_{feature}/enabled', True)
      self.active_features.add(feature)
      rospy.loginfo(f"Enabled: {feature}")
```

**테스트**:
```bash
# 모니터링
top -p $(pgrep -f ros)
# CPU 75% 이상에서 기능 비활성화 확인
```

---

### ❌ 문제 2: QR 방향 불안정

**증상**:
- QR 감지 후 로봇이 목표에서 180도 회전한 방향으로 이동
- 또는 반대 방향으로 회전

**원인**:
```python
# ❌ 방향 정보 무시
qr_center = detect_qr(frame)
goal.position = qr_center
goal.orientation = quaternion_from_euler(0, 0, 0)  # 고정!
```

**해결**:
```python
def detect_qr_with_orientation(frame):
  # OpenCV의 QRCodeDetector 사용
  qr_detector = cv2.QRCodeDetector()
  retval, decoded_info, points, _ = qr_detector.detectAndDecodeMulti(frame)
  
  if retval and len(points) > 0:
    qr_points = points[0]  # 첫 번째 QR의 네 꼭짓점
    
    # QR의 top-left → top-right 벡터로 방향 계산
    top_left = qr_points[0]
    top_right = qr_points[1]
    
    delta_x = top_right[0] - top_left[0]
    delta_y = top_right[1] - top_left[1]
    
    # 각도 계산 (라디안)
    qr_angle = np.arctan2(delta_y, delta_x)
    
    # 카메라 기준각 조정 (카메라 장착 위치에 따라)
    camera_offset = np.pi / 2  # 90도 회전
    final_angle = qr_angle + camera_offset
    
    return {
      'position': (qr_points.mean(axis=0)[0], 
                   qr_points.mean(axis=0)[1]),
      'orientation': final_angle,
      'confidence': 0.9
    }
  
  return None
```

**테스트**:
```bash
# QR 코드를 다양한 각도로 배치
# 로봇이 올바른 방향으로 이동하는지 확인
```

---

## 📊 공통 디버깅 팁

### 1. 로깅 활성화
```python
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)
logger.debug("State: %s, Value: %d", state, value)
```

### 2. ROS 토픽 모니터링
```bash
# 토픽 확인
ros2 topic list

# 토픽 내용 확인
ros2 topic echo /topic_name

# 토픽 주기 확인
ros2 topic hz /topic_name

# 토픽 대역폭 사용량
ros2 topic bw /topic_name
```

### 3. 성능 프로파일링
```python
import time
import cProfile

# 간단한 시간 측정
start = time.perf_counter()
expensive_operation()
elapsed = time.perf_counter() - start
print(f"Took {elapsed:.3f} seconds")

# 상세 프로파일링
cProfile.run('expensive_operation()')
```

### 4. 하드웨어 연결 확인
```bash
# USB 포트 확인
ls /dev/ttyUSB*
ls /dev/ttyACM*

# 직렬 통신 테스트
cat /dev/ttyUSB0

# 포트 권한 설정
sudo usermod -a -G dialout $USER
```

---

## 🆘 긴급 연락

문제 해결이 안 되면:
1. [ROS Discourse](https://discourse.ros.org/) 검색
2. [Stack Overflow](https://stackoverflow.com/) ROS 태그
3. GitHub Issues 검색
4. 팀 멘토에게 문의
5. 해당 프로젝트의 로그 & 상태 메시지 수집 후 보고

