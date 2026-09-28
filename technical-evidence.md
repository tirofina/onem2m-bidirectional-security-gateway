# 제안서 기술 검증 메모

검증일: 2026-09-22. 제품 구현·성능 시험은 수행하지 않았으며, 아래 설계와 합격 기준은 개발 제안이다.

## 1. 확인한 기준선

- 저장소: IoTKETI/mobius4, 현재 checkout `3da4d8523694b5077e348ab1c9e769fb3a30df24` (v4.24.5). `git status --short` 출력 없음. 본 검증에서 제품 코드는 수정하지 않았다.
- oneM2M은 보안 기능이 없는 플랫폼이 아니다. TS-0002는 가용성, 기밀성, 무결성, 상호 인증, 무단 접근·사칭·재전송 대응, 안전에 영향을 주는 응용의 보안을 요구한다. 요구사항의 존재와 개별 구현·배포에서 제공되는 기능은 구분해야 한다. [S1]
- Suricata는 MQTT topic 및 payload 등의 필드를 검사할 수 있다. 따라서 “기존 IPS는 MQTT 내용을 이해하지 못한다”는 표현은 틀리다. 우리의 개발 범위는 인증된 장치, oneM2M 자원, 장치 상태, 데이터 신뢰도를 함께 사용하는 운영 정책이다. IPS 차단은 inline 배치와 drop/reject 규칙이 필요하다. [S3][S4]
- “공격 대부분이 IoT→서버”라는 비율 주장은 이 조사로 검증하지 못했다. 양방향 위협을 다루는 설계 이유로만 사용하고 통계처럼 제시하지 않는다.

## 2. 문서 본문용 구현 설계 (약 1쪽)

**목표.** 허브를 통과하는 oneM2M 통신을 양방향으로 보호한다. IoT→서버 구간에는 인증 주체별 요청량·데이터 크기·자원 접근 범위·신선도·형식 정책을 적용하고, 서버→IoT 구간에는 명령 범위·순서·현재 상태·입력 데이터 신뢰도를 확인한다. 방화벽과 Suricata는 네트워크 접근 및 알려진 공격을 담당하고, 자체 정책 엔진은 자원과 장치의 의미를 반영한다. 키트는 도어락·보일러 모형으로 이 판단을 보여주는 검증 플랫폼이다.

**인증과 통합.** HTTPS/MQTT의 신뢰할 수 있는 인증 종단에서 인증 주체를 oneM2M originator와 연결한다. 사용자가 적은 `fr`/`X-M2M-Origin`을 인증 사실로 취급하지 않는다. Mobius4의 HTTP와 MQTT가 합류하는 `cse/reqPrim.js`에 최소한의 정책 호출부를 추가한다. 문법 검사 후 원격 CSE로 전달하기 전에는 공통 유입 정책을, 자원 해석 및 기존 ACP 통과 후 실제 핸들러 실행 전에는 자원별 정책을 적용한다. 그룹 요청은 각 구성원의 요청에도 정책이 적용되도록 한다. HTTP 프록시 하나로 차단을 완성했다고 주장하지 않는다.

**명령과 상태.** 정책 결과에는 허용·거부·보류, 근거 규칙, 정책 버전, 참조한 장치 상태 버전과 시각을 남긴다. 허용된 명령에는 짧은 유효시간과 단일 실행 식별자를 부여하고, 실제 장치에 명령을 내리는 어댑터에서 상태·만료·중복을 다시 검사한다. DB의 자원 변경 성공이 실제 장치 동작 성공을 의미하지 않으므로 `요청→승인→전달→실행 확인/실패/결과 불명`을 구분한다. 검사 직후 상태가 바뀌는 TOCTOU 문제는 자원 버전 조건과 실행 직전 재검증으로 줄이며, 물리적 안전은 장치의 로컬 인터록을 유지한다.

**알림·우회.** notification은 기본적으로 자원 변경 통지 경로이므로 사전 차단 장치로만 의존하지 않는다. CSE가 생성한 notification 및 원격 전달도 허용된 목적지로 제한하고, 해당 통지가 액추에이터를 동작시키면 실행 어댑터에서 동일한 명령 정책을 적용한다. 일반 HTTP, MQTT broker 직접 접근, 다른 CSE, 그룹 fanout, 로컬 제어 포트가 정책을 우회하지 못하도록 네트워크 경계를 구성한다. 관리자 화면에는 원인과 조치가 이해되는 문장, 장치 상태, 차단 이력과 해제 절차를 제시한다.

**장애와 데이터.** 센서 값이 비정상이라는 이유만으로 실제 화재·가스·과열 경보를 버리지 않는다. 원본과 근거를 보존하고 신뢰도 경고를 붙이며, 확인되지 않은 값에 의존하는 위험 명령만 제한한다. 정책 서비스 장애 시 고위험 원격 명령은 보류하고 장치의 로컬 안전 동작을 유지한다. 텔레메트리·비상경보에는 제한된 큐와 별도 우선순위를 둔다. 알림 장애는 로컬 감사기록과 재전송 큐로 처리하고, 감사 저장 실패 시 동작은 장치 위험 등급에 맞춰 정의한다. 암호화된 내용을 중간 장치가 자동으로 읽을 수 없으므로 평문 검사는 신뢰하는 TLS 종단 이후에서 수행한다. 종단간 암호화가 유지되는 payload는 해당 종단의 실행 정책으로 보호한다. [S5]

