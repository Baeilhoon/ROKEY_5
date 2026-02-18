# 📚 Documentation Index

이 디렉토리의 모든 문서에 대한 인덱스입니다.

## 📖 포트폴리오 개요
- **[메인 README](../README.md)** - 전체 프로젝트 개요, 4개 프로젝트 요약

---

## 🚀 프로젝트별 상세 설명

### Project 1: 컨베이어벨트 자동화
- **[상세 설명서](../project1_conveyor_belt/README.md)**
  - 개요, 기술 스택, 문제 해결
  - 아키텍처 및 상태 머신 설명

### Project 2: SLAM 자율주행 로봇
- **[상세 설명서](../project2_slam_robot/README.md)**
  - EEPROM 활용, 상태 관리
  - 펌웨어 설계 및 제어 흐름

### Project 3: LLM 협동로봇
- **[상세 설명서](../project3_multimodal_llm/README.md)**
  - Vision 파이프라인, YOLO 통합
  - 신뢰도 검증 및 필터링

### Project 4: TIAGo 배달로봇
- **[상세 설명서](../project4_tiago_delivery/README.md)**
  - Nav2 구현, QR 인식
  - CPU 최적화 및 우선순위 관리

---

## 🛠️ 기술 가이드

### [기술 스택 (TECH_STACK.md)](TECH_STACK.md)
프로젝트에서 사용한 모든 기술 상세 설명:
- 프로그래밍 언어 (C++, Python, C)
- ROS2 & Navigation
- AI/Vision (YOLO, OpenCV)
- 로봇 플랫폼 (Arduino, TurtleBot3, TIAGo, Doosan)
- 참고 자료 모음

---

## 🎓 학습 내용

### [배운 내용 (LESSONS_LEARNED.md)](LESSONS_LEARNED.md)
7가지 핵심 학습 내용:
1. **상태 머신의 중요성**
   - FSM 패턴, 상태 다이어그램
   
2. **계층화 설계**
   - PC ↔ 로봇 ↔ Arduino 계층 분리
   
3. **AI 신뢰도 관리**
   - 다단계 검증, 신뢰도 필터링
   
4. **고성능 시스템 설계**
   - 우선순위 기반 리소스 관리
   
5. **디버깅 전략**
   - 로깅, 모니터링, 성능 프로파일링
   
6. **임베디드 메모리 관리**
   - EEPROM 활용, RAM vs 비휘발성 메모리
   
7. **팀 협업 원칙**
   - 계약 기반 설계, 명확한 인터페이스

---

## 🏗️ 시스템 아키텍처

### [아키텍처 (ARCHITECTURE.md)](ARCHITECTURE.md)
각 프로젝트의 시스템 설계:
- 아키텍처 다이어그램 (ASCII art)
- 메시지 흐름 설명
- 제어 흐름 시각화
- 설계 원칙 및 확장 가능성

---

## 🔧 문제 해결

### [Troubleshooting (TROUBLESHOOTING.md)](TROUBLESHOOTING.md)
프로젝트별 주요 문제와 해결책:

**Project 1 - 컨베이어벨트**:
- IR 센서 노이즈 제거 (디바운싱)
- 동기화 메커니즘
- ROS2-Arduino 통신 지연

**Project 2 - SLAM 로봇**:
- 상태 전환 중 reset() 재호출
- 필드 관리 부족
- 전원 재시작 시 상태 유실

**Project 3 - LLM 협동로봇**:
- 인식 오류 처리
- 조명 변화 대응

**Project 4 - TIAGo 배달로봇**:
- FPS 저하 및 CPU 최적화
- QR 방향 불안정

---

## 🔍 빠른 검색

| 주제 | 문서 | 섹션 |
|------|------|------|
| 상태 머신 | LESSONS_LEARNED | 1️⃣ |
| ROS2 | TECH_STACK | ROS2 & 로봇 프레임워크 |
| YOLO | TECH_STACK | AI & Vision |
| 디버깅 | TROUBLESHOOTING | 공통 팁 |
| 아키텍처 | ARCHITECTURE | 전체 프로젝트 연계도 |
| EEPROM | LESSONS_LEARNED | 6️⃣ |
| 로깅 | LESSONS_LEARNED | 5️⃣ |
| Nav2 | TECH_STACK | ROS2 Nav2 |

---

## 📋 문서 작성 규칙

모든 문서는 다음 구조를 따릅니다:

```
# 제목

> 간단한 설명

---

## 섹션 1

상세 내용

### 소섹션
더 구체적인 내용

---

## 섹션 2
...
```

---

## 🔗 외부 참고 자료

| 분야 | 링크 |
|------|------|
| **ROS2** | https://docs.ros.org/en/humble/ |
| **Nav2** | https://navigation.ros.org/ |
| **YOLO** | https://docs.ultralytics.com/ |
| **OpenCV** | https://docs.opencv.org/ |
| **Arduino** | https://www.arduino.cc/reference/ |
| **C++** | https://en.cppreference.com/ |
| **Python** | https://www.python.org/doc/ |
| **ROS Discourse** | https://discourse.ros.org/ |

---

## 📝 마지막 업데이트

- **생성일**: 2026년 2월 19일
- **대상 보급처**: GitHub ROKEY_5 저장소
- **용도**: 포트폴리오 및 교육 자료

