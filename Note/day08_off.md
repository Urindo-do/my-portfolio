# 🦾 Physical AI 부트캠프 — Day 3: 게임 제작 마무리 & 4축 로봇팔 서버화

> 팀 모노리스 / 오프라인 수업 정리
> 주제: OLED 슈팅 게임 마무리 → 4축 로봇팔 조립(PCA9685) → Web Serial로 조종 → ESP32를 AP 서버로 만들어 폰으로 조종
> 수업은 모션 티칭(매크로)까지 진도가 나갔지만, 이 문서는 내가 직접 확인까지 끝낸 **ESP32 서버화 단계**까지만 기록한다.

---

## 목차

1. [오늘 활동 개요](#1-오늘-활동-개요)
2. [활동 ① 게임 제작 마무리](#2-활동--게임-제작-마무리)
3. [활동 ② 4축 로봇팔 조립 — 90도 기준 정렬](#3-활동--4축-로봇팔-조립--90도-기준-정렬)
4. [활동 ③ 웹 슬라이더로 로봇팔 조종 (Web Serial + JSON)](#4-활동--웹-슬라이더로-로봇팔-조종-web-serial--json)
5. [활동 ④ ESP32를 서버로 — 폰으로 조종 (AP 모드)](#5-활동--esp32를-서버로--폰으로-조종-ap-모드)
6. [핵심 개념: PCA9685를 쓰는 이유](#6-핵심-개념-pca9685를-쓰는-이유)
7. [트러블슈팅](#7-트러블슈팅)
8. [오늘 느낀 점](#8-오늘-느낀-점)
9. [오늘의 실습 사진 · 영상](#9-오늘의-실습-사진--영상)

---

## 1. 오늘 활동 개요

Day 2가 아날로그 입력(조도센서·조이스틱)을 익히는 시간이었다면, 오늘은 그 입력들을 실제 **결과물 두 개**로 완성하는 날이었다.

- 활동 ①: 조이스틱 2개 + OLED + 부저로 만들던 우주선 슈팅 게임 마무리
- 활동 ②: 4축 로봇팔 하드웨어 조립 (서보 4개 + PCA9685)
- 활동 ③: 웹페이지 슬라이더로 로봇팔을 USB(Web Serial)로 조종
- 활동 ④: ESP32가 스스로 Wi-Fi 공유기(AP)가 되어, 케이블 없이 폰으로 로봇팔 조종

---

## 2. 활동 ① 게임 제작 마무리

기존 우주선 슈팅 게임(조이스틱 1개 + OLED + 부저)을 두 방향으로 확장했다.

- **2인 대전 버전**: 조이스틱 2개로 좌우 영역을 나눠(왼쪽 P1 / 오른쪽 P2) 서로 총알을 쏘는 대전. `Player`, `Bullet` 구조체로 두 플레이어를 동일한 함수로 처리했다.
- **1인 AI 대전 버전**: 조이스틱 1개만 남기고, 상대(P2) 자리를 AI 로직으로 교체. AI는 "다가오는 총알 회피 → 없으면 플레이어 Y좌표 추적 → 가끔 랜덤 방향 전환" 3단계 우선순위로 움직이고, 라운드를 이길수록 속도·연사가 빨라지는 난이도 시스템을 넣었다.

### 관찰

- 총알을 배열(`Bullet bullets[MAX_BULLETS]`)로 관리하면 발사 개수 제한 없이 여러 발을 동시에 다룰 수 있다. `active` 플래그로 빈 슬롯을 재사용하는 패턴을 처음 써봤다.
- `delay()` 대신 `millis()` 기반 쿨다운을 쓰면 연사 간격을 조절하면서도 게임이 멈추지 않는다. 효과음(`tone()`)도 지속시간을 짧게 주면 논블로킹으로 동작한다.

---

## 3. 활동 ② 4축 로봇팔 조립 — 90도 기준 정렬

### 조립의 대원칙

서보모터는 0~180도로 움직이는데, **90도가 물리적으로 정중앙에 오도록 조립**해야 양쪽으로 균등하게 움직일 수 있다. 90도가 중앙에서 벗어난 채 조립하면 한쪽 가동범위가 잘려서 나중에 매우 귀찮아진다.

- 순서: 먼저 코드로 서보를 90도로 보내놓고 → 그 상태에서 팔 부품을 원하는 방향으로 끼운다. (부품을 끼우는 각도가 "90도 = 이 자세"를 정의한다)
- 예외: 집게(그리퍼)는 **살짝 벌어진 상태**를 90도로 잡는 게 좋다고 배웠다.
- 조립 치수 메모: 12 / 16 / 20 / 25 / 30

### 배선

| PCA9685 핀 | 연결 대상 | 비고 |
| --- | --- | --- |
| VCC (로직 전원) | ESP32 3.3V | 칩 구동용, 서보 전원 아님 |
| GND | ESP32 GND | 공통 그라운드 |
| SDA (데이터) | GPIO 8 | I2C |
| SCL (클럭) | GPIO 9 | I2C (OLED도 같은 선 공유) |
| V+ / GND (터미널 블록) | 외부 5V 2~3A | ESP32 3.3V에 연결 금지 |
| 서보 4개 | 채널 0~3 | CH0 밑동 · CH1 어깨 · CH2 팔꿈치 · CH3 집게 |

필요 라이브러리: `Adafruit PWM Servo Driver Library`, `ArduinoJson`

### 코드 (전체 — 4축 90도 초기화)

```cpp
#include <Wire.h>
#include <Adafruit_PWMServoDriver.h>

#define I2C_SDA 8
#define I2C_SCL 9

Adafruit_PWMServoDriver pwm = Adafruit_PWMServoDriver();

#define SERVOMIN 150 // 0도 매핑 펄스 (12비트 값)
#define SERVOMAX 600 // 180도 매핑 펄스 (12비트 값)

void setServoAngle(int channel, int angle) {
  int pulse = map(angle, 0, 180, SERVOMIN, SERVOMAX);
    pwm.setPWM(channel, 0, pulse);
    }

    void setup() {
      Serial.begin(115200);
        delay(2000);
          Wire.begin(I2C_SDA, I2C_SCL);
            pwm.begin();
              pwm.setOscillatorFrequency(27000000);
                pwm.setPWMFreq(50); // 서보모터는 50Hz 주기
                  delay(50);
                    for (int ch = 0; ch < 4; ch++) setServoAngle(ch, 90);
                      Serial.println("모든 모터(CH 0~3) 90도 전송 완료!");
                      }

                      void loop() {
                        for (int ch = 0; ch < 4; ch++) setServoAngle(ch, 90); // 90도 자세 주기적 유지
                          delay(1000);
                          }
                          ```

                          ### 관찰

                          - 서보는 위치 값을 밖으로 돌려주지 않는 **명령만 받는 단방향 부품**이다. 90도가 진짜 중앙인지 확인하려면 0→90→180 스윕을 눈으로 보고 양쪽 회전량이 비슷한지 비교하는 방법뿐이다.
                          - 서보가 이미 90도 근처에 있으면 계속 90도만 보내도 거의 안 움직이고 "지잉" 소리만 난다. 이는 위치를 유지하려 힘을 주는 정상 동작이다.

                          ---

                          ## 4. 활동 ③ 웹 슬라이더로 로봇팔 조종 (Web Serial + JSON)

                          ### 구조

                          브라우저 슬라이더 → JSON 한 줄 생성 → USB(Web Serial) → ESP32가 파싱 → 서보 4개 회전

                          프로토콜은 딱 한 줄: `{"s0":90,"s1":110,"s2":60,"s3":45}`

                          - JSON으로 네 값을 한 줄에 묶는 이유: 각도를 하나씩 보내면 관절이 따로 움직여 어색하다. 한 줄로 보내면 보드가 한 번에 받아 동시에 반영하고, 사람이 눈으로 읽을 수 있어 디버깅도 쉽다.
                          - 끝에 `\n`을 붙이는 이유: 보드 입장에선 글자가 계속 흘러들어올 뿐이라, "어디까지가 한 줄인지" 알려주는 표시가 엔터다.
                          - Web Serial API는 브라우저가 USB 시리얼 포트에 직접 연결하는 기능이다. 설치할 프로그램도 서버도 필요 없고, Chrome·Edge에서만 동작한다.

                          ### 코드 (아두이노 쪽 — JSON 수신)

                          ```cpp
                          #include <Adafruit_PWMServoDriver.h>
                          #include <ArduinoJson.h>
                          #include <Wire.h>

                          Adafruit_PWMServoDriver pwm = Adafruit_PWMServoDriver();
                          #define SERVOMIN 150
                          #define SERVOMAX 600

                          // 초기각도는 반드시 전부 90! (로봇팔은 모든 모터가 90도인 상태로 조립되어 있음)
                          int servoAngles[4] = {90, 90, 90, 90};

                          #define BASE_CH 0 // 밑동(좌우 회전) = PCA9685 CH 0
                          void setServoAngle(int channel, int angle) {
                            if (channel == BASE_CH) angle = 180 - angle; // 밑동만 좌우 반전 (조립 방향 때문)
                              int pulse = map(angle, 0, 180, SERVOMIN, SERVOMAX);
                                pwm.setPWM(channel, 0, pulse);
                                }

                                void setup() {
                                  Serial.begin(115200);
                                    delay(1500);
                                      Wire.begin(8, 9);
                                        pwm.begin();
                                          pwm.setOscillatorFrequency(27000000);
                                            pwm.setPWMFreq(50);
                                              delay(10);
                                                for (int i = 0; i < 4; i++) setServoAngle(i, servoAngles[i]);
                                                }

                                                void loop() {
                                                  if (Serial.available() > 0) {
                                                      StaticJsonDocument<200> doc;
                                                          DeserializationError error = deserializeJson(doc, Serial);
                                                              if (!error) {
                                                                    if (doc.containsKey("s0")) servoAngles[0] = doc["s0"];
                                                                          if (doc.containsKey("s1")) servoAngles[1] = doc["s1"];
                                                                                if (doc.containsKey("s2")) servoAngles[2] = doc["s2"];
                                                                                      if (doc.containsKey("s3")) servoAngles[3] = doc["s3"];
                                                                                            for (int i = 0; i < 4; i++) {
                                                                                                    servoAngles[i] = constrain(servoAngles[i], 0, 180);
                                                                                                            setServoAngle(i, servoAngles[i]);
                                                                                                                  }
                                                                                                                      } else {
                                                                                                                            while (Serial.available() > 0) Serial.read(); // 깨진 데이터 버리기
                                                                                                                                }
                                                                                                                                  }
                                                                                                                                  }
                                                                                                                                  ```

                                                                                                                                  ### 관찰

                                                                                                                                  - `ArduinoJson`의 `deserializeJson()`으로 한 줄을 파싱하고, `containsKey()`로 있는 값만 갱신한다.
                                                                                                                                  - `constrain(0, 180)`으로 가동범위 밖 값을 잘라내야 기구부에 무리가 가지 않는다.
                                                                                                                                  - 밑동(CH0)은 서보 조립 방향 때문에 `angle = 180 - angle`로 좌우를 반전시켜야 슬라이더 방향과 실제 회전 방향이 일치했다.

                                                                                                                                  ---

                                                                                                                                  ## 5. 활동 ④ ESP32를 서버로 — 폰으로 조종 (AP 모드)

                                                                                                                                  ### 웹 슬라이더(USB)와 무엇이 다른가

                                                                                                                                  | 구분 | 웹 슬라이더(USB) | ESP32 서버(AP 모드) |
                                                                                                                                  | --- | --- | --- |
                                                                                                                                  | 웹페이지 위치 | 내 PC | 보드 안 (문자열로 저장) |
                                                                                                                                  | 전달 통로 | USB 시리얼 | Wi-Fi (HTTP) |
                                                                                                                                  | 데이터 형식 | JSON 한 줄 | URL 파라미터 `/set?ch=0&ang=120` |
                                                                                                                                  | 필요한 것 | PC + 케이블 | 폰만 (인터넷도 불필요) |

                                                                                                                                  ### 오늘 이해한 핵심 세 가지

                                                                                                                                  1. **USB는 이제 전원 공급용이다.** 데이터 송수신은 ESP32가 Wi-Fi로 직접 하고, 노트북은 전원(POWER) 역할만 한다. 보조배터리로 바꾸면 컴퓨터 자체가 필요 없다.
                                                                                                                                  2. **ESP32가 스스로 공유기가 된다(AP 모드).** `WiFi.softAP()` 한 줄로 집 공유기·인터넷과 무관한 독립 네트워크가 생긴다. 폰 Wi-Fi 목록에 `RobotArm-XXXX` 같은 이름이 뜨고, 접속하면 `http://192.168.4.1`이 보드 자신의 주소다. 보드마다 이름이 다른 이유는 MAC 주소 **뒤 2바이트**를 이름에 붙이기 때문(앞 3바이트는 제조사 공통이라 쓰면 안 됨).
                                                                                                                                  3. **홈페이지를 보드가 "기억"한다.** HTML 전체가 `const char PAGE_HTML[] PROGMEM = R"HTML(...)HTML";` 형태로 코드 안에 하나의 문자열 상수로 박혀 있고, 컴파일하면 플래시 메모리에 저장된다. 접속하면 `handleRoot()`가 이 문자열을 그대로 전송하고, 브라우저는 파일인지 문자열인지 구분하지 못하고 그냥 렌더링한다.

                                                                                                                                  ### 코드 (핵심 부분 — Wi-Fi 이름 생성 · 라우팅)

                                                                                                                                  ```cpp
                                                                                                                                  #include <WiFi.h>
                                                                                                                                  #include <WebServer.h>
                                                                                                                                  #include <DNSServer.h>

                                                                                                                                  WebServer server(80);
                                                                                                                                  const char *AP_PASSWORD = "12345678";
                                                                                                                                  String apSSID;

                                                                                                                                  void setup() {
                                                                                                                                    // ... 서보·OLED 초기화 ...

                                                                                                                                      WiFi.mode(WIFI_AP);
                                                                                                                                        uint8_t mac[6];
                                                                                                                                          WiFi.softAPmacAddress(mac);
                                                                                                                                            char suffix[5];
                                                                                                                                              sprintf(suffix, "%02X%02X", mac[4], mac[5]); // MAC 뒤 2바이트만 사용
                                                                                                                                                apSSID = "RobotArm-" + String(suffix);
                                                                                                                                                  WiFi.softAP(apSSID.c_str(), AP_PASSWORD); // 이 한 줄로 공유기가 됨

                                                                                                                                                    server.on("/", handleRoot);       // 컨트롤 페이지 전송
                                                                                                                                                      server.on("/set", handleSet);     // /set?ch=0&ang=120 -> 서보 각도 설정
                                                                                                                                                        server.on("/state", handleState); // 현재 4축 각도를 JSON으로
                                                                                                                                                          server.on("/home", handleHome);   // 전부 90도로 부드럽게 복귀
                                                                                                                                                            server.begin();
                                                                                                                                                            }

                                                                                                                                                            void loop() {
                                                                                                                                                              server.handleClient();
                                                                                                                                                              }
                                                                                                                                                              ```

                                                                                                                                                              ### 관찰

                                                                                                                                                              - 요청 라우팅: `/` 컨트롤 페이지 · `/set` 서보 제어 · `/state` 현재 각도 조회 · `/home` 90도 복귀.
                                                                                                                                                              - 슬라이더를 빠르게 흔들면 요청이 폭주하므로, 웹페이지 쪽에서 60ms에 한 번만 `fetch()`를 보내도록 제한했다.
                                                                                                                                                              - `moveSmooth()`는 목표 자세까지 20ms 단위로 잘게 쪼개 부드럽게 이동시킨다. `Pose` 구조체로 "자세 한 벌"을 표현하는 방식을 처음 써봤다.

                                                                                                                                                              ---

                                                                                                                                                              ## 6. 핵심 개념: PCA9685를 쓰는 이유

                                                                                                                                                              ESP32에 서보 4개를 직접 달지 않는 이유는 **전원 분리**다.

                                                                                                                                                              - 서보모터는 움직일 때 개당 수백 mA를 먹는다. ESP32의 3.3V 핀으로 4개를 감당하면 보드가 리셋되거나 파손될 수 있다.
                                                                                                                                                              - PCA9685는 서보 전원(V+)을 **외부 5V 전원에서 따로** 받아 전원 문제를 분리하고, ESP32와는 **통신선 2가닥(SDA/SCL, I2C)** 만으로 최대 16채널 서보를 제어한다.
                                                                                                                                                              - 요약: 전원은 분리, 제어는 I2C 두 가닥.

                                                                                                                                                              ---

                                                                                                                                                              ## 7. 트러블슈팅

                                                                                                                                                              | 증상 | 원인 | 해결 |
                                                                                                                                                              | --- | --- | --- |
                                                                                                                                                              | `Failed to open serial port` | 아두이노 IDE 시리얼 모니터가 COM 포트를 점유 — 포트는 한 번에 한 프로그램만 열 수 있다 | 시리얼 모니터를 닫고 재연결. 웹페이지와 시리얼 모니터는 동시 사용 불가 |
                                                                                                                                                              | 서보에서 계속 "지지직" 소리 | 서보 불량 + 기구부가 꽉 물려 스톨(stall) 상태 | 서보 교체, 부품 체결을 살짝 느슨하게 조정해서 해결 |

                                                                                                                                                              수업에서 배운 원칙: 서보는 0~180 명령을 다 받지만 **기구부의 물리 한계는 훨씬 좁다**. 한계에 닿았는데 명령이 계속 들어가면 멈춘 채 힘만 가하는 스톨 상태가 되어 발열·고장으로 이어진다. 처음 움직일 땐 5~10도씩 조금씩 탐색하고, "지잉—" 소리가 나면 즉시 반대 방향으로 되돌려야 한다.

                                                                                                                                                              ---

                                                                                                                                                              ## 8. 오늘 느낀 점

                                                                                                                                                              - 하드웨어는 결국 **90도 기준 조립**이 전부였다. 코드로 90도를 먼저 만들고, 그 상태에서 부품을 끼우는 순서를 지켜야 한다는 걸 몸으로 배웠다.
                                                                                                                                                              - 통신은 결국 "약속(프로토콜)"이 핵심이다. JSON 한 줄 + 개행 문자, 또는 URL 파라미터처럼 단순한 형식일수록 디버깅이 쉬웠다.
                                                                                                                                                              - COM 포트는 한 번에 한 프로그램만 쓸 수 있다는 걸 에러를 직접 겪으며 이해했다. 시리얼 모니터와 웹페이지는 양자택일이다.
                                                                                                                                                              - ESP32 하나로 공유기도 되고 웹서버도 될 수 있다는 게 오늘 가장 신기했던 부분. 웹페이지가 파일이 아니라 코드 속 문자열로 보드 플래시에 산다는 개념이 새로웠다.
                                                                                                                                                              - 다음 수업 진도(모션 티칭/매크로)는 나중에 다시 정리할 것.

                                                                                                                                                              ---

                                                                                                                                                              ## 9. 오늘의 실습 사진 · 영상

                                                                                                                                                              ### PCA9685 + 서보 배선
                                                                                                                                                              ![PCA9685 서보 드라이버와 서보모터 배선 모습](images/day08_pca9685_wiring.jpg)

                                                                                                                                                              ### 4축 로봇팔 조립 (3D 프린팅)
                                                                                                                                                              ![90도 기준으로 조립 중인 4축 로봇팔](images/day08_arm_assembly.jpg)

                                                                                                                                                              ### ESP32 AP 모드 Wi-Fi 접속 정보 (시리얼 모니터)
                                                                                                                                                              ![시리얼 모니터에 표시된 RobotArm Wi-Fi 이름과 접속 주소](images/day08_wifi_serial.jpg)

                                                                                                                                                              > 📹 오늘은 영상도 함께 찍었다. 유튜브에 업로드 후 아래에 링크를 채운다.
                                                                                                                                                              > `[![로봇팔 웹 슬라이더 조종](https://img.youtube.com/vi/VIDEO_ID/0.jpg)](https://youtu.be/VIDEO_ID)`
                                                                                                                                                              >
                                                                                                                                                              > - [ ] 영상 1 — 웹 슬라이더로 로봇팔 조종 (링크 추가 예정)
                                                                                                                                                              > - [ ] 영상 2 — 폰 Wi-Fi 접속 조종 (링크 추가 예정)
                                                                                                                                                              >
                                                                                                                                                              > 이미지 3장은 파일명(`day08_pca9685_wiring.jpg`, `day08_arm_assembly.jpg`, `day08_wifi_serial.jpg`)에 맞춰 `Note/images/` 폴더에 드래그 업로드하면 위 링크가 바로 연결된다.

                                                                                                                                                              ---

                                                                                                                                                              ## ✅ 오늘의 성과 요약

                                                                                                                                                              - [x] 우주선 슈팅 게임 2인 대전 + 1인 AI 대전으로 마무리
                                                                                                                                                              - [x] 4축 로봇팔 하드웨어 조립 — 90도 기준 정렬 원칙 이해
                                                                                                                                                              - [x] PCA9685로 서보 전원 분리 + I2C 제어 원리 이해
                                                                                                                                                              - [x] 웹 슬라이더 + Web Serial + JSON으로 USB 조종 완성
                                                                                                                                                              - [x] ESP32 AP 모드로 서버화 — 폰으로 케이블 없이 조종 완성
                                                                                                                                                              - [ ] 모션 티칭(매크로) 단계는 다음 기록에서 정리

                                                                                                                                                              ---

                                                                                                                                                              *Physical AI 부트캠프 Day 3 학습 정리 — 팀 모노리스*
                                                                                                                                                              