# Vivado Project: UART, Sensor & Stopwatch Controller

이 프로젝트는 UART 통신, 초음파 센서(SR04), 온습도 센서(DHT11), 그리고 스톱워치 기능을 통합한 FPGA Verilog 설계입니다. Vivado 환경에서 개발되었으며, 스위치와 버튼을 통해 다양한 모드를 제어하고 결과를 FND(7-Segment) 디스플레이에 출력하거나 UART를 통해 PC로 전송할 수 있습니다.

## 주요 기능 (Features)
- **UART 통신**: 
  - ASCII 명령어를 수신하여 제어 (ex: `r`, `l`, `u`, `d`, `s` 등)
  - 연산 완료된 FND 데이터를 UART를 통해 PC로 송신
- **초음파 센서 (HC-SR04)**: 거리 측정 및 FND 출력
- **온습도 센서 (DHT11)**: 온도 및 습도 데이터 수집 및 FND 출력
- **스톱워치 (Stopwatch)**:
  - Run/Stop, Clear 기능
  - 시/분 모드 및 초/밀리초 모드
  - Up/Down 카운트 지원
- **FND 제어 (7-Segment)**: 스위치 입력에 따라 센서 데이터나 스톱워치 데이터를 선택적으로 디스플레이

## 입력 및 제어 방법
- **스위치 (SW[4:0])**:
  - `SW[4]`: 초음파 센서 모드 활성화
  - `SW[3]`: 온습도 센서 모드 활성화 (0: 온도, 1: 습도)
  - `SW[2]`: 스톱워치 시/분 or 초/밀리초 선택
  - `SW[1]`: 시계 / 스톱워치 모드 선택
  - `SW[0]`: 스톱워치 Up/Down 모드 선택
- **버튼 (BTN_R, BTN_L, BTN_U, BTN_D)**: 스톱워치 동작 및 기능 제어

## 프로젝트 구조
소스 코드는 `0221_yun_uart_sensor_stopwatch.srcs/sources_1/` 디렉토리 내에 위치하고 있습니다.
- `uart_project_top.v`: 최상위(Top) 모듈
- `uart_top.v`, `askii_decoder.v`, `ascii_sender.v`: UART 송수신 및 디코딩 모듈
- `top_stopwatch.v`, `controlunit.v`: 스톱워치 제어 모듈
- `sr04_top.v`: 초음파 센서 제어 모듈
- `dht11_controller.v`: 온습도 센서 제어 모듈
- `fnd_controller.v`: FND 디스플레이 제어 모듈
- `fifo.v`, `uart_fifo.v`: 데이터 버퍼용 FIFO 모듈

## 깃허브 업로드 안내
이 레포지토리는 소스 코드 위주로 관리하기 위해 `.gitignore`가 설정되어 있습니다. (Vivado의 `.cache`, `.sim`, `.runs` 등의 임시 파일은 제외됨)