## 3. 저장소에서 직접 확인한 근거

모든 링크는 검증한 commit에 고정했다.

| 근거 | 실제 확인 내용 | 설계 의미 |
|---|---|---|
| [bindings/http.js L224–231](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/bindings/http.js#L224) | HTTP를 primitive로 변환해 `prim_handling` 호출. 다른 CRUD도 같은 중앙 함수 호출 | HTTP 전용 미들웨어만으로 MQTT까지 보호할 수 없음 |
| [bindings/mqtt.js L136–210](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/bindings/mqtt.js#L136) | MQTT 수신 primitive가 동일 중앙 함수로 들어감 | 두 binding에 공통 정책 적용 가능 |
| [cse/reqPrim.js L41–60](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/cse/reqPrim.js#L41) | schema 검사 후 대상 CSE 판단. 원격이면 L60에서 조기 반환 | 유입 정책은 L53 이전에 두어 forwarding 누락 방지 |
| [cse/reqPrim.js L88–196](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/cse/reqPrim.js#L88) | 자원 해석 L88, ACP L174, 그룹 fanout L196. discovery/verification에 별도 흐름 존재 | 로컬 위험 정책은 기존 ACP를 대체하지 않고 정상 권한 검사 뒤 side effect 전에 추가. 조기 반환 흐름 별도 검토 |
| [cse/resources/grp.js L504–539](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/cse/resources/grp.js#L504) | 구성원별 요청이 `prim_handling`에 다시 들어감 | 그룹·각 구성원 정책의 일관성 시험 필요 |
| [cse/noti.js L41](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/cse/noti.js#L41) 및 [L424](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/cse/noti.js#L424) | 일반 자원 변경 통지 경로와 HTTP/MQTT 송신. net=7 blocking 미구현이라고 명시 | subscription을 이미 완성된 사전 실행 차단장치로 설명하면 안 됨 |
| [docs/tls.md L209–231](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/docs/tls.md#L209) | 서버 인증·암호화. 클라이언트 인증서와 originator 연결 없음. HTTP listener 유지 | HTTPS 사용만으로 장치 인증을 입증할 수 없음; 인증 주체 연결·평문 진입 차단 필요 |
| [cse/hostingCSE.js L1757–1777](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/cse/hostingCSE.js#L1757) | 설정된 admin originator와 `fr` 일치 시 특별 허용 | 관리 identity 보호 및 인증된 주체와의 연결 중요 |
| [config/default.json L148](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/config/default.json#L148) / [docker-compose.yml L118–122](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/docker-compose.yml#L118) | source default Helmet/rate limit false, Compose 기본 true | “Mobius4에는 rate limit이 없다/기본 배포가 모두 꺼져 있다” 금지. 장치별·MQTT 포함 quota의 추가 가치로 설명 |
| [docker/mosquitto/mosquitto.conf](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/docker/mosquitto/mosquitto.conf) / [bindings/mqtt.js L49](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/bindings/mqtt.js#L49) | private Compose망 broker의 anonymous 허용; host 미공개. CSE 연결 코드 tcp 및 자격증명 옵션 없음 | 외부 공개용 인증·topic ACL·TLS 설계를 별도로 구현해야 함. 현 기본 구성을 인터넷 공개로 단정하면 안 됨 |
| [cse/missing-data-subscription.js](https://github.com/IoTKETI/mobius4/blob/3da4d8523694b5077e348ab1c9e769fb3a30df24/cse/missing-data-subscription.js) | missing data notification 구현 파일 존재 | 오래된 support matrix를 근거로 “누락 데이터 감지가 없다”라고 쓰지 말 것 |

## 4. MVP 검증 10항목

모든 수치는 제안한 합격 목표이며 측정된 성능이 아니다. 개발 착수 시 시험 장비·메시지·정상 부하를 고정한다. 비교군은 A: 보안 설정을 적용한 Mobius4, B: A+방화벽/Suricata, C: B+자체 정책. 약한 기본 설정을 그대로 둔 A와 비교하여 성과를 부풀리지 않는다.

| ID | 시험 | 합격 기준 초안 / 증거 |
|---|---|---|
| T01 | 인증 장치의 정상 HTTP/MQTT 왕복, 접근 허용/거부 | 정상 데이터·명령 전부 처리, 기존 ACP 거부는 유지. 응답·자원·실행 로그 대조 |
| T02 | 다른 장치 및 admin originator 사칭, 허용되지 않은 MQTT topic | 위조 `fr`만으로 권한 취득 0건. 인증 주체와 자원 소유권 불일치 및 broker ACL 거부 기록 |
| T03 | IoT→서버 과다 요청·과대 payload·다른 자원 쓰기 | 설정 quota/size/scope 위반 차단. 함께 동작하는 정상 장치의 성공률 99% 이상 목표. 장치별 처리량·오류·CPU/메모리 기록 |
| T04 | 오래된 데이터·중복 요청·MQTT 재전송 | 만료된 제어 입력은 사용하지 않음. 합법적 재전송 때문에 공격으로 오판하지 않으며 액추에이터 중복 실행 0건 |
| T05 | 높은 온도/CO 등 실제 비상과 조작 의심값 구분 | 비상경보 소실 0건. 의심값은 근거와 원본 보존, 검증되지 않은 입력에 의존하는 위험 명령 제한 |
| T06 | 서버→IoT 권한은 정상이나 안전 범위/순서 위반 명령 | 등록된 위험 시험 사례에서 실제 실행 0건. 정책 근거·장치 상태·정책 버전을 남김 |
| T07 | 센서 조작→정상 서버 자동화→위험 명령 연쇄 | 입력 이상 사건과 명령 판단을 같은 사건으로 추적. 사전 정의한 위험 명령 실행 0건. A/B/C별 검출·차단 차이 보고 |
| T08 | MQTT 직접 경로·원격 CSE·그룹·notification·일반 HTTP 우회 | 범위 내 각 경로에서 동일 정책 적용. 허용하지 않은 egress 목적지 접근 0건. 허브 외 경로는 배포 전 제거/격리 |
| T09 | 정책 판단 뒤 상태 변화, 프로세스/네트워크/알림/감사 장애 | 상태 버전 불일치 명령 실행 0건. 고위험 원격 명령 보류, 로컬 안전 동작 유지. 재시작 뒤 결과 불명 표시·큐 재전송·중복 실행 방지 |
| T10 | 정상 부하 성능 및 설명 가능한 운영 | 초기 부하 예: 장치 100대×초당 1개×payload 1KiB, HTTP/MQTT 별도 및 혼합 30분. B 대비 C의 p95 추가 지연 20ms 이하, 정상 요청 오차단 0.5% 이하, 지역 대시보드 경보 p95 3초 이하를 초기 목표로 삼음. p50/p95/p99·부하별 처리량·리소스 보고; 달성했다고 서술하지 않음 |

## 5. 공식 웹 근거 6건

- **[S1] oneM2M TS-0002 v5.3.0 §6.4 Security Requirements** — SER001/008/009/012, SER075 등. 요구사항과 구현 보장을 구분. https://specifications.onem2m.org/ts/ts-0002/v5.3.0/6.4/
- **[S2] oneM2M TS-0010 Security (공식 release notes diff 페이지)** — MQTT broker 인증, 인증 정보와 AE/CSE ID 연결, topic 권한, 수신 CSE 자원 권한의 분리. 정확한 출시 대상 버전의 준수성은 개발 착수 때 고정할 것. https://specifications.onem2m.org/ts/ts-0010/latest/release_notes_diff/7-security.html
- **[S3] OISF Suricata 8.0.4 §8.33 MQTT Keywords** — MQTT header/topic/payload 검사 지원. https://docs.suricata.io/en/suricata-8.0.4/rules/mqtt-keywords.html
- **[S4] OISF Suricata 8.0.4 §22.1 IPS Concept** — inline L2/L3, 기본 허용, drop/reject 규칙. 장애 예외 정책까지 설정 필요. https://docs.suricata.io/en/suricata-8.0.4/ips/ips-concept.html
- **[S5] OISF 공식 Suricata 설정 문서 Encrypted traffic** — 암호화 트래픽을 복호화하지 않으며 암호화 이후 payload 검사는 제한. https://github.com/OISF/suricata/blob/main/doc/userguide/configuration/suricata-yaml.rst
- **[S6] oneM2M TS-0040 v5.0.2 §6.5.2** — blocking subscription을 이용한 Modbus 쓰기·자원 동기화 절차. 이를 악성 명령 판단을 갖춘 보안 제품과 동일시하지 말 것. https://specifications.onem2m.org/ts/ts-0040/v5.0.2/6.5/

## 6. 제안서에서 피할 표현

- “oneM2M에는 감시·차단 기능이 없다” → “표준의 보안 기능과 기존 구현을 바탕으로 장치별 상태·안전 정책 및 운영 기능을 결합한다.”
- “IPS는 packet만 보고 명령을 못 본다” → “IPS의 MQTT 검사를 재사용하고, 자원·장치 상태를 함께 판단하는 정책을 추가한다.”
- “실행 전 차단으로 사고를 완전히 막는다” → “검사 가능한 명령을 정책에 따라 보류/차단하며, 실행 어댑터 검증과 로컬 인터록을 함께 사용한다.”
- “모든 공격을 차단하는 전방위 gateway” → “허브를 통과하는 지원 프로토콜과 배포 경계 안에서 양방향 보호를 제공한다.”
- “정상 인증 데이터이므로 센서 값이 참이다” → “인증은 송신 주체를 검증하며 물리적 측정값의 참 여부는 별도 근거가 필요하다.”
- “국내 최초/세계 최초/특허 가능/검출률 99% 달성” → 실제 비교·특허 검토·시험 결과 없이는 쓰지 않는다.
