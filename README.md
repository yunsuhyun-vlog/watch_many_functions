# UART 기반 센서 2종 및 시계/스톱워치 제어 컨트롤러 (UART-Sensor-Stopwatch-Controller)

본 프로젝트는 FPGA(Vivado) 환경에서 **UART 통신, 초음파 센서(HC-SR04), 온습도 센서(DHT11), 그리고 시계/스톱워치 모듈**을 통합하여 제어하는 시스템을 RTL(Verilog)로 직접 설계한 팀 프로젝트입니다.

## 🎯 수행 목표 및 담당 역할
- **수행 기간**: 2026-02-11 ~ 2026-02-18
- **수행 목표**: 버튼 및 스위치, UART 통신을 통한 시간/센서 기능 제어 및 상태 읽기 로직 설계
- **담당 역할 (팀 프로젝트)**: Design Module (SR04 Controller, DHT11 Controller 설계)
- **사용 기술**: Synchronous FIFO 설계, 2-stage Synchronizer, UART 16x Oversampling, Button Debounce 회로 설계

---

## 🏗️ 전체 아키텍처 및 핵심 모듈 (Overall Architecture)

### 1. 센서 제어 모듈
- **SR04 Sensor (거리 감지)**
  - Pulse width: Trigger 신호를 송신하고 물체에 반사되어 돌아오는 Echo 신호 수신
  - Distance: 1us 주기의 카운터를 생성하여 거리를 계산 (`거리(cm) = 측정 시간(us) / 58`)
- **DHT11 Sensor (온습도 감지)**
  - Data Format (총 40-bit): `습도 16-bits` + `온도 16-bits` + `Checksum 8-bits`(데이터 무결성 검증용)

### 2. UART 통신 및 시계 제어 모듈
- **UART 통신 (9600 bps)**
  - `baud_tick`: 9600bps 통신 속도에 맞춘 Tick 주기 생성
  - `uart_rx` / `uart_tx`: 외부 데이터 샘플링(수신) 및 데이터 드라이빙(송신)
- **ASCII Decoder & Stopwatch**
  - PC에서 입력된 문자열(ASCII)을 디코딩하여 하드웨어 버튼 신호로 변환
  - 입력된 버튼과 스위치 신호에 따라 시계/스톱워치의 **시작, 정지, 모드 전환, 시간 설정** 수행

---

## 🛠️ 주요 하드웨어 설계 기술 (Tech Skills)

### 1. Synchronous FIFO (데이터 병목 해소)
- **상태 제어 신호**: Full / Empty 플래그를 생성하여 송수신 측의 읽기/쓰기 동작을 제어
- **사용 이점**:
  1. 기존 데이터가 덮어씌워지는 데이터 유실(Data Loss) 완벽 방지
  2. FPGA 메인 로직과 외부 PC 사이의 **통신 속도 차이로 인해 발생하는 병목(Bottleneck) 현상 해소**

### 2. 2-Stage Synchronizer (안정성 확보)
- **구조**: 2-bit Shift Register 형태의 동기화 회로
- **설계 과정**: 비동기적으로 들어오는 DHT11 센서 신호 입력단에 싱크로나이저 설계
- **사용 이점**: 외부 비동기 신호로 인해 발생할 수 있는 **Setup Time / Hold Time 위반(Metastability)** 방지

### 3. UART 16x Oversampling (통신 신뢰성 향상)
- **구조**: 비동기 통신(UART)의 Frame을 안정적으로 수신하기 위한 기법
- **설계 과정**: Tick Counter를 설계하여 1 baud당 16번 샘플링(16x Oversampling)을 수행하고 비트의 정중앙을 타겟팅하여 데이터를 읽음
- **사용 이점**:
  1. 에지(Edge) 부근에서 발생할 수 있는 Metastability 문제 사전 예방
  2. 타이밍 오차 누적으로 인한 **샘플링 시점 밀림 현상을 방지**하여 안전 마진 확보

### 4. Button Debounce (노이즈 필터링)
- **구조**: 8-bit Shift Register 기반
- **설계 과정**: 100kHz 클럭 주기마다 현재 버튼 상태를 최상위 비트에 넣고 오른쪽으로 Shift하며 상태 지속 여부 판별
- **사용 이점**: 사용자가 물리적인 버튼을 누를 때 발생하는 **채터링/바운싱(Bouncing) 노이즈 제거**

---

## 📊 입출력 인터페이스 및 결과 (Result)

### 입력 및 출력 장치 (Basys-3 보드)
- **버튼 (Button)**: 스톱워치 Run/Stop/Clear, 시계 초/분/시 설정
- **스위치 (Switch)**: Watch / Stopwatch 모드 전환, 스톱워치 Up/Down Count 모드 설정
- **출력 장치**: 4-Digit 7-Segment (FND Display)

### 시뮬레이션 및 동작 결과
- DHT11 센서와 SR04 초음파 센서 모듈의 타이밍 시뮬레이션 검증 완료
- **실제 측정 데이터**: 강의실 내부 온도 `13.3℃`, 습도 `29.01%`, 물체와의 거리 `51cm` 정상 측정 확인

---

## 📂 프로젝트 폴더 구조
소스 코드는 `0221_yun_uart_sensor_stopwatch.srcs/sources_1/` 내에 위치합니다.
- `uart_project_top.v`: 전체 시스템 최상위(Top) 모듈
- `uart_top.v`, `askii_decoder.v`: UART 송수신 및 ASCII 디코딩
- `top_stopwatch.v`, `controlunit.v`: 시계 및 스톱워치 제어 모듈
- `sr04_top.v`, `dht11_controller.v`: 센서 컨트롤러 모듈
- `fifo.v`, `uart_fifo.v`: 동기식 FIFO 모듈
- `fnd_controller.v`: 7-Segment 출력 제어
