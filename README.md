# 🤖 ROKEY Bootcamp 5기 — 로봇 시스템 프로젝트 포트폴리오

> **배일훈 (Bae Ilhun)** | 두산 로보틱스 ROKEY 부트캠프 5기 (2025.07 – 2026.01)  
> Embedded & Robot Software Developer | ROS2 | Firmware | AI Integration  

[![ROS2](https://img.shields.io/badge/ROS2-Humble-22314E?logo=ros)](https://docs.ros.org/en/humble/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu)](https://ubuntu.com/)
[![C++](https://img.shields.io/badge/C++-17-00599C?logo=cplusplus)](https://isocpp.org/)
[![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📌 About This Repository

이 저장소는 **두산 로보틱스 ROKEY 부트캠프 5기** 과정에서 수행한  
**4개의 실제 로봇 시시템 프로젝트**를 정리한 포트폴리오입니다.

### 🎯 핵심 가치

각 프로젝트는 단순 "동작하는 코드"를 넘어  
**실제 환경에서 발생한 문제 → 원인 분석 → 구조적 해결**을 반복하며 진행했습니다.

| 역량 | 적용 |
|------|------|
| **문제 해결** | 상태 머신, 디바운싱, EEPROM 활용 |
| **시스템 설계** | ROS2 토픽/서비스, 계층화 아키텍처 |
| **AI 통합** | YOLO, Vision 파이프라인, 신뢰도 검증 |
| **임베디드 개발** | Arduino 펌웨어, 상태 관리, 실시간 제어 |
| **팀 협업** | GitHub, 명확한 인터페이스 정의 |

> **개발 철학**  
> _"동작하는 코드와 신뢰할 수 있는 시스템은 다르다"_

---

## 🗂️ 프로젝트 목록

4개의 프로젝트를 통해 **임베디드 제어 → ROS2 시스템 → AI 통합 → 자율주행**으로 점진적 확장했습니다.

| # | 프로젝트 | 핵심 기술 | 역할 | 상세 |
|---|----------|-----------|------|------|
| **1** | [🏭 컨베이어벨트 자동화](#-project-1-컨베이어벨트-자동화-공정-제어-시스템) | ROS2, Arduino, 상태머신, 동기화 | 30% | [📖 상세보기](project1_conveyor_belt/README.md) |
| **2** | [🤖 SLAM 자율주행 로봇](#-project-2-slam-기반-자율주행-로봇-시스템) | Arduino, 상태관리, EEPROM, 펌웨어 | 20% | [📖 상세보기](project2_slam_robot/README.md) |
| **3** | [🎯 멀티모달 LLM 협동로봇](#-project-3-멀티모달-llm-협동로봇-지능제어-시스템) | YOLO, Vision, 신뢰도 검증, 파이프라인 | 팀 | [📖 상세보기](project3_multimodal_llm/README.md) |
| **4** | [🚀 TIAGo 자율배달로봇](#-project-4-디지털트윈-기반-tiago-자율배달로봇) | Nav2, CPU 최적화, QR 인식, 시뮬레이션 | 팀 | [📖 상세보기](project4_tiago_delivery/README.md) |

---

## 📖 프로젝트 상세 정보

각 프로젝트의 **기술 스택, 문제 해결, 아키텍처**는 각 폴더의 README에서 확인할 수 있습니다:

- **🏭 [Project 1: 컨베이어벨트 자동화](project1_conveyor_belt/README.md)** 
  - ROS2 기반 로봇팔 연동 | 기여도 30% | Arduino 펌웨어 + ROS2 브릿지

- **🤖 [Project 2: SLAM 자율주행 로봇](project2_slam_robot/README.md)** 
  - MCU 펌웨어 상태 관리 | 기여도 20% | 그리퍼 제어 시스템

- **🎯 [Project 3: LLM 협동로봇](project3_multimodal_llm/README.md)** 
  - AI Vision 파이프라인 | 팀 프로젝트 | Webcam 인식 시스템

- **🚀 [Project 4: TIAGo 자율배달로봇](project4_tiago_delivery/README.md)** 
  - ROS2 Nav2 자율주행 | 팀 프로젝트 | 객체/QR 인식 통합

---

## 🛠️ 공통 기술 스택

### 💻 프로그래밍 언어
```
C++17        ROS2 시스템 및 고성능 연산 담당
Python 3.10  AI/Vision 파이프라인 구현
C (Arduino)  MCU 펌웨어 개발
```

### 🤖 로봇 프레임워크 & 플랫폼
```
ROS2 Humble      분산 로봇 시스템
ROS2 Nav2        자율주행 및 경로 계획
TurtleBot3       모바일 로봇 플랫폼
Doosan Robot     협동로봇 (Human-Robot Collaboration)
TIAGo            모바일 매니퓨레이터
```

### 🧠 AI & Vision
```
YOLO v8          실시간 객체 감지
OpenCV           이미지 전처리 및 처리
TensorFlow       신경망 추론
QR Code 인식     위치 마킹 및 네비게이션
```

### 📌 핵심 개발 역량

| 분야 | 기술 | 프로젝트 |
|------|------|---------|
| **임베디드 제어** | 상태머신, 디바운싱, EEPROM, 직렬통신 | 1, 2 |
| **시스템 설계** | ROS2 토픽/서비스, 아키텍처 설계 | 1, 2, 3, 4 |
| **Vision 통합** | YOLO, 신뢰도 검증, 파이프라인 | 3, 4 |
| **경로 계획** | Nav2, AMCL, 로컬 플래닝 (DWB) | 4 |
| **문제 해결** | 근본 원인 분석, 구조적 해결 | 모든 프로젝트 |
| **팀 협업** | GitHub, 명확한 인터페이스, 문서화 | 모든 프로젝트 |

---

## 📚 주요 학습 내용

### 1️⃣ 상태 머신의 중요성
- 복잡한 제어 흐름도 명확한 상태 정의로 관리 가능
- 예측 불가능한 오동작을 구조적으로 방지

### 2️⃣ 임베디드 ↔ 고수준 시스템 간 인터페이스
- 하위 하드웨어 제어와 상위 애플리케이션 분리
- ROS2 토픽/서비스로 추상화하여 확장성 확보

### 3️⃣ AI 신뢰도의 현실성
- 100% 정확한 AI는 존재하지 않음
- 낮은 신뢰도는 필터링, 다단계 검증으로 보완

### 4️⃣ 시스템 성능 최적화
- 리소스 병목 파악 및 우선순위 기반 분배
- 모든 기능을 동시에 구동 ≠ 안정적인 시스템

### 5️⃣ 디버깅과 안정성
- 한 번의 좋은 설계 >> 여러 번의 패치
- 로깅, 타임스탬프, 상태 추적으로 버그 조기 발견

---

## 📁 저장소 구조

```
ROKEY_5/
├── README.md                          # 이 파일 (포트폴리오 개요)
│
├── project1_conveyor_belt/            # 🏭 컨베이어벨트 자동화
│   ├── README.md                      # 상세 설명서
│   ├── firmware/                      # Arduino 펌웨어
│   ├── src/                           # ROS2 노드
│   └── config/
│
├── project2_slam_robot/               # 🤖 SLAM 자율주행 로봇
│   ├── README.md
│   ├── firmware/
│   ├── ros_nodes/
│   └── config/
│
├── project3_multimodal_llm/           # 🎯 LLM 협동로봇
│   ├── README.md
│   ├── src/
│   ├── config/
│   └── launch/
│
├── project4_tiago_delivery/           # 🚀 TIAGo 자율배달
│   ├── README.md
│   ├── launch/
│   ├── src/
│   ├── config/
│   └── msg/
│
├── docs/                              # 📚 추가 문서
│   ├── TECH_STACK.md                  # 기술 스택 상세
│   ├── LESSONS_LEARNED.md             # 배운 내용
│   ├── ARCHITECTURE.md                # 시스템 아키텍처
│   └── TROUBLESHOOTING.md             # 문제 해결 가이드
│
├── .gitignore                         # Git 무시 파일
└── LICENSE                            # MIT 라이선스
```

---

## 🚀 빠른 시작

### 각 프로젝트 상세 보기
모든 프로젝트는 **개별 README**를 가지고 있습니다.

- 🏭 [Project 1: 컨베이어벨트](project1_conveyor_belt/README.md)
- 🤖 [Project 2: SLAM 로봇](project2_slam_robot/README.md)
- 🎯 [Project 3: LLM 협동로봇](project3_multimodal_llm/README.md)
- 🚀 [Project 4: TIAGo 배달로봇](project4_tiago_delivery/README.md)

### 기술 스택 상세
- 📖 [기술 스택 가이드](docs/TECH_STACK.md)

### 문제 해결 기록
- 🔧 [발생 문제 및 해결 방법](docs/TROUBLESHOOTING.md)

---

## 👤 Contact & Links

| | |
|---|---|
| **이름** | 배일훈 (Bae Ilhun) |
| **이메일** | bae1hon@gmail.com |
| **GitHub** | [github.com/Baeilhoon](https://github.com/Baeilhoon) |
| **전화** | 010-2089-4401 |
| **교육 과정** | 두산 로보틱스 ROKEY Boot Camp 5기 (2025.07 – 2026.01) |

---

## 📜 라이선스

이 프로젝트는 **MIT License** 하에 공개됩니다.  
자유롭게 사용, 수정, 배포할 수 있습니다.

[LICENSE 파일 보기](LICENSE)

---

## 🙏 감사의 말

이 프로젝트들은 많은 사람들의 지원과 피드백 덕분에 완성되었습니다:

- 두산 로보틱스 ROKEY 부트캠프 강사진
- 협력 팀원들 (프로젝트 3, 4)
- 멘토 및 리뷰어

---

<div align="center">
  <h3>🎓 두산 로보틱스 ROKEY Boot Camp 5기 수료</h3>
  <p><strong>2026.01.09</strong> 수료 · 임베디드 & 로봇 소프트웨어 개발자</p>
  <br/>
  
  **"동작하는 코드와 신뢰할 수 있는 시스템은 다르다"**
  
  이 철학으로 모든 프로젝트를 진행했습니다.
</div>
